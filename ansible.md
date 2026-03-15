## Ansible Installation

### Step 1 - Add Ansible Repository
```bash
sudo apt-get update
sudo apt-get install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
```
> **Why?** Ubuntu's default repo may have an older Ansible version. We add the official Ansible PPA to get the latest stable version.

---

### Step 2 - Install Ansible
```bash
sudo apt-get install -y ansible
ansible --version
```

---

### Step 3 - Setup Inventory File
```bash
sudo nano /etc/ansible/hosts
```
> **Why?** The inventory file tells Ansible which servers to manage. Without this, Ansible does not know which machines to connect to.

```ini
# Add your servers like this:
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

---

### Step 4 - Setup SSH Keys
```bash
ssh-keygen -t rsa -b 4096
```
> **Why?** Ansible connects to remote servers via SSH. SSH keys allow passwordless login which is required for automation. Without this, Ansible will ask password every time.

```bash
ssh-copy-id ubuntu@<target-server-ip>
```
> **Why?** Copies our public key to the target server so we can SSH in without a password.

---

### Basic Ansible Commands
```bash
ansible all -m ping
```
> **Why?** Tests SSH connection to all servers in inventory. If you get "pong" back, the connection is working fine.

```bash
ansible all -m shell -a "uptime"
```
> **Why?** Runs the uptime command on all servers to see how long they have been running.
> `-m shell` = use shell module, `-a` = arguments (the actual command)

```bash
ansible webservers -m apt -a "name=nginx state=present" --become
```
> **Why?** Installs nginx on all webservers. `--become` means run as sudo.

```bash
ansible-playbook my-playbook.yml
```
> **Why?** Runs a playbook (a file with multiple tasks) on the servers defined inside it.

```bash
ansible-playbook my-playbook.yml --check
```
> **Why?** Dry run. Shows what WOULD happen without actually making any changes. Always good to run before the real playbook.

---

### Basic Playbook Example
```yaml
# my-playbook.yml
---
- name: Setup Web Server
  hosts: webservers
  become: yes          # Run all tasks as sudo

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      # Why? Always update package list before installing

    - name: Install nginx
      apt:
        name: nginx
        state: present
      # Why? state: present means install if not already installed

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes
      # Why? Starts nginx now AND auto-starts it on every reboot
```

---

---

## Quick Reference Table

| Tool | Check Version | Start Service | Check Status |
|------|--------------|---------------|--------------|
| Docker | `docker --version` | `sudo systemctl start docker` | `sudo systemctl status docker` |
| Kubernetes | `kubectl version` | `sudo systemctl start kubelet` | `kubectl get nodes` |
| Jenkins | Jenkins UI | `sudo systemctl start jenkins` | `sudo systemctl status jenkins` |
| Terraform | `terraform --version` | N/A (CLI tool) | N/A |
| Ansible | `ansible --version` | N/A (CLI tool) | N/A |

---

## Common Troubleshooting Commands

```bash
# Check if a service is running
sudo systemctl status <service-name>

# See live logs of a service
sudo journalctl -u <service-name> -f

# Check which ports are open
sudo ss -tlnp

# Check disk space (important before installations)
df -h

# Check available RAM
free -h
```

---

*Personal DevOps Installation Notes*
