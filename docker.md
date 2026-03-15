# DevOps Tools - Installation Notes
> For Ubuntu/Debian based systems

---

## 1. Docker Installation

### Step 1 - Update system and install basic tools
```bash
sudo apt-get update
```
> **Why?** Before installing anything, we update the package list so our system knows about the latest available software.

```bash
sudo apt-get install -y ca-certificates curl gnupg
```
> **Why?**
> - `ca-certificates` → Allows system to verify SSL certificates (secure downloads)
> - `curl` → Tool to download files from internet
> - `gnupg` → Used to verify that downloaded software is genuine (not fake)

---

### Step 2 - Add Docker's GPG Key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
> **Why?** GPG key is like a digital signature. It proves that the Docker software we are downloading is officially from Docker and has not been tampered with.

---

### Step 3 - Add Docker Repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
> **Why?** This adds Docker's official download source to our system. Without this, apt would not know where to download Docker from.

---

### Step 4 - Install Docker
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
> **Why?**
> - `docker-ce` → Main Docker engine
> - `docker-ce-cli` → Command line tool to talk to Docker
> - `containerd.io` → Manages container lifecycle (start, stop, delete)
> - `docker-buildx-plugin` → Used to build Docker images
> - `docker-compose-plugin` → Allows running multiple containers together

---

### Step 5 - Start Docker Service
```bash
sudo systemctl start docker
sudo systemctl enable docker
```
> **Why?**
> - `start` → Starts Docker right now
> - `enable` → Makes Docker start automatically every time server reboots

---

### Step 6 - Add user to Docker group
```bash
sudo usermod -aG docker $USER
newgrp docker
```
> **Why?** By default, only root can run Docker commands. This adds our user to the docker group so we can run Docker without typing sudo every time.

---

### Step 7 - Verify Installation
```bash
docker --version
docker run hello-world
```
> **Why?** `hello-world` is a test image. If it runs successfully, Docker is working correctly.

---

---
