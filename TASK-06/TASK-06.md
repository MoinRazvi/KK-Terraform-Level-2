# 🚀 Task-06: Launch EC2 Instance from Custom AMI Using Terraform

## 📖 Overview

In this task, we create a custom Amazon Machine Image (AMI) from an existing EC2 instance named **devops-ec2**. The AMI acts as a reusable machine template that can be used for backup, disaster recovery, environment replication, and rapid scaling.

Once the AMI is created, a new EC2 instance named **devops-ec2-new** is launched using the newly created AMI.

---

## 🎯 Objectives

✅ Create an AMI named **devops-ec2-ami**

✅ Use the existing EC2 instance **devops-ec2** as the source

✅ Launch a new EC2 instance named **devops-ec2-new**

✅ Output the AMI ID

✅ Output the new EC2 Instance ID

---

## 🏗️ Architecture

```text
Existing EC2 Instance
     (devops-ec2)
            │
            ▼
   📦 Create AMI
  (devops-ec2-ami)
            │
            ▼
     🖥️ New EC2
  (devops-ec2-new)
```

---

# 📌 Understanding Resource vs Data Source

Before implementing the solution, it is important to understand when to use a **resource** and when to use a **data source**.

### ✅ Use Resource Reference

If the EC2 instance is created and managed by the current Terraform configuration:

```hcl
resource "aws_instance" "ec2" {
  ...
}
```

Use:

```hcl
source_instance_id = aws_instance.ec2.id
```

Terraform already manages the instance and knows its ID.

---

### ✅ Use Data Source

If the EC2 instance already exists in AWS and is **not managed by the current Terraform code**, use a data source:

```hcl
data "aws_instance" "ec2" {
  filter {
    name   = "tag:Name"
    values = ["devops-ec2"]
  }

  instance_state_names = ["running"]
}
```

Then:

```hcl
source_instance_id = data.aws_instance.ec2.id
```

This allows Terraform to locate and use an existing resource.

---

### 🎯 For This Task

The task states:

> "They have an existing EC2 instance named devops-ec2"

This usually indicates that the instance already exists and should be looked up using a **data source**, which is the recommended approach for this lab.

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
└── outputs.tf
```

---

# ⚙️ main.tf

## Step 1: Locate Existing EC2 Instance

```hcl
data "aws_instance" "ec2" {

  filter {
    name   = "tag:Name"
    values = ["devops-ec2"]
  }

  instance_state_names = ["running"]
}
```

---

## Step 2: Create AMI from Existing Instance

```hcl
resource "aws_ami_from_instance" "devops_ami" {
  name               = "devops-ec2-ami"
  source_instance_id = data.aws_instance.ec2.id
}
```

---

## Step 3: Launch New Instance Using AMI

```hcl
resource "aws_instance" "devops_ec2_new" {
  ami           = aws_ami_from_instance.devops_ami.id
  instance_type = "t2.micro"

  tags = {
    Name = "devops-ec2-new"
  }
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_ami_id" {
  value = aws_ami_from_instance.devops_ami.id
}

output "KKE_new_instance_id" {
  value = aws_instance.devops_ec2_new.id
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

### 3️⃣ Review Execution Plan

```bash
terraform plan
```

Expected:

```text
+ aws_ami_from_instance.devops_ami
+ aws_instance.devops_ec2_new
```

---

### 4️⃣ Apply Configuration

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify Existing EC2 Instance

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=devops-ec2" \
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

## ✅ Verify AMI Creation

```bash
aws ec2 describe-images \
--owners self \
--filters "Name=name,Values=devops-ec2-ami" \
--query "Images[*].[ImageId,Name,State]" \
--output table
```

Expected:

```text
---------------------------------------------------
|               DescribeImages                     |
+---------------+------------------+-------------+
| ami-xxxxxxxxx | devops-ec2-ami   | available   |
+---------------+------------------+-------------+
```

✔️ Verify the AMI state is **available**

---

## ✅ Verify New EC2 Instance

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=devops-ec2-new" \
--query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
--output table
```

Expected:

```text
------------------------------------------------
|              DescribeInstances               |
+----------------------+-----------------------+
| i-yyyyyyyyyyyyyyyyy  | running               |
+----------------------+-----------------------+
```

---

## ✅ Verify AMI Used by New Instance

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=devops-ec2-new" \
--query "Reservations[*].Instances[*].[ImageId]" \
--output table
```

Verify that the AMI ID matches:

```text
ami-xxxxxxxxxxxx
```

returned by:

```bash
terraform output KKE_ami_id
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
KKE_ami_id = "ami-xxxxxxxxxxxx"

KKE_new_instance_id = "i-yyyyyyyyyyyyyyyy"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_ami_from_instance.devops_ami
aws_instance.devops_ec2_new
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

🎉 This confirms Terraform state and AWS infrastructure are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 Terraform Data Sources

🔹 Resource vs Data Source Usage

🔹 Custom AMI Creation

🔹 EC2 Instance Cloning

🔹 Infrastructure as Code (IaC)

🔹 Terraform Dependencies

🔹 Terraform State Management

🔹 AWS Resource Verification

---

# 🏆 Outcome

Successfully created a reusable custom AMI from an existing EC2 instance and launched a new EC2 instance using that AMI. This approach is commonly used for backup strategies, rapid environment provisioning, disaster recovery planning, and horizontal scaling in production cloud environments. 🚀☁️📦🖥️
