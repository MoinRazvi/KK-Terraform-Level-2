# � Task-17: Access Secrets Manager with IAM Role Using Terraform

## 📖 Overview

In this task, we create a secret in AWS Secrets Manager, provision an IAM Role trusted by EC2, and attach an **inline IAM policy** that allows retrieval of the secret.

This is a common production pattern where applications running on EC2 securely retrieve credentials from Secrets Manager instead of storing them in configuration files or source code.

---

## 🎯 Objectives

✅ Create Secrets Manager Secret **devops-app-secret**

✅ Store secret value:

```json
{"db_user":"admin","db_pass":"supersecret"}
```

✅ Create IAM Role **devops-app-role**

✅ Configure EC2 as trusted entity

✅ Create Inline IAM Policy **devops-app-policy**

✅ Allow **secretsmanager:GetSecretValue**

✅ Restrict access to only the created secret

---

# 🏗️ Architecture

```text
               🖥️ EC2 Instance
                       │
                       ▼
                 🎭 IAM Role
               devops-app-role
                       │
                       ▼
             📜 Inline IAM Policy
              devops-app-policy
                       │
                       ▼
            🔐 Secrets Manager Secret
              devops-app-secret
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
variable "KKE_SECRET_NAME" {
  type = string
}

variable "KKE_SECRET_VALUE" {
  type = string
}

variable "KKE_ROLE_NAME" {
  type = string
}

variable "KKE_POLICY_NAME" {
  type = string
}
```

---

# 📋 terraform.tfvars

```hcl
KKE_SECRET_NAME = "devops-app-secret"

KKE_SECRET_VALUE = "{\"db_user\":\"admin\",\"db_pass\":\"supersecret\"}"

KKE_ROLE_NAME = "devops-app-role"

KKE_POLICY_NAME = "devops-app-policy"
```

---

# ⚙️ main.tf

## Create Secret

```hcl
resource "aws_secretsmanager_secret" "app_secret" {
  name = var.KKE_SECRET_NAME
}
```

---

## Create Secret Value

```hcl
resource "aws_secretsmanager_secret_version" "app_secret_value" {
  secret_id     = aws_secretsmanager_secret.app_secret.id
  secret_string = var.KKE_SECRET_VALUE
}
```

---

## Create IAM Role

```hcl
resource "aws_iam_role" "app_role" {
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

## Create Inline IAM Policy

> ⚠️ The task specifically asks for an **inline IAM policy**, therefore use `aws_iam_role_policy` instead of `aws_iam_policy` and `aws_iam_role_policy_attachment`.

```hcl
resource "aws_iam_role_policy" "app_policy" {
  name = var.KKE_POLICY_NAME

  role = aws_iam_role.app_role.id

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "secretsmanager:GetSecretValue"
      ]

      Resource = aws_secretsmanager_secret.app_secret.arn
    }]
  })
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_secret_name" {
  value = aws_secretsmanager_secret.app_secret.name
}

output "KKE_role_name" {
  value = aws_iam_role.app_role.name
}

output "KKE_policy_name" {
  value = aws_iam_role_policy.app_policy.name
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

Expected:

```text
+ aws_secretsmanager_secret.app_secret
+ aws_secretsmanager_secret_version.app_secret_value
+ aws_iam_role.app_role
+ aws_iam_role_policy.app_policy
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

## ✅ Verify Secret

```bash
aws secretsmanager describe-secret \
--secret-id devops-app-secret
```

Expected:

```text
Name: devops-app-secret
```

---

## ✅ Verify Secret Value

```bash
aws secretsmanager get-secret-value \
--secret-id devops-app-secret
```

Expected:

```json
{
  "db_user":"admin",
  "db_pass":"supersecret"
}
```

---

## ✅ Verify IAM Role

```bash
aws iam get-role \
--role-name devops-app-role
```

Expected:

```text
RoleName: devops-app-role
```

---

## ✅ Verify Inline Policy

```bash
aws iam list-role-policies \
--role-name devops-app-role
```

Expected:

```text
devops-app-policy
```

---

## ✅ Verify Policy Content

```bash
aws iam get-role-policy \
--role-name devops-app-role \
--policy-name devops-app-policy
```

Verify:

```json
"Action": "secretsmanager:GetSecretValue"
```

and verify the policy is restricted to:

```text
devops-app-secret
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
KKE_secret_name = "devops-app-secret"

KKE_role_name = "devops-app-role"

KKE_policy_name = "devops-app-policy"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_secretsmanager_secret.app_secret
aws_secretsmanager_secret_version.app_secret_value
aws_iam_role.app_role
aws_iam_role_policy.app_policy
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

🔹 AWS Secrets Manager

🔹 Secret Version Management

🔹 IAM Roles

🔹 Inline IAM Policies

🔹 EC2 Trust Relationships

🔹 Least Privilege Access

🔹 Secrets Retrieval Security

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned a Secrets Manager secret, created an EC2-trusted IAM role, and attached an inline IAM policy that grants permission to retrieve only the specified secret. This pattern is widely used in production environments to securely manage database credentials, API keys, and application secrets without exposing sensitive information in code or configuration files. 🚀🔐☁️
