# 🚀 Task-04: Deploy Multiple EC2 Instances with Terraform

## 📖 Overview

In this task, we use Terraform's **`count`** meta-argument to provision multiple EC2 instances dynamically. This approach eliminates repetitive code and enables scalable infrastructure deployment using a single resource block.

The instances will be created with a consistent naming convention using the prefix **datacenter-instance** and will utilize the latest Amazon Linux 2 AMI retrieved dynamically through a data source.

---

## 🎯 Objectives

✅ Create **3 EC2 instances** using Terraform `count`

✅ Use **t2.micro** instance type

✅ Use **datacenter-key** as the SSH key pair

✅ Apply dynamic naming:

* datacenter-instance-1
* datacenter-instance-2
* datacenter-instance-3

✅ Store configuration values using variables

✅ Retrieve latest Amazon Linux 2 AMI dynamically

✅ Output all created instance names

---

## 🏗️ Architecture

```text
Terraform
    │
    ▼
Count = 3
    │
 ┌──┴──────────────────────┐
 │                         │
 ▼                         ▼
EC2-1                 EC2-2                 EC2-3
datacenter-          datacenter-          datacenter-
instance-1           instance-2           instance-3
```

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
├── variables.tf
├── locals.tf
├── terraform.tfvars
└── outputs.tf
```

---

# 📝 variables.tf

```hcl
variable "KKE_INSTANCE_COUNT" {
  description = "Number of instances"
  type        = number
}

variable "KKE_INSTANCE_TYPE" {
  description = "EC2 Instance Type"
  type        = string
}

variable "KKE_KEY_NAME" {
  description = "Key Pair Name"
  type        = string
}

variable "KKE_INSTANCE_PREFIX" {
  description = "Instance Name Prefix"
  type        = string
}
```

---

# ⚙️ locals.tf

### 🔍 Retrieve Latest Amazon Linux 2 AMI

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*"]
  }
}

locals {
  AMI_ID = data.aws_ami.amazon_linux.id
}
```

---

# ⚙️ main.tf

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "datacenter_instance" {
  count         = var.KKE_INSTANCE_COUNT
  ami           = local.AMI_ID
  instance_type = var.KKE_INSTANCE_TYPE
  key_name      = var.KKE_KEY_NAME

  tags = {
    Name = "${var.KKE_INSTANCE_PREFIX}-${count.index + 1}"
  }
}
```

---

# 📋 terraform.tfvars

```hcl
KKE_INSTANCE_COUNT  = 3
KKE_INSTANCE_TYPE   = "t2.micro"
KKE_KEY_NAME        = "datacenter-key"
KKE_INSTANCE_PREFIX = "datacenter-instance"
```

---

# 📤 outputs.tf

```hcl
output "kke_instance_names" {
  value = [
    for instance in aws_instance.datacenter_instance :
    instance.tags["Name"]
  ]
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
Plan: 3 to add, 0 to change, 0 to destroy.
```

Resources:

```text
aws_instance.datacenter_instance[0]
aws_instance.datacenter_instance[1]
aws_instance.datacenter_instance[2]
```

---

### 4️⃣ Deploy Infrastructure

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify EC2 Instances via AWS CLI

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=datacenter-instance-*" \
--query "Reservations[*].Instances[*].[InstanceId,Tags[0].Value,State.Name]" \
--output table
```

Expected:

```text
-----------------------------------------------------------
|                    DescribeInstances                    |
+----------------------+----------------------+-----------+
| i-xxxxxxxxxxxxxxx    | datacenter-instance-1| running   |
| i-yyyyyyyyyyyyyyy    | datacenter-instance-2| running   |
| i-zzzzzzzzzzzzzzz    | datacenter-instance-3| running   |
+----------------------+----------------------+-----------+
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
kke_instance_names = [
  "datacenter-instance-1",
  "datacenter-instance-2",
  "datacenter-instance-3"
]
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_instance.datacenter_instance[0]
aws_instance.datacenter_instance[1]
aws_instance.datacenter_instance[2]
```

---

## ✅ Verify Instance Count

```bash
terraform state list | grep aws_instance | wc -l
```

Expected:

```text
3
```

✔️ Confirms Terraform created exactly three EC2 instances.

---

## ✅ Verify AMI Used

```bash
terraform console
```

```hcl
local.AMI_ID
```

Expected:

```text
ami-xxxxxxxxxxxxxxxxx
```

✔️ Confirms the latest Amazon Linux 2 AMI was retrieved dynamically.

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

🎉 This confirms Terraform state and AWS infrastructure are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 Terraform `count` Meta-Argument

🔹 Dynamic Resource Creation

🔹 AWS EC2 Provisioning

🔹 Data Sources

🔹 Local Variables

🔹 Dynamic Resource Naming

🔹 Infrastructure as Code (IaC)

🔹 Terraform State Management

---

# 🏆 Outcome

Successfully deployed **three EC2 instances** using Terraform's `count` functionality, following a consistent naming convention and dynamically retrieving the latest Amazon Linux 2 AMI. This demonstrates a scalable and maintainable approach to infrastructure provisioning while reducing code duplication and improving automation efficiency. 🚀☁️🏗️
