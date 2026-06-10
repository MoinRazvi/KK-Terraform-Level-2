# � Task-15: Attach IAM Policy for DynamoDB Access Using Terraform

## 📖 Overview

In this task, we provision a DynamoDB table and implement fine-grained IAM access control by creating an IAM Role and attaching a read-only IAM Policy.

The policy grants access only to the specific DynamoDB table and only for read operations:

* `dynamodb:GetItem`
* `dynamodb:Scan`
* `dynamodb:Query`

This follows the AWS **Principle of Least Privilege**, ensuring that consumers can read data without modifying or deleting it.

---

## 🎯 Objectives

✅ Create DynamoDB Table **devops-table**

✅ Create IAM Role **devops-role**

✅ Create IAM Policy **devops-readonly-policy**

✅ Grant only:

* GetItem
* Scan
* Query

✅ Restrict permissions to the specific table

✅ Attach policy to role

✅ Output table, role, and policy names

---

# 🏗️ Architecture

```text
             🗄️ DynamoDB Table
                devops-table
                       ▲
                       │
         Read Only Access (Get/Scan/Query)
                       │
                       ▼
               📜 IAM Policy
          devops-readonly-policy
                       │
                       ▼
                🎭 IAM Role
                devops-role
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
variable "KKE_TABLE_NAME" {
  description = "DynamoDB Table Name"
  type        = string
}

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
KKE_TABLE_NAME  = "devops-table"
KKE_ROLE_NAME   = "devops-role"
KKE_POLICY_NAME = "devops-readonly-policy"
```

---

# ⚙️ main.tf

## Create DynamoDB Table

```hcl
resource "aws_dynamodb_table" "devops_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"

  hash_key = "id"

  attribute {
    name = "id"
    type = "S"
  }
}
```

---

## Create IAM Role

```hcl
resource "aws_iam_role" "devops_role" {
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
resource "aws_iam_policy" "devops_readonly_policy" {
  name = var.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:Query"
      ]

      Resource = aws_dynamodb_table.devops_table.arn
    }]
  })
}
```

---

## Attach Policy to Role

```hcl
resource "aws_iam_role_policy_attachment" "policy_attachment" {
  role       = aws_iam_role.devops_role.name
  policy_arn = aws_iam_policy.devops_readonly_policy.arn
}
```

---

# 📤 outputs.tf

```hcl
output "kke_dynamodb_table" {
  value = aws_dynamodb_table.devops_table.name
}

output "kke_iam_role_name" {
  value = aws_iam_role.devops_role.name
}

output "kke_iam_policy_name" {
  value = aws_iam_policy.devops_readonly_policy.name
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
+ aws_dynamodb_table.devops_table
+ aws_iam_role.devops_role
+ aws_iam_policy.devops_readonly_policy
+ aws_iam_role_policy_attachment.policy_attachment
```

---

### 4️⃣ Apply Configuration

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify DynamoDB Table

```bash
aws dynamodb describe-table \
--table-name devops-table
```

Expected:

```text
TableStatus : ACTIVE
```

---

## ✅ Verify IAM Role

```bash
aws iam get-role \
--role-name devops-role
```

Expected:

```text
RoleName : devops-role
```

---

## ✅ Verify IAM Policy

```bash
aws iam list-policies \
--scope Local
```

Expected:

```text
devops-readonly-policy
```

---

## ✅ Verify Policy Attachment

```bash
aws iam list-attached-role-policies \
--role-name devops-role
```

Expected:

```text
devops-readonly-policy
```

---

## ✅ Verify Allowed Actions

```bash
aws iam get-policy-version \
--policy-arn <policy-arn> \
--version-id v1
```

Verify:

```json
"dynamodb:GetItem"
"dynamodb:Scan"
"dynamodb:Query"
```

are present.

Verify that:

```json
"dynamodb:PutItem"
"dynamodb:DeleteItem"
"dynamodb:UpdateItem"
```

are NOT present.

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
kke_dynamodb_table = "devops-table"

kke_iam_role_name = "devops-role"

kke_iam_policy_name = "devops-readonly-policy"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_dynamodb_table.devops_table
aws_iam_role.devops_role
aws_iam_policy.devops_readonly_policy
aws_iam_role_policy_attachment.policy_attachment
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

🔹 DynamoDB Table Creation

🔹 IAM Roles

🔹 IAM Policies

🔹 IAM Role Policy Attachments

🔹 Principle of Least Privilege

🔹 Fine-Grained Access Control

🔹 Terraform Variables & Outputs

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned a DynamoDB table, IAM role, and a read-only IAM policy that grants access only to the specific table. This implementation follows AWS security best practices by limiting permissions to only the required read operations and restricting access to a single DynamoDB resource. 🚀🗄️🔐☁️
