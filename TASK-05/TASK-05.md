# 🚀 Task-05: Associate Elastic IP with EC2 Instance Using Terraform

## 📖 Overview

In this task, we provision an AWS EC2 instance using Terraform and associate an Elastic IP (EIP) to provide a stable public IP address. This ensures that the application hosted on the EC2 instance remains accessible through a consistent endpoint, even if the instance is stopped and started.

To improve portability and maintainability, configurable values such as instance name, instance type, and Elastic IP name are managed using Terraform variables.

---

## 🎯 Objectives

✅ Create an EC2 instance named **xfusion-ec2**

✅ Use the latest **Amazon Linux 2 AMI**

✅ Launch the instance as **t2.micro**

✅ Allocate and associate an Elastic IP named **xfusion-eip**

✅ Manage configuration using variables

✅ Output the instance name and Elastic IP address

---

## 🏗️ Architecture

```text
                ☁️ AWS Cloud
                      │
                      ▼
              🖥️ EC2 Instance
               (xfusion-ec2)
                      │
                      ▼
            🌍 Elastic IP (EIP)
               (xfusion-eip)
                      │
                      ▼
             Stable Public Access
```

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```

---

# 📝 variables.tf

```hcl
variable "KKE_INSTANCE_NAME" {
  description = "EC2 Instance Name"
  type        = string
}

variable "KKE_INSTANCE_TYPE" {
  description = "EC2 Instance Type"
  type        = string
}

variable "KKE_EIP_NAME" {
  description = "Elastic IP Name"
  type        = string
}
```

---

# 📋 terraform.tfvars

```hcl
KKE_INSTANCE_NAME = "xfusion-ec2"
KKE_INSTANCE_TYPE = "t2.micro"
KKE_EIP_NAME      = "xfusion-eip"
```

---

# ⚙️ main.tf

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}

resource "aws_instance" "xfusion_ec2" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.KKE_INSTANCE_TYPE

  tags = {
    Name = var.KKE_INSTANCE_NAME
  }
}

resource "aws_eip" "xfusion_eip" {
  domain   = "vpc"
  instance = aws_instance.xfusion_ec2.id

  tags = {
    Name = var.KKE_EIP_NAME
  }
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_instance_name" {
  value = aws_instance.xfusion_ec2.tags["Name"]
}

output "KKE_eip" {
  value = aws_eip.xfusion_eip.public_ip
}
```

---

# 🚀 Deployment Steps

### 1️⃣ Initialize Terraform

```bash
terraform init
```

Expected:

```text
Terraform has been successfully initialized!
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

### 3️⃣ Review Execution Plan

```bash
terraform plan
```

Expected:

```text
+ aws_instance.xfusion_ec2
+ aws_eip.xfusion_eip
```

---

### 4️⃣ Deploy Infrastructure

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify EC2 Instance Creation

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=xfusion-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
--output table
```

Expected:

```text
------------------------------------
|      DescribeInstances           |
+----------------------+-----------+
| i-xxxxxxxxxxxxxxxxx  | running   |
+----------------------+-----------+
```

---

## ✅ Verify Elastic IP Allocation

```bash
aws ec2 describe-addresses \
--query "Addresses[*].[PublicIp,InstanceId]" \
--output table
```

Expected:

```text
-----------------------------------------
|        DescribeAddresses              |
+---------------+----------------------+
| 54.xx.xx.xx   | i-xxxxxxxxxxxxxxxxx   |
+---------------+----------------------+
```

✔️ Elastic IP successfully associated with the EC2 instance.

---

## ✅ Verify EC2 Public IP

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=xfusion-ec2" \
--query "Reservations[*].Instances[*].[PublicIpAddress]" \
--output table
```

Expected:

```text
--------------------
| PublicIpAddress  |
+------------------+
| 54.xx.xx.xx      |
+------------------+
```

✔️ Public IP should match the Elastic IP.

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
KKE_instance_name = "xfusion-ec2"

KKE_eip = "54.xx.xx.xx"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_instance.xfusion_ec2
aws_eip.xfusion_eip
```

---

## ✅ Final Validation (Mandatory)

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

# 📚 Key Concepts Learned

🔹 Terraform Data Sources

🔹 Dynamic AMI Discovery

🔹 EC2 Instance Provisioning

🔹 Elastic IP Allocation & Association

🔹 Terraform Variables & Outputs

🔹 Infrastructure as Code (IaC)

🔹 AWS Resource Verification

🔹 Terraform State Management

---

# 🏆 Outcome

Successfully provisioned an Amazon Linux 2 EC2 instance and associated a static Elastic IP using Terraform. The application now has a stable public endpoint, ensuring reliable access and simplifying infrastructure management. 🚀☁️🌍
