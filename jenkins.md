## Jenkins Installation

### Step 1 - Install Java
```bash
sudo apt-get update
sudo apt-get install -y fontconfig openjdk-17-jre
java -version
```
> **Why?** Jenkins is a Java application. It cannot run without Java. Java 17 is the recommended version for modern Jenkins.

---

### Step 2 - Add Jenkins Repository
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```
> **Why?** Jenkins is not available in Ubuntu's default repository. We add Jenkins' official repo so apt can download it directly from Jenkins.

---

### Step 3 - Install Jenkins
```bash
sudo apt-get update
sudo apt-get install -y jenkins
```

---

### Step 4 - Start Jenkins
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```
> **Why?**
> - `start` → Starts Jenkins now
> - `enable` → Auto-starts Jenkins on every reboot
> - `status` → Confirms Jenkins is running without errors

---

### Step 5 - Get Initial Admin Password
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
> **Why?** First time Jenkins starts, it creates a random admin password in this file. We need this to login for the first time through the browser.

---

### Step 6 - Open Jenkins in Browser
```
http://<your-server-ip>:8080
```
> Paste the password from Step 5, install suggested plugins, and create your admin user.

---

### Step 7 - Allow Jenkins to use Docker
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
> **Why?** If your Jenkins pipelines need to build Docker images, Jenkins user needs permission to run Docker commands. This adds jenkins user to the docker group.

---
