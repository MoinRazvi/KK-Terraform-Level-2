# 🚀 Task-09: Prevent S3 Bucket Deletion via Terraform

## 📖 Overview

In production environments, certain resources such as S3 buckets may contain critical application data, backups, logs, or compliance-related information. Accidentally deleting these resources can lead to data loss and service disruptions.

Terraform provides the **`lifecycle`** block with the **`prevent_destroy`** argument to protect resources from accidental deletion.

In this task, we create an S3 bucket and apply a lifecycle policy that prevents Terraform from destroying it.

---

## 🎯 Objectives

✅ Create an S3 bucket named **datacenter-s3-30309**

✅ Use Terraform variable **KKE_BUCKET_NAME**

✅ Protect the bucket using **prevent_destroy**

✅ Output the bucket name

---

## 🏗️ Architecture

```text id="hmga6s"
             Terraform
                  │
                  ▼
        📦 S3 Bucket
      datacenter-s3-30309
                  │
                  ▼
       🔒 prevent_destroy
                  │
                  ▼
    Protection Against Accidental
         Terraform Deletion
```

---

# 📂 Project Structure

```bash id="6z4t7f"
terraform/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```

---

# 📝 variables.tf

```hcl id="8k6msd"
variable "KKE_BUCKET_NAME" {
  description = "S3 Bucket Name"
  type        = string
}
```

---

# 📋 terraform.tfvars

```hcl id="jy4tqv"
KKE_BUCKET_NAME = "datacenter-s3-30309"
```

---

# ⚙️ main.tf

```hcl id="42c6ws"
resource "aws_s3_bucket" "datacenter_bucket" {
  bucket = var.KKE_BUCKET_NAME

  lifecycle {
    prevent_destroy = true
  }
}
```

---

# 📤 outputs.tf

```hcl id="f0z39h"
output "s3_bucket_name" {
  value = aws_s3_bucket.datacenter_bucket.bucket
}
```

---

# 🚀 Deployment Steps

### 1️⃣ Initialize Terraform

```bash id="7ssx4q"
terraform init
```

Expected:

```text id="72i3n0"
Terraform has been successfully initialized!
```

---

### 2️⃣ Validate Configuration

```bash id="2chjlwm"
terraform validate
```

Expected:

```text id="86k59v"
Success! The configuration is valid.
```

---

### 3️⃣ Review Execution Plan

```bash id="7kqfph"
terraform plan
```

Expected:

```text id="ymq8ul"
+ aws_s3_bucket.datacenter_bucket
```

---

### 4️⃣ Deploy Infrastructure

```bash id="f4xlnx"
terraform apply -auto-approve
```

Expected:

```text id="dhn4q9"
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify Bucket Creation

```bash id="um8vzt"
aws s3 ls | grep datacenter-s3-30309
```

Expected:

```text id="c08oqn"
datacenter-s3-30309
```

---

## ✅ Verify Terraform Output

```bash id="sd4xdi"
terraform output
```

Expected:

```text id="uq2xqe"
s3_bucket_name = "datacenter-s3-30309"
```

---

## ✅ Verify Lifecycle Rule Exists

Display Terraform configuration:

```bash id="m4a54u"
terraform state show aws_s3_bucket.datacenter_bucket
```

Confirm the bucket is managed by Terraform and protected by the lifecycle configuration in code.

---

## 🔒 Test prevent_destroy Protection

Attempt:

```bash id="fysf6j"
terraform destroy
```

Expected:

```text id="pxs30h"
Error: Instance cannot be destroyed

Resource aws_s3_bucket.datacenter_bucket has lifecycle.prevent_destroy set
```

✔️ This confirms Terraform protection is working correctly.

> ⚠️ Do not force removal unless specifically required by the task.

---

## ✅ Verify Terraform State

```bash id="2n2h3t"
terraform state list
```

Expected:

```text id="3v79xv"
aws_s3_bucket.datacenter_bucket
```

---

## ✅ Final Validation (Mandatory)

Run:

```bash id="dlrxxu"
terraform plan
```

Expected:

```text id="7rlulx"
No changes. Your infrastructure matches the configuration.
```

🎉 This confirms Terraform state and AWS infrastructure are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 Terraform Lifecycle Meta-Arguments

🔹 Resource Protection

🔹 S3 Bucket Management

🔹 Infrastructure as Code (IaC)

🔹 Terraform Variables

🔹 Terraform Outputs

🔹 State Management

🔹 Production Safety Controls

---

# 🏆 Outcome

Successfully provisioned an S3 bucket and protected it using Terraform's **prevent_destroy** lifecycle rule. This ensures critical storage resources cannot be accidentally deleted through Terraform operations, improving infrastructure safety and operational reliability. 🚀🔒☁️📦
