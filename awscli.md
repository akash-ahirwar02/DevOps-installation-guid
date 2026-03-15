## AWS CLI Installation

### What is AWS CLI?
AWS CLI (Command Line Interface) lets you manage all AWS services (EC2, S3, EKS, IAM etc.) directly from your terminal instead of clicking around in the AWS Console.

---

### PRE-REQUISITES - Do This in AWS Console First

> Before using AWS CLI, you need to create an IAM User with Access Keys in AWS Console.

#### Step A - Create IAM User in AWS Console

1. Login to **AWS Console** → Go to **IAM**
2. Click **Users** → **Create User**
3. Enter a username (example: `devops-user`)
4. Click **Next**

#### Step B - Attach Permissions

5. Select **Attach policies directly**
6. Search and attach the policies you need:
   - `AdministratorAccess` → Full access to everything (use carefully)
   - OR attach specific policies like:
     - `AmazonEC2FullAccess` → For EC2
     - `AmazonS3FullAccess` → For S3
     - `AmazonEKSFullAccess` → For EKS
7. Click **Next** → **Create User**

#### Step C - Create Access Keys

8. Click on the user you just created
9. Go to **Security credentials** tab
10. Scroll down to **Access keys** → Click **Create access key**
11. Select **Command Line Interface (CLI)**
12. Check the confirmation box → Click **Next** → **Create access key**
13. **IMPORTANT: Download the .csv file or copy both keys NOW**
    - `Access Key ID` → Example: `AKIAIOSFODNN7EXAMPLE`
    - `Secret Access Key` → Example: `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`

> **Why save keys immediately?** AWS shows the Secret Access Key only ONCE. If you close this page without saving, you will have to create new keys.

---

### Step 1 - Install AWS CLI on Ubuntu
```bash
sudo apt-get update
sudo apt-get install -y unzip curl
```
> **Why?** We need curl to download and unzip to extract the AWS CLI package.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
> **Why?** Downloads the latest AWS CLI v2 installer from Amazon's official server.

```bash
unzip awscliv2.zip
```
> **Why?** Extracts the downloaded zip file.

```bash
sudo ./aws/install
```
> **Why?** Runs the installer which places the aws binary in /usr/local/bin.

```bash
aws --version
```
> **Why?** Confirms AWS CLI was installed successfully.

---

### Step 2 - Configure AWS CLI with Your Keys
```bash
aws configure
```
> **Why?** This command sets up your AWS credentials so CLI knows which AWS account to connect to and with what permissions.

It will ask 4 things:
```
AWS Access Key ID:     → Paste your Access Key ID from console
AWS Secret Access Key: → Paste your Secret Access Key from console
Default region name:   → Enter your region (example: us-east-1 or ap-south-1)
Default output format: → Type json (recommended for readable output)
```

> **Why region?** AWS has data centers in multiple regions worldwide. You need to tell CLI which region your resources are in.
> **Why json?** JSON format is easy to read and can be parsed by other tools.

---

### Step 3 - Verify Configuration
```bash
aws sts get-caller-identity
```
> **Why?** This checks if your credentials are working. It returns your AWS Account ID, User ID, and ARN. If you see this, your CLI is connected to AWS successfully.

Expected output:
```json
{
    "UserId": "AIDAIOSFODNN7EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/devops-user"
}
```

---

### Basic AWS CLI Commands

#### EC2 Commands
```bash
aws ec2 describe-instances
```
> **Why?** Lists all EC2 instances in your configured region with their details.

```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' --output table
```
> **Why?** Same as above but shows only Instance ID, State, and IP in a clean table format. Much easier to read.

```bash
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```
> **Why?** Start or stop an EC2 instance from terminal without going to console.

#### S3 Commands
```bash
aws s3 ls
```
> **Why?** Lists all your S3 buckets.

```bash
aws s3 ls s3://my-bucket-name
```
> **Why?** Lists files inside a specific bucket.

```bash
aws s3 cp myfile.txt s3://my-bucket-name/
```
> **Why?** Uploads a file to S3 bucket.

```bash
aws s3 cp s3://my-bucket-name/myfile.txt .
```
> **Why?** Downloads a file from S3 to current directory.

```bash
aws s3 sync ./local-folder s3://my-bucket-name/
```
> **Why?** Syncs entire local folder to S3. Only uploads files that are new or changed.

#### EKS Commands
```bash
aws eks list-clusters
```
> **Why?** Shows all EKS clusters in your region.

```bash
aws eks update-kubeconfig --name my-cluster --region us-east-1
```
> **Why?** Configures kubectl to connect to your EKS cluster. After this command, all kubectl commands will work with your EKS cluster.

```bash
aws eks list-nodegroups --cluster-name my-cluster
```
> **Why?** Lists all node groups in an EKS cluster.

#### IAM Commands
```bash
aws iam list-users
```
> **Why?** Lists all IAM users in your AWS account.

```bash
aws iam get-user
```
> **Why?** Shows details of the currently configured IAM user.

---

### Where are Credentials Stored?
```bash
cat ~/.aws/credentials
```
> Credentials are stored in this file. Never share this file or commit it to GitHub!

```bash
cat ~/.aws/config
```
> Region and output format are stored here.

---

### Multiple AWS Accounts (Profiles)
```bash
aws configure --profile work
aws configure --profile personal
```
> **Why?** If you have multiple AWS accounts, you can save them as separate profiles.

```bash
# Use a specific profile
aws s3 ls --profile work
aws ec2 describe-instances --profile personal
```

```bash
# Set a default profile for current session
export AWS_PROFILE=work
```

---

*Personal DevOps Installation Notes*
