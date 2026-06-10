# 🚀 Task-10: Grant EC2 Access to S3 Bucket Using Terraform

## 📖 Overview

In this task, we implement a secure AWS design pattern where an EC2 instance uploads application logs to an S3 bucket using an IAM Role instead of access keys.

The solution includes:

✅ S3 bucket for log storage

✅ IAM policy with `s3:PutObject` permission

✅ IAM role trusted by EC2

✅ IAM Instance Profile

✅ EC2 instance attached to the IAM role

This follows AWS security best practices by providing temporary credentials through the Instance Metadata Service (IMDS).

---

## 🎯 Objectives

✅ Create S3 bucket **xfusion-logs-4120**

✅ Create IAM policy **xfusion-access-policy**

✅ Create IAM role **xfusion-role**

✅ Attach policy to role

✅ Create IAM Instance Profile

✅ Launch EC2 instance **xfusion-ec2**

✅ Attach IAM role to EC2

✅ Allow uploads to the S3 bucket

---

# 🏗️ Architecture

```text
                 🖥️ EC2 Instance
                   xfusion-ec2
                         │
                         ▼
                🎭 IAM Role
                 xfusion-role
                         │
                         ▼
                📜 IAM Policy
            xfusion-access-policy
                         │
                         ▼
                   📦 S3 Bucket
                xfusion-logs-4120
```

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
├── data.tf
├── variables.tf
└── terraform.tfvars
```

---

# 📝 variables.tf

```hcl
variable "KKE_BUCKET_NAME" {
  description = "S3 Bucket Name"
  type        = string
}

variable "KKE_POLICY_NAME" {
  description = "IAM Policy Name"
  type        = string
}

variable "KKE_ROLE_NAME" {
  description = "IAM Role Name"
  type        = string
}
```

---

# 📋 terraform.tfvars

```hcl
KKE_BUCKET_NAME = "xfusion-logs-4120"
KKE_POLICY_NAME = "xfusion-access-policy"
KKE_ROLE_NAME   = "xfusion-role"
```

---

# 📂 data.tf

## Fetch Latest Amazon Linux 2 AMI

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}
```

---

# ⚙️ main.tf

## Create S3 Bucket

```hcl
resource "aws_s3_bucket" "logs_bucket" {
  bucket = var.KKE_BUCKET_NAME
}
```

---

## Create IAM Role

```hcl
resource "aws_iam_role" "xfusion_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Principal = {
        Service = "ec2.amazonaws.com"
      }

      Action = "sts:AssumeRole"
    }]
  })
}
```

---

## Create IAM Policy

```hcl
resource "aws_iam_policy" "xfusion_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "s3:PutObject"
      ]

      Resource = "${aws_s3_bucket.logs_bucket.arn}/*"
    }]
  })
}
```

---

## Attach Policy to Role

```hcl
resource "aws_iam_role_policy_attachment" "policy_attach" {
  role       = aws_iam_role.xfusion_role.name
  policy_arn = aws_iam_policy.xfusion_policy.arn
}
```

---

## Create Instance Profile

```hcl
resource "aws_iam_instance_profile" "xfusion_profile" {
  name = "xfusion-instance-profile"
  role = aws_iam_role.xfusion_role.name
}
```

---

## Create EC2 Instance

```hcl
resource "aws_instance" "xfusion_ec2" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  iam_instance_profile   = aws_iam_instance_profile.xfusion_profile.name

  tags = {
    Name = "xfusion-ec2"
  }
}
```

---

# 🚀 Deployment Steps

### 1️⃣ Initialize Terraform

```bash
terraform init
```

---

### 2️⃣ Validate Configuration

```bash
terraform validate
```

Expected:

```text
Success! The configuration is valid.
```

---

### 3️⃣ Review Plan

```bash
terraform plan
```

Expected Resources:

```text
+ aws_s3_bucket.logs_bucket
+ aws_iam_role.xfusion_role
+ aws_iam_policy.xfusion_policy
+ aws_iam_role_policy_attachment.policy_attach
+ aws_iam_instance_profile.xfusion_profile
+ aws_instance.xfusion_ec2
```

---

### 4️⃣ Deploy Infrastructure

```bash
terraform apply -auto-approve
```

---

# 🔍 Verification Steps

## ✅ Verify S3 Bucket

```bash
aws s3 ls | grep xfusion-logs-4120
```

Expected:

```text
xfusion-logs-4120
```

---

## ✅ Verify IAM Role

```bash
aws iam get-role \
--role-name xfusion-role
```

Expected:

```text
RoleName: xfusion-role
```

---

## ✅ Verify IAM Policy

```bash
aws iam get-policy \
--policy-arn <policy-arn>
```

Verify:

```text
s3:PutObject
```

permission exists.

---

## ✅ Verify Policy Attachment

```bash
aws iam list-attached-role-policies \
--role-name xfusion-role
```

Expected:

```text
xfusion-access-policy
```

---

## ✅ Verify Instance Profile

```bash
aws iam get-instance-profile \
--instance-profile-name xfusion-instance-profile
```

Expected:

```text
RoleName: xfusion-role
```

---

## ✅ Verify EC2 Instance

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=xfusion-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,State.Name]"
```

Expected:

```text
running
```

---

## ✅ Verify IAM Role Attached to EC2

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=xfusion-ec2" \
--query "Reservations[*].Instances[*].IamInstanceProfile.Arn"
```

Expected:

```text
xfusion-instance-profile
```

---

## ✅ Test Upload Permission

SSH into the instance and run:

```bash
echo "Terraform Test" > app.log

aws s3 cp app.log s3://xfusion-logs-4120/
```

Verify:

```bash
aws s3 ls s3://xfusion-logs-4120/
```

Expected:

```text
app.log
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_s3_bucket.logs_bucket
aws_iam_role.xfusion_role
aws_iam_policy.xfusion_policy
aws_iam_role_policy_attachment.policy_attach
aws_iam_instance_profile.xfusion_profile
aws_instance.xfusion_ec2
```

---

## ✅ Final Validation (Mandatory)

```bash
terraform plan
```

Expected:

```text
No changes. Your infrastructure matches the configuration.
```

🎉 Infrastructure and Terraform state are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 IAM Roles

🔹 IAM Policies

🔹 IAM Instance Profiles

🔹 EC2 Role-Based Access

🔹 Amazon S3 Permissions

🔹 Least Privilege Principle

🔹 Terraform Resource Dependencies

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned an EC2 instance with an attached IAM Role that securely uploads application logs to an S3 bucket without using static AWS credentials. This implementation follows AWS security best practices and demonstrates a production-grade pattern for service-to-service authentication. 🚀☁️🔐📦🖥️
