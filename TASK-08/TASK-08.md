# 🚀 Task-08: Sync Data to S3 Bucket with Terraform

## 📖 Overview

As part of a data migration project, the objective is to create a new private S3 bucket and migrate all data from an existing S3 bucket into the newly provisioned bucket. After migration, both buckets must contain identical data to ensure consistency and integrity.

Terraform is used to provision the destination bucket, while the AWS CLI is used to perform the actual data synchronization between the buckets.

---

## 🎯 Objectives

✅ Create a new private S3 bucket **nautilus-sync-1158**

✅ Store the bucket name in a variable named **KKE_BUCKET**

✅ Migrate all data from **nautilus-s3-24453**

✅ Ensure both buckets contain identical data

✅ Output the bucket name and ACL

---

## 🏗️ Architecture

```text
                 Source Bucket
              (nautilus-s3-24453)
                        │
                        │ aws s3 sync
                        ▼
              Destination Bucket
             (nautilus-sync-1158)
                        │
                        ▼
               Data Verification
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
variable "KKE_BUCKET" {
  description = "Destination S3 Bucket Name"
  type        = string
}
```

---

# 📋 terraform.tfvars

```hcl
KKE_BUCKET = "nautilus-sync-1158"
```

---

# ⚙️ main.tf

> **Note:** The source bucket configuration already exists in the lab environment and is retained below.

```hcl
# Existing Source Bucket

resource "aws_s3_bucket" "wordpress_bucket" {
  bucket = "nautilus-s3-24453"
}

resource "aws_s3_bucket_acl" "wordpress_bucket_acl" {
  bucket = aws_s3_bucket.wordpress_bucket.id
  acl    = "private"
}

# Destination Bucket

resource "aws_s3_bucket" "sync_bucket" {
  bucket = var.KKE_BUCKET
}

resource "aws_s3_bucket_acl" "sync_bucket_acl" {
  bucket = aws_s3_bucket.sync_bucket.id
  acl    = "private"
}
```

---

# 📤 outputs.tf

```hcl
output "new_kke_bucket_name" {
  value = aws_s3_bucket.sync_bucket.bucket
}

output "new_kke_bucket_acl" {
  value = aws_s3_bucket_acl.sync_bucket_acl.acl
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
+ aws_s3_bucket.sync_bucket
+ aws_s3_bucket_acl.sync_bucket_acl
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

# 📦 Data Migration

After the bucket is created, sync all data from the source bucket to the destination bucket.

```bash
aws s3 sync s3://nautilus-s3-24453 s3://nautilus-sync-1158
```

Expected:

```text
upload: ...
upload: ...
upload: ...
```

---

# 🔍 Verification Steps

## ✅ Verify Source Bucket Exists

```bash
aws s3 ls | grep nautilus-s3-24453
```

Expected:

```text
nautilus-s3-24453
```

---

## ✅ Verify Destination Bucket Exists

```bash
aws s3 ls | grep nautilus-sync-1158
```

Expected:

```text
nautilus-sync-1158
```

---

## ✅ Verify Source Bucket Object Count

```bash
aws s3 ls s3://nautilus-s3-24453 --recursive | wc -l
```

Example:

```text
25
```

---

## ✅ Verify Destination Bucket Object Count

```bash
aws s3 ls s3://nautilus-sync-1158 --recursive | wc -l
```

Expected:

```text
25
```

Object count should match the source bucket.

---

## ✅ Verify Data Consistency

Run a dry-run synchronization:

```bash
aws s3 sync \
s3://nautilus-s3-24453 \
s3://nautilus-sync-1158 \
--dryrun
```

Expected:

```text
(no output)
```

✔️ No output means both buckets contain identical data.

---

## ✅ Verify Bucket ACL

```bash
aws s3api get-bucket-acl \
--bucket nautilus-sync-1158
```

Verify that the bucket remains private.

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
new_kke_bucket_name = "nautilus-sync-1158"

new_kke_bucket_acl = "private"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_s3_bucket.sync_bucket
aws_s3_bucket_acl.sync_bucket_acl
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

🔹 Amazon S3 Bucket Provisioning

🔹 Private Bucket Configuration

🔹 AWS CLI Data Synchronization

🔹 Data Migration Validation

🔹 Infrastructure as Code (IaC)

🔹 Terraform Variables & Outputs

🔹 Terraform State Management

🔹 AWS Storage Operations

---

# 🏆 Outcome

Successfully provisioned a new private S3 bucket using Terraform and migrated all data from the existing source bucket using AWS CLI synchronization. Verified that both buckets contain identical data, ensuring a successful and consistent migration process. 🚀☁️📦🔄
