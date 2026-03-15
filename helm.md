## Helm Installation

### What is Helm?
Helm is a package manager for Kubernetes. Just like `apt` installs software on Ubuntu, Helm installs applications on Kubernetes cluster. These packages are called **Charts**.

---

### Step 1 - Install Helm
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
```
> **Why?** Downloads Helm's GPG key to verify the package is genuine.

```bash
sudo apt-get install apt-transport-https --yes
```
> **Why?** Allows apt to download packages over HTTPS (secure connection).

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
```
> **Why?** Adds Helm's official repository so apt knows where to download Helm from.

```bash
sudo apt-get update
sudo apt-get install helm
```

---

### Step 2 - Verify Installation
```bash
helm version
```

---

### Basic Helm Commands
```bash
helm repo add <repo-name> <repo-url>
```
> **Why?** Adds a chart repository (like adding a new app store). Example: adding bitnami repo which has 100+ popular apps.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
> **Why?** `repo update` refreshes the list of available charts from all added repos. Like `apt-get update`.

```bash
helm search repo nginx
```
> **Why?** Searches for available charts in your added repositories.

```bash
helm install my-nginx bitnami/nginx
```
> **Why?** Installs nginx on your Kubernetes cluster using Helm. `my-nginx` is the release name you give it.

```bash
helm list
```
> **Why?** Shows all Helm releases (installed apps) in current namespace.

```bash
helm uninstall my-nginx
```
> **Why?** Removes the installed app and all its Kubernetes resources.

```bash
helm upgrade my-nginx bitnami/nginx
```
> **Why?** Upgrades an existing release to the latest version of the chart.

```bash
helm rollback my-nginx 1
```
> **Why?** Rolls back to a previous version if something breaks after upgrade.

```bash
helm status my-nginx
```
> **Why?** Shows the current status and details of an installed release.

---
