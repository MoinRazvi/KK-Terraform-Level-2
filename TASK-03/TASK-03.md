## 🚀  Task-03: Replace Existing EC2 Instance via Terraform

### 📖 Overview

In production environments, there are situations where an EC2 instance must be recreated without modifying the Terraform configuration. Terraform provides the **`-replace`** option to forcefully destroy and recreate a resource while keeping the configuration unchanged.

In this task, we use Terraform's replacement mechanism to recreate the existing EC2 instance **devops-ec2** and verify that a new instance with a different Instance ID is provisioned.

---

## 🎯 Objectives

✅ Identify the existing EC2 instance managed by Terraform

✅ Force Terraform to replace the EC2 instance

✅ Verify that a new EC2 instance is created

✅ Confirm the new Instance ID differs from the old one

✅ Ensure Terraform state remains synchronized

---

## 🛠️ Technologies Used

* ☁️ AWS EC2
* 🏗️ Terraform
* 📦 Infrastructure as Code (IaC)
* 🔄 Terraform Resource Replacement

---

## 🏗️ Architecture

```text
Before Replacement

Terraform State
       │
       ▼
 EC2 Instance
 (devops-ec2)
 Instance ID:
 i-1234567890


After Replacement

Terraform State
       │
       ▼
 EC2 Instance
 (devops-ec2)
 Instance ID:
 i-0987654321
```

⚡ Resource configuration remains unchanged, but Terraform destroys and recreates the instance.

---

# 🔍 Step 1: Verify Existing EC2 Instance

List Terraform-managed resources:

```bash
terraform state list
```

Expected:

```text
aws_instance.devops_ec2
```

---

## 🔍 Step 2: Capture Existing Instance ID

Retrieve the current instance ID:

```bash
terraform state show aws_instance.devops_ec2 | grep id
```

Example:

```text
id = i-0a1b2c3d4e5f67890
```

📝 Save this Instance ID for later comparison.

---

## 🔍 Alternative Verification

Using AWS CLI:

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=devops-ec2" \
--query "Reservations[*].Instances[*].[InstanceId]" \
--output table
```

Example:

```text
-------------------------
| DescribeInstances     |
+-----------------------+
| i-0a1b2c3d4e5f67890   |
+-----------------------+
```

---

# 🚀 Step 3: Force Resource Replacement

Execute:

```bash
terraform apply -replace="aws_instance.devops_ec2" -auto-approve
```

Terraform will display:

```text
-/+ resource "aws_instance" "devops_ec2" {
```

Meaning:

```text
- Destroy existing instance
+ Create new instance
```

Expected Summary:

```text
Plan: 1 to add, 0 to change, 1 to destroy.
```

---

# 🔍 Step 4: Verify New Instance Creation

After successful execution:

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=devops-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
--output table
```

Example:

```text
----------------------------------
|      DescribeInstances         |
+----------------------+---------+
| i-0987654321abcdef0  | running |
+----------------------+---------+
```

---

## ✅ Compare Instance IDs

### Before Replacement

```text
i-0a1b2c3d4e5f67890
```

### After Replacement

```text
i-0987654321abcdef0
```

✔️ Instance IDs are different

✔️ Terraform successfully recreated the EC2 instance

---

# 🔍 Step 5: Verify Terraform State

Check resource state:

```bash
terraform state show aws_instance.devops_ec2
```

Verify that:

* New Instance ID is present
* Resource status is healthy
* Terraform state updated correctly

---

# 🔍 Step 6: Verify Infrastructure

List running instances:

```bash
aws ec2 describe-instances \
--filters "Name=instance-state-name,Values=running" \
--query "Reservations[*].Instances[*].[InstanceId,Tags[0].Value]" \
--output table
```

Confirm:

```text
devops-ec2
```

exists and is running.

---

# ✅ Final Validation (Mandatory)

Run:

```bash
terraform plan
```

Expected:

```text
No changes. Your infrastructure matches the configuration.
```

🎉 This confirms Terraform state and AWS infrastructure are fully synchronized after the replacement operation.

---

## 📚 Key Concepts Learned

🔹 Terraform Resource Lifecycle

🔹 Force Resource Recreation

🔹 Terraform `-replace` Option

🔹 Infrastructure Recovery Scenarios

🔹 State Management

🔹 Resource Drift Prevention

🔹 AWS EC2 Instance Management

---

## 🏆 Outcome

Successfully used Terraform's **`-replace`** option to destroy and recreate the EC2 instance **devops-ec2** without modifying the infrastructure code. The new EC2 instance was provisioned with a different Instance ID while maintaining the same configuration, demonstrating Terraform's ability to safely recover and recreate resources when required. 🚀🔄☁️
