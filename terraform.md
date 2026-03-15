## Terraform Installation

### Step 1 - Install required tools
```bash
sudo apt-get update
sudo apt-get install -y gnupg software-properties-common
```
> **Why?** `gnupg` verifies HashiCorp's digital signature. `software-properties-common` allows managing custom repositories.

---

### Step 2 - Add HashiCorp GPG Key
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```
> **Why?** This key proves the Terraform package is officially from HashiCorp and has not been modified by anyone.

---

### Step 3 - Add HashiCorp Repository
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list
```
> **Why?** Terraform is not in Ubuntu's default repo. This tells apt where to find and download Terraform from.

---

### Step 4 - Install Terraform
```bash
sudo apt-get update
sudo apt-get install -y terraform
terraform --version
```

---

### Basic Terraform Commands
```bash
terraform init
```
> **Why?** Downloads required providers (AWS, Azure, GCP etc.) and sets up the backend. Must run this first in every new project folder.

```bash
terraform plan
```
> **Why?** Shows what changes WILL be made without actually doing anything. Always run this before apply to review changes.

```bash
terraform apply
```
> **Why?** Actually creates or modifies the infrastructure. Type 'yes' to confirm.

```bash
terraform destroy
```
> **Why?** Deletes all infrastructure created by this Terraform config. Use carefully!

```bash
terraform fmt
```
> **Why?** Auto-formats your .tf files to follow standard style.

```bash
terraform validate
```
> **Why?** Checks if your config files have syntax errors before running plan/apply.

---

---
