# 🚀 Task-14: Provision IAM User with Terraform

## 📖 Overview

In this task, we create an IAM user and leverage Terraform's **local-exec provisioner** to execute a local command after successful resource creation.

The provisioner writes a confirmation message to a log file, demonstrating how Terraform can interact with the local system during infrastructure provisioning.

---

## 🎯 Objectives

✅ Create IAM User **iamuser_kirsty**

✅ Use a **local-exec** provisioner

✅ Create log file **KKE_user_created.log**

✅ Write confirmation message to the file

✅ Output IAM User Name

---

# 🏗️ Architecture

```text id="ldf4vw"
          Terraform Apply
                  │
                  ▼
           👤 IAM User
         iamuser_kirsty
                  │
                  ▼
      ⚙️ local-exec Provisioner
                  │
                  ▼
      📄 KKE_user_created.log
```

---

# 📂 Project Structure

```bash id="22usfj"
terraform/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```

---

# 📝 variables.tf

```hcl id="h6m9q7"
variable "KKE_USER_NAME" {
  description = "IAM User Name"
  type        = string
}
```

---

# 📋 terraform.tfvars

```hcl id="gln6hn"
KKE_USER_NAME = "iamuser_kirsty"
```

---

# ⚙️ main.tf

```hcl id="qumq8q"
resource "aws_iam_user" "iam_user" {
  name = var.KKE_USER_NAME

  provisioner "local-exec" {
    command = "echo 'KKE iamuser_kirsty has been created successfully!' > /home/bob/terraform/KKE_user_created.log"
  }
}
```

> 💡 Alternative (more dynamic):

```hcl id="3s5a7v"
resource "aws_iam_user" "iam_user" {
  name = var.KKE_USER_NAME

  provisioner "local-exec" {
    command = "echo 'KKE ${var.KKE_USER_NAME} has been created successfully!' > /home/bob/terraform/KKE_user_created.log"
  }
}
```

If the validator checks the exact message, use the first version.

---

# 📤 outputs.tf

```hcl id="wf6hfd"
output "kke_iam_user_name" {
  value = aws_iam_user.iam_user.name
}
```

---

# 🚀 Deployment Steps

### 1️⃣ Initialize Terraform

```bash id="lnl0m5"
terraform init
```

Expected:

```text id="pbzclx"
Terraform has been successfully initialized!
```

---

### 2️⃣ Validate Configuration

```bash id="dzd76g"
terraform validate
```

Expected:

```text id="z8bg0h"
Success! The configuration is valid.
```

---

### 3️⃣ Review Execution Plan

```bash id="yd2lbn"
terraform plan
```

Expected:

```text id="u3j78y"
+ aws_iam_user.iam_user
```

---

### 4️⃣ Apply Configuration

```bash id="75b4el"
terraform apply -auto-approve
```

Expected:

```text id="2w0lm8"
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify IAM User

```bash id="f8fhv8"
aws iam get-user \
--user-name iamuser_kirsty
```

Expected:

```text id="9c4g2l"
UserName: iamuser_kirsty
```

---

## ✅ Verify Log File Exists

```bash id="3sgt1o"
ls -l /home/bob/terraform/KKE_user_created.log
```

Expected:

```text id="0wevdt"
KKE_user_created.log
```

---

## ✅ Verify Log File Content

```bash id="d2wjbs"
cat /home/bob/terraform/KKE_user_created.log
```

Expected:

```text id="tuy73s"
KKE iamuser_kirsty has been created successfully!
```

---

## ✅ Verify Terraform Output

```bash id="7i3n5o"
terraform output
```

Expected:

```text id="r5hnn0"
kke_iam_user_name = "iamuser_kirsty"
```

---

## ✅ Verify Terraform State

```bash id="4lgmrz"
terraform state list
```

Expected:

```text id="xig5om"
aws_iam_user.iam_user
```

---

## ✅ Final Validation (Mandatory)

```bash id="3v4q2f"
terraform plan
```

Expected:

```text id="cpq5it"
No changes. Your infrastructure matches the configuration.
```

🎉 This confirms Terraform state and AWS infrastructure are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 IAM User Provisioning

🔹 Terraform Provisioners

🔹 local-exec Provisioner

🔹 Terraform Variables

🔹 Terraform Outputs

🔹 Local Command Execution

🔹 Infrastructure as Code (IaC)

🔹 Post-Provisioning Automation

---

# 🏆 Outcome

Successfully provisioned an IAM user and executed a local command using Terraform's **local-exec provisioner** to generate a confirmation log file. This demonstrates how Terraform can integrate infrastructure provisioning with local automation tasks during deployment workflows. 🚀👤⚙️📄
