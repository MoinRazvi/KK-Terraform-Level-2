## 🚀 Task-02: Launch EC2 in Private VPC Subnet Using Terraform

### 📖 Overview

In this task, we provision a **private AWS network environment** using Terraform by creating:

* 🌐 A dedicated VPC
* 🔒 A private Subnet (without auto-assigned public IPs)
* 🖥️ An EC2 instance deployed inside the private subnet
* 🛡️ A Security Group that allows access only from within the VPC CIDR range

This setup simulates a common production architecture where backend servers are isolated from the public internet and can only communicate internally.

---

## 🎯 Objectives

✅ Create a VPC named **devops-priv-vpc**

✅ Create a private subnet named **devops-priv-subnet**

✅ Disable automatic public IP assignment

✅ Launch a **t2.micro** EC2 instance

✅ Restrict access to the EC2 instance from within the VPC only

✅ Use variables and outputs for better configuration management

---

## 🏗️ Architecture

```text
VPC (10.0.0.0/16)
│
└── Private Subnet (10.0.1.0/24)
      │
      ├── Security Group
      │      └── Allows traffic only from 10.0.0.0/16
      │
      └── EC2 Instance (t2.micro)
              └── Private Access Only
```

---

## 📂 Project Structure

```bash
terraform/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```

---

## 📝 variables.tf

```hcl
variable "KKE_VPC_CIDR" {
  description = "CIDR block for VPC"
  type        = string
}

variable "KKE_SUBNET_CIDR" {
  description = "CIDR block for Subnet"
  type        = string
}
```

---

## 📋 terraform.tfvars

```hcl
KKE_VPC_CIDR    = "10.0.0.0/16"
KKE_SUBNET_CIDR = "10.0.1.0/24"
```

---

## ⚙️ main.tf

> ⚠️ Use the AMI ID provided in your lab environment if different.

```hcl
provider "aws" {
  region = "us-east-1"
}

data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}

resource "aws_vpc" "private_vpc" {
  cidr_block = var.KKE_VPC_CIDR

  tags = {
    Name = "devops-priv-vpc"
  }
}

resource "aws_subnet" "private_subnet" {
  vpc_id                  = aws_vpc.private_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "devops-priv-subnet"
  }
}

resource "aws_security_group" "private_sg" {
  name   = "devops-priv-sg"
  vpc_id = aws_vpc.private_vpc.id

  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "private_ec2" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.private_subnet.id
  vpc_security_group_ids = [aws_security_group.private_sg.id]

  tags = {
    Name = "devops-priv-ec2"
  }
}
```

---

## 📤 outputs.tf

```hcl
output "KKE_vpc_name" {
  value = aws_vpc.private_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  value = aws_subnet.private_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  value = aws_instance.private_ec2.tags["Name"]
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
+ aws_vpc.private_vpc
+ aws_subnet.private_subnet
+ aws_security_group.private_sg
+ aws_instance.private_ec2
```

---

### 4️⃣ Deploy Infrastructure

```bash
terraform apply -auto-approve
```

---

## 🔍 Verification Steps

### ✅ Verify VPC

```bash
aws ec2 describe-vpcs \
--filters "Name=tag:Name,Values=devops-priv-vpc" \
--query "Vpcs[*].[VpcId,CidrBlock]" \
--output table
```

Expected:

```text
------------------------------------
|           DescribeVpcs          |
+--------------+------------------+
| vpc-xxxxxxx  | 10.0.0.0/16      |
+--------------+------------------+
```

---

### ✅ Verify Subnet

```bash
aws ec2 describe-subnets \
--filters "Name=tag:Name,Values=devops-priv-subnet" \
--query "Subnets[*].[SubnetId,CidrBlock,MapPublicIpOnLaunch]" \
--output table
```

Expected:

```text
----------------------------------------------------
|               DescribeSubnets                    |
+-------------+--------------+---------------------+
| subnet-xxxx | 10.0.1.0/24  | False               |
+-------------+--------------+---------------------+
```

✔️ Verify that `MapPublicIpOnLaunch` is **False**

---

### ✅ Verify EC2 Instance

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=devops-priv-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,PrivateIpAddress,State.Name]" \
--output table
```

Expected:

```text
--------------------------------------------------
|             DescribeInstances                  |
+-------------+---------------+------------------+
| i-xxxxxxxxx | 10.0.1.xx     | running          |
+-------------+---------------+------------------+
```

✔️ Instance should only have a **Private IP**

---

### ✅ Verify Security Group Rules

```bash
aws ec2 describe-security-groups \
--group-ids <security-group-id>
```

Verify:

```text
Ingress CIDR = 10.0.0.0/16
```

✔️ Access restricted to resources inside the VPC.

---

### ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
KKE_vpc_name     = "devops-priv-vpc"
KKE_subnet_name  = "devops-priv-subnet"
KKE_ec2_private  = "devops-priv-ec2"
```

---

### ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_vpc.private_vpc
aws_subnet.private_subnet
aws_security_group.private_sg
aws_instance.private_ec2
```

---

### ✅ Final Validation (Mandatory)

Run:

```bash
terraform plan
```

Expected:

```text
No changes. Your infrastructure matches the configuration.
```

🎉 This confirms Terraform state and AWS resources are fully synchronized.

---

## 📚 Key Concepts Learned

🔹 AWS VPC Networking

🔹 Private Subnet Configuration

🔹 Security Group Restrictions

🔹 EC2 Deployment in Private Networks

🔹 Terraform Variables & Outputs

🔹 Infrastructure as Code (IaC)

🔹 AWS Resource Verification

🔹 Terraform State Management

---

## 🏆 Outcome

Successfully provisioned a **private AWS environment** consisting of a VPC, private subnet, security group, and EC2 instance using Terraform. The EC2 instance remains isolated from the public internet and is accessible only from within the VPC, following secure cloud architecture best practices. 🚀🔒☁️
