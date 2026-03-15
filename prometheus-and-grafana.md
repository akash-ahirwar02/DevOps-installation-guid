## Prometheus Installation

### What is Prometheus?
Prometheus is a monitoring tool. It collects metrics (CPU usage, memory, request count etc.) from your servers and applications and stores them. You can then query these metrics and set alerts.

---

### Pre-requisite
> Prometheus is best installed on Kubernetes using Helm. Make sure Helm and kubectl are working before proceeding.

---

### Step 1 - Add Prometheus Helm Repository
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
> **Why?** This adds the official Prometheus community charts repository. All Prometheus related charts are available here.

---

### Step 2 - Create a Namespace for Monitoring
```bash
kubectl create namespace monitoring
```
> **Why?** It is a good practice to keep monitoring tools in a separate namespace. This keeps things organized and separated from your application pods.

---

### Step 3 - Install Prometheus
```bash
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring
```
> **Why?**
> - `prometheus` (first) → Name we are giving to this release
> - `prometheus-community/prometheus` → Chart we are installing
> - `--namespace monitoring` → Install it in the monitoring namespace we created

---

### Step 4 - Verify Installation
```bash
kubectl get pods -n monitoring
```
> **Why?** Check that all Prometheus pods are Running. You should see prometheus-server, alertmanager, and node-exporter pods.

```bash
kubectl get svc -n monitoring
```
> **Why?** See what services were created. We need the service name to access Prometheus UI.

---

### Step 5 - Access Prometheus UI (Port Forward)
```bash
kubectl port-forward svc/prometheus-server 9090:80 -n monitoring
```
> **Why?** Prometheus service is not exposed publicly by default. Port forwarding lets us access it on our local machine at port 9090.

```
Now open in browser: http://localhost:9090
```

---

### Important Prometheus Concepts
```
Metrics     → Numbers that Prometheus collects (CPU %, memory bytes, request count)
Scrape      → Prometheus pulling metrics from a target (it checks every 15-30 seconds)
Exporter    → A small agent that exposes metrics for Prometheus to collect
              (node-exporter for server metrics, kube-state-metrics for K8s metrics)
AlertManager → Component that sends alerts via email/Slack when something goes wrong
PromQL      → Prometheus Query Language used to query metrics
```

### Useful PromQL Queries
```promql
# CPU usage of all nodes
node_cpu_seconds_total

# Available memory
node_memory_MemAvailable_bytes

# Number of running pods
kube_pod_status_phase{phase="Running"}

# HTTP request rate
rate(http_requests_total[5m])
```

---



## Grafana Installation

### What is Grafana?
Grafana is a visualization tool. It connects to Prometheus (or other data sources) and creates beautiful, interactive dashboards from the metrics data. Prometheus stores data, Grafana shows it as graphs and charts.

---

### Step 1 - Add Grafana Helm Repository
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
> **Why?** Adds Grafana's official Helm chart repository.

---

### Step 2 - Install Grafana
```bash
helm install grafana grafana/grafana \
  --namespace monitoring
```
> **Why?** Installs Grafana in the same monitoring namespace where Prometheus is running. Keeping them together makes it easy to connect them.

---

### Step 3 - Get Grafana Admin Password
```bash
kubectl get secret --namespace monitoring grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
> **Why?** Grafana generates a random admin password and stores it as a Kubernetes secret. This command decodes and shows that password so we can log in.

---

### Step 4 - Access Grafana UI (Port Forward)
```bash
kubectl port-forward svc/grafana 3000:80 -n monitoring
```
> **Why?** Just like Prometheus, Grafana is not publicly exposed by default. This makes it accessible at port 3000 on our local machine.

```
Open in browser: http://localhost:3000
Username: admin
Password: (from Step 3)
```

---

### Step 5 - Connect Prometheus as Data Source in Grafana

1. Login to Grafana
2. Go to **Settings (gear icon) → Data Sources**
3. Click **Add data source**
4. Select **Prometheus**
5. In URL field, enter:
```
http://prometheus-server.monitoring.svc.cluster.local
```
> **Why this URL?** This is the internal Kubernetes DNS name for the Prometheus service. Since both Grafana and Prometheus are inside the same cluster, they can talk to each other using this internal URL. No need to expose Prometheus publicly.

6. Click **Save & Test** → Should show "Data source is working"

---

### Step 6 - Import a Dashboard

Instead of building dashboards from scratch, we can import pre-made dashboards.

1. Go to **Dashboards → Import**
2. Enter Dashboard ID: `315` (Kubernetes cluster monitoring)
3. Click **Load**
4. Select Prometheus as data source
5. Click **Import**

> **Popular Dashboard IDs:**
> - `315` → Kubernetes cluster overview
> - `1860` → Node Exporter Full (server metrics)
> - `6417` → Kubernetes pods monitoring

---

### Install Both Together (Recommended)
```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```
> **Why?** `kube-prometheus-stack` installs Prometheus + Grafana + AlertManager + all exporters together in one command. It also comes with pre-built Kubernetes dashboards. This is the easiest and most complete setup.

```bash
# Get Grafana password for kube-prometheus-stack
kubectl get secret --namespace monitoring kube-prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# Access Grafana
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

---

