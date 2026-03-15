## Kubernetes - kubeadm Installation

> Run these steps on ALL nodes (both Master and Worker) unless mentioned otherwise.

---

### Step 1 - Turn off Swap
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
> **Why?** Kubernetes requires swap to be OFF. If swap is on, Kubernetes cannot correctly manage memory for pods.
> - `swapoff -a` → Disables swap immediately
> - `sed` command → Comments out swap line in /etc/fstab so it stays off after reboot

---

### Step 2 - Load Required Kernel Modules
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
> **Why?**
> - `overlay` → Required for container storage to work efficiently
> - `br_netfilter` → Required for Kubernetes networking. Allows iptables to see bridged traffic so pods can communicate with each other.

---

### Step 3 - Set Networking Parameters
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```
> **Why?**
> - `bridge-nf-call-iptables = 1` → Allows iptables to process traffic from containers
> - `ip_forward = 1` → Allows server to forward network packets between pods and services
> - Without these, pod-to-pod communication will NOT work

---

### Step 4 - Install and Configure containerd
```bash
sudo apt-get update
sudo apt-get install -y containerd.io

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```
> **Why?** containerd is the container runtime - it actually runs the containers.
> We set `SystemdCgroup = true` because Kubernetes uses systemd to manage processes, and containerd must use the same cgroup driver, otherwise kubelet will fail to start.

---

### Step 5 - Install kubeadm, kubelet, kubectl
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
> **Why?**
> - `kubelet` → Runs on every node, manages pods on that node
> - `kubeadm` → Tool to initialize and join cluster
> - `kubectl` → Command line tool to interact with cluster
> - `apt-mark hold` → Prevents accidental upgrade of these packages which could break the cluster

---

### Step 6 - Initialize Cluster (Master Node ONLY)
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
> **Why?**
> - Sets up the control plane (API server, etcd, scheduler, controller manager)
> - `--pod-network-cidr=192.168.0.0/16` → Defines IP range for pods. This specific range is required by Calico CNI.

---

### Step 7 - Setup kubeconfig (Master Node ONLY)
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
> **Why?** kubectl needs a config file to know which cluster to connect to. This copies the admin config to our home directory so kubectl works without sudo.

---

### Step 8 - Install Calico CNI (Master Node ONLY)
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```
> **Why?** CNI (Container Network Interface) is required for pods to communicate across nodes. Without CNI, all pods will stay in Pending state. Calico is one of the most popular and stable CNI plugins.

---

### Step 9 - Join Worker Nodes
```bash
# This command is shown at the end of kubeadm init output on master
sudo kubeadm join <master-ip>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```
> **Why?** This command connects the worker node to the master so the cluster can schedule pods on it.

```bash
# If token expired (tokens expire after 24 hours), run this on master:
kubeadm token create --print-join-command
```
> **Why?** kubeadm tokens expire after 24 hours for security reasons. This creates a fresh token and prints the complete join command.

---

### Verify
```bash
kubectl get nodes
kubectl get pods -n kube-system
```
