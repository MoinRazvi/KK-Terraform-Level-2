# � Task-16: Send Notifications from IAM Events to SNS Using Terraform

## 📖 Overview

In this task, we configure an Amazon SNS Topic and create an IAM Role with a policy that allows EC2 instances to publish messages to the topic.

Instead of using variables, the task explicitly requires the values to be defined in **locals.tf**, which is commonly used for fixed configuration values that do not change across environments.

---

## 🎯 Objectives

✅ Create SNS Topic **devops-sns-topic**

✅ Create IAM Role **devops-sns-role**

✅ Configure EC2 as the trusted entity

✅ Create IAM Policy **devops-sns-policy**

✅ Allow **sns:Publish** to the specific SNS topic

✅ Attach policy to role

✅ Output topic, role, and policy names

---

# 🏗️ Architecture

```text
           🖥️ EC2 Instance
                   │
                   ▼
            🎭 IAM Role
          devops-sns-role
                   │
                   ▼
            📜 IAM Policy
         devops-sns-policy
                   │
                   ▼
              📢 SNS Topic
          devops-sns-topic
```

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
├── locals.tf
└── outputs.tf
```

---

# 📄 locals.tf

```hcl
locals {
  KKE_SNS_TOPIC_NAME = "devops-sns-topic"
  KKE_ROLE_NAME      = "devops-sns-role"
  KKE_POLICY_NAME    = "devops-sns-policy"
}
```

---

# ⚙️ main.tf

## Create SNS Topic

```hcl
resource "aws_sns_topic" "devops_topic" {
  name = local.KKE_SNS_TOPIC_NAME
}
```

---

## Create IAM Role

```hcl
resource "aws_iam_role" "devops_role" {
  name = local.KKE_ROLE_NAME

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
resource "aws_iam_policy" "devops_policy" {
  name = local.KKE_POLICY_NAME

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "sns:Publish"
      ]

      Resource = aws_sns_topic.devops_topic.arn
    }]
  })
}
```

---

## Attach Policy to Role

```hcl
resource "aws_iam_role_policy_attachment" "policy_attachment" {
  role       = aws_iam_role.devops_role.name
  policy_arn = aws_iam_policy.devops_policy.arn
}
```

---

# 📤 outputs.tf

```hcl
output "kke_sns_topic_name" {
  value = aws_sns_topic.devops_topic.name
}

output "kke_role_name" {
  value = aws_iam_role.devops_role.name
}

output "kke_policy_name" {
  value = aws_iam_policy.devops_policy.name
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
+ aws_sns_topic.devops_topic
+ aws_iam_role.devops_role
+ aws_iam_policy.devops_policy
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

## ✅ Verify SNS Topic

```bash
aws sns list-topics
```

Verify:

```text
devops-sns-topic
```

---

## ✅ Verify IAM Role

```bash
aws iam get-role \
--role-name devops-sns-role
```

Expected:

```text
RoleName: devops-sns-role
```

---

## ✅ Verify IAM Policy

```bash
aws iam list-policies \
--scope Local
```

Expected:

```text
devops-sns-policy
```

---

## ✅ Verify Policy Attachment

```bash
aws iam list-attached-role-policies \
--role-name devops-sns-role
```

Expected:

```text
devops-sns-policy
```

---

## ✅ Verify SNS Publish Permission

```bash
aws iam get-policy-version \
--policy-arn <policy-arn> \
--version-id v1
```

Verify:

```json
"sns:Publish"
```

and ensure the resource is restricted to:

```text
devops-sns-topic
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
kke_sns_topic_name = "devops-sns-topic"

kke_role_name = "devops-sns-role"

kke_policy_name = "devops-sns-policy"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_sns_topic.devops_topic
aws_iam_role.devops_role
aws_iam_policy.devops_policy
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

🔹 Amazon SNS

🔹 IAM Roles

🔹 IAM Policies

🔹 IAM Trust Relationships

🔹 SNS Publish Permissions

🔹 Terraform Locals

🔹 IAM Role Policy Attachments

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned an SNS Topic, IAM Role, and IAM Policy that allows EC2 instances to publish messages to the SNS topic. This setup enables secure inter-service communication while following AWS security best practices and the principle of least privilege. 🚀📢🔐☁️
