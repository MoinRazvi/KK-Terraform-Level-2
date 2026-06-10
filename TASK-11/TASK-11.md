# 🚀 Task-11: Implement S3 Lifecycle Management Policy Using Terraform

## 📖 Overview

As part of storage optimization and cost management, the DevOps team wants to automatically transition objects to a lower-cost storage class after a specified period and eventually delete them when they are no longer required.

In this task, we will:

✅ Create an S3 bucket

✅ Enable bucket versioning

✅ Configure a lifecycle rule

✅ Transition objects to **STANDARD_IA** after 30 days

✅ Delete objects after 365 days

---

## 🎯 Objectives

✅ Create S3 bucket **nautilus-lifecycle-2645**

✅ Enable Versioning

✅ Create lifecycle rule **nautilus-lifecycle-rule**

✅ Transition objects to **STANDARD_IA** after **30 days**

✅ Expire objects after **365 days**

✅ Output bucket name

---

# 🏗️ Architecture

```text
               📦 S3 Bucket
          nautilus-lifecycle-2645
                      │
                      ▼
            🔄 Versioning Enabled
                      │
                      ▼
            📋 Lifecycle Rule
      (nautilus-lifecycle-rule)
                      │
         ┌────────────┴────────────┐
         ▼                         ▼
  Day 30: STANDARD_IA      Day 365: Delete
```

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
└── outputs.tf
```

---

# ⚙️ main.tf

```hcl
resource "aws_s3_bucket" "lifecycle_bucket" {
  bucket = "nautilus-lifecycle-2645"
}

resource "aws_s3_bucket_versioning" "bucket_versioning" {
  bucket = aws_s3_bucket.lifecycle_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_lifecycle_configuration" "lifecycle_rule" {
  bucket = aws_s3_bucket.lifecycle_bucket.id

  rule {
    id     = "nautilus-lifecycle-rule"
    status = "Enabled"

    filter {
      prefix = ""
    }

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    expiration {
      days = 365
    }
  }

  depends_on = [
    aws_s3_bucket_versioning.bucket_versioning
  ]
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_bucket_name" {
  value = aws_s3_bucket.lifecycle_bucket.bucket
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
+ aws_s3_bucket.lifecycle_bucket
+ aws_s3_bucket_versioning.bucket_versioning
+ aws_s3_bucket_lifecycle_configuration.lifecycle_rule
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

## ✅ Verify Bucket Creation

```bash
aws s3 ls | grep nautilus-lifecycle-2645
```

Expected:

```text
nautilus-lifecycle-2645
```

---

## ✅ Verify Versioning

```bash
aws s3api get-bucket-versioning \
--bucket nautilus-lifecycle-2645
```

Expected:

```json
{
  "Status": "Enabled"
}
```

---

## ✅ Verify Lifecycle Rule

```bash
aws s3api get-bucket-lifecycle-configuration \
--bucket nautilus-lifecycle-2645
```

Expected:

```text
ID: nautilus-lifecycle-rule
Transition: 30 Days → STANDARD_IA
Expiration: 365 Days
```

---

## ✅ Verify Lifecycle Rule Details

```bash
aws s3api get-bucket-lifecycle-configuration \
--bucket nautilus-lifecycle-2645 \
--query "Rules[*].[ID,Status]"
```

Expected:

```text
------------------------------------
| nautilus-lifecycle-rule | Enabled |
------------------------------------
```

---

## ✅ Verify Terraform Output

```bash
terraform output
```

Expected:

```text
KKE_bucket_name = "nautilus-lifecycle-2645"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_s3_bucket.lifecycle_bucket
aws_s3_bucket_versioning.bucket_versioning
aws_s3_bucket_lifecycle_configuration.lifecycle_rule
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

🔹 Amazon S3 Lifecycle Policies

🔹 Storage Class Transitions

🔹 STANDARD_IA Storage

🔹 Object Expiration

🔹 S3 Versioning

🔹 Terraform Resource Dependencies

🔹 Cost Optimization Strategies

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned an S3 bucket with versioning enabled and implemented a lifecycle management policy that automatically transitions objects to **STANDARD_IA** after 30 days and deletes them after 365 days. This approach helps optimize storage costs while maintaining efficient object lifecycle governance. 🚀☁️📦💰
