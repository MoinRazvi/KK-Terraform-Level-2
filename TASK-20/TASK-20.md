# 🚀 Task-20: Create DynamoDB Table Using CloudFormation with Terraform

## 📖 Overview

In this task, Terraform is used to provision an AWS CloudFormation Stack, and CloudFormation is responsible for creating the DynamoDB table.

A key requirement is to use a **lifecycle block** to ignore changes to the `parameters` attribute. This prevents Terraform from attempting unnecessary updates if parameter values change outside of Terraform.

---

## 🎯 Objectives

✅ Create CloudFormation Stack **devops-dynamodb-stack**

✅ Create DynamoDB Table **devops-cf-dynamodb-table**

✅ Use existing local variable **cf_template_body**

✅ Ignore changes to `parameters`

✅ Output Stack Name

---

# 🏗️ Architecture

```text
            Terraform
                 │
                 ▼
     CloudFormation Stack
      devops-dynamodb-stack
                 │
                 ▼
          DynamoDB Table
    devops-cf-dynamodb-table
```

---

# 📂 Project Structure

```bash
terraform/
├── main.tf
├── variables.tf
├── locals.tf
└── outputs.tf
```

---

# 📝 variables.tf

```hcl
variable "KKE_DYNAMODB_TABLE_NAME" {
  description = "DynamoDB Table Name"
  type        = string
  default     = "devops-cf-dynamodb-table"
}
```

> If the lab specifically requires a tfvars file, move the value there and remove the default.

---

# 📄 locals.tf

The lab mentions that `locals.tf` is already provided.

It should look similar to:

```hcl
locals {
  cf_template_body = <<TEMPLATE
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  DynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Ref TableName
      BillingMode: PAY_PER_REQUEST

      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S

      KeySchema:
        - AttributeName: id
          KeyType: HASH

Parameters:
  TableName:
    Type: String
TEMPLATE
}
```

⚠️ Do not modify if already provided by the lab.

---

# ⚙️ main.tf

```hcl
resource "aws_cloudformation_stack" "devops_stack" {
  name = "devops-dynamodb-stack"

  template_body = local.cf_template_body

  parameters = {
    TableName = var.KKE_DYNAMODB_TABLE_NAME
  }

  lifecycle {
    ignore_changes = [
      parameters
    ]
  }
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_stack_name" {
  value = aws_cloudformation_stack.devops_stack.name
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
+ aws_cloudformation_stack.devops_stack
```

---

### 4️⃣ Apply Configuration

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify CloudFormation Stack

```bash
aws cloudformation describe-stacks \
--stack-name devops-dynamodb-stack
```

Expected:

```text
StackStatus : CREATE_COMPLETE
```

---

## ✅ Verify DynamoDB Table

```bash
aws dynamodb describe-table \
--table-name devops-cf-dynamodb-table
```

Expected:

```text
TableStatus : ACTIVE
```

---

## ✅ Verify Stack Resources

```bash
aws cloudformation list-stack-resources \
--stack-name devops-dynamodb-stack
```

Expected:

```text
AWS::DynamoDB::Table
```

---

## ✅ Verify Terraform Output

```bash
terraform output
```

Expected:

```text
KKE_stack_name = "devops-dynamodb-stack"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_cloudformation_stack.devops_stack
```

---

## ✅ Verify Lifecycle Ignore Changes

Inspect state:

```bash
terraform state show aws_cloudformation_stack.devops_stack
```

Confirm:

```text
parameters
```

are managed and ignored for future drift detection.

---

## ✅ Final Validation

```bash
terraform plan
```

Expected:

```text
No changes. Your infrastructure matches the configuration.
```

🎉 This confirms that Terraform state, CloudFormation stack, and DynamoDB table are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 AWS CloudFormation Stack

🔹 Terraform + CloudFormation Integration

🔹 DynamoDB Provisioning

🔹 CloudFormation Parameters

🔹 Terraform Lifecycle Meta-Arguments

🔹 `ignore_changes`

🔹 Infrastructure as Code (IaC)

🔹 Hybrid Provisioning Approaches

---

# 💡 Terraform Learning Point

This task demonstrates a common enterprise pattern:

```text
Terraform
    ↓
CloudFormation
    ↓
AWS Resources
```

Instead of creating resources directly with Terraform, organizations sometimes use Terraform to orchestrate CloudFormation stacks, especially when teams already have existing CloudFormation templates.

The `lifecycle` block:

```hcl
lifecycle {
  ignore_changes = [parameters]
}
```

prevents Terraform from detecting and applying unnecessary updates when parameter values drift, which is frequently used in production environments managing externally updated stacks.

---

# 🏆 Outcome

Successfully provisioned a CloudFormation Stack using Terraform and created a DynamoDB table through the stack. The configuration includes lifecycle management to ignore parameter changes, demonstrating a practical enterprise pattern for integrating Terraform with existing CloudFormation-based infrastructure. 🚀☁️🗄️📜
