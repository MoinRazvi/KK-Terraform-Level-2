# 🚀 Task-13: Attach IAM Role with Inline Policy Using Terraform

## 📖 Overview

In this task, we implement IAM-based access control by creating an IAM Role and an IAM Policy that grants permission to list EC2 instances. The policy is then attached to the role, enabling controlled access to AWS resources.

This follows the principle of least privilege by granting only the permissions required for the intended operation.

---

## 🎯 Objectives

✅ Create IAM Role **datacenter-role**

✅ Create IAM Policy **datacenter-policy**

✅ Allow **ec2:DescribeInstances**

✅ Attach Policy to Role

✅ Output Role Name

✅ Output Policy Name

---

# 🏗️ Architecture

```text
          👤 IAM Role
       datacenter-role
               │
               ▼
        📜 IAM Policy
      datacenter-policy
               │
               ▼
      ec2:DescribeInstances
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
variable "KKE_ROLE_NAME" {
  description = "IAM Role Name"
  type        = string
}

variable "KKE_POLICY_NAME" {
  description = "IAM Policy Name"
  type        = string
}
```

---

# 📋 terraform.tfvars

```hcl
KKE_ROLE_NAME   = "datacenter-role"
KKE_POLICY_NAME = "datacenter-policy"
```

---

# ⚙️ main.tf

## Create IAM Role

```hcl
resource "aws_iam_role" "datacenter_role" {
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
resource "aws_iam_policy" "datacenter_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "ec2:DescribeInstances"
      ]

      Resource = "*"
    }]
  })
}
```

---

## Attach Policy to Role

```hcl
resource "aws_iam_role_policy_attachment" "policy_attach" {
  role       = aws_iam_role.datacenter_role.name
  policy_arn = aws_iam_policy.datacenter_policy.arn
}
```

---

# 📤 outputs.tf

```hcl
output "kke_iam_role_name" {
  value = aws_iam_role.datacenter_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.datacenter_policy.name
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
+ aws_iam_role.datacenter_role
+ aws_iam_policy.datacenter_policy
+ aws_iam_role_policy_attachment.policy_attach
```

---

### 4️⃣ Apply Configuration

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify IAM Role

```bash
aws iam get-role \
--role-name datacenter-role
```

Expected:

```text
RoleName: datacenter-role
```

---

## ✅ Verify IAM Policy

```bash
aws iam list-policies \
--scope Local
```

Expected:

```text
datacenter-policy
```

---

## ✅ Verify Policy Attachment

```bash
aws iam list-attached-role-policies \
--role-name datacenter-role
```

Expected:

```text
datacenter-policy
```

---

## ✅ Verify Policy Permissions

```bash
aws iam get-policy-version \
--policy-arn <policy-arn> \
--version-id v1
```

Expected:

```json
{
  "Action": "ec2:DescribeInstances"
}
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
kke_iam_role_name = "datacenter-role"

kke_iam_policy_name = "datacenter-policy"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_iam_role.datacenter_role
aws_iam_policy.datacenter_policy
aws_iam_role_policy_attachment.policy_attach
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

🎉 This confirms Terraform state and AWS infrastructure are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 IAM Roles

🔹 IAM Policies

🔹 Policy Attachments

🔹 EC2 Read-Only Permissions

🔹 Principle of Least Privilege

🔹 Terraform Variables

🔹 Terraform Outputs

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned an IAM Role and IAM Policy using Terraform and attached the policy to the role, enabling secure access to EC2 instance metadata through the `ec2:DescribeInstances` permission. This demonstrates a common IAM access-control pattern used in production AWS environments. 🚀🔐☁️📜
