## 🚀 Task-01: Create VPC and Subnet Using Terraform

### 📖 Overview

In this task, we create an AWS **VPC** and a **Subnet** using Terraform while explicitly defining the resource dependency using the `depends_on` argument. This ensures Terraform provisions the VPC before creating the Subnet, maintaining the correct infrastructure deployment order.

---

## 🎯 Objectives

✅ Create a VPC named **datacenter-vpc**

✅ Create a Subnet named **datacenter-subnet**

✅ Use `depends_on` to explicitly define resource dependency

✅ Manage resource names using Terraform variables

✅ Display resource names using outputs

---

## 🛠️ Technologies Used

* ☁️ AWS VPC
* 🌐 AWS Subnet
* 🏗️ Terraform
* 📦 Infrastructure as Code (IaC)

---

## 📂 Project Structure

```bash
terraform/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```

---

## ⚙️ Implementation

### 📝 Variables Configuration

Define reusable variables for the VPC and Subnet names.

```hcl
variable "KKE_VPC_NAME" {
  type = string
}

variable "KKE_SUBNET_NAME" {
  type = string
}
```

---

### 🌐 VPC and Subnet Creation

Create the VPC and Subnet while explicitly defining the dependency.

```hcl
resource "aws_vpc" "datacenter_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = var.KKE_VPC_NAME
  }
}

resource "aws_subnet" "datacenter_subnet" {
  vpc_id     = aws_vpc.datacenter_vpc.id
  cidr_block = "10.0.1.0/24"

  depends_on = [
    aws_vpc.datacenter_vpc
  ]

  tags = {
    Name = var.KKE_SUBNET_NAME
  }
}
```

---

### 📋 Variable Values

```hcl
KKE_VPC_NAME    = "datacenter-vpc"
KKE_SUBNET_NAME = "datacenter-subnet"
```

---

### 📤 Outputs

```hcl
output "kke_vpc_name" {
  value = aws_vpc.datacenter_vpc.tags["Name"]
}

output "kke_subnet_name" {
  value = aws_subnet.datacenter_subnet.tags["Name"]
}
```

---

## 🚀 Deployment Steps

### 1️⃣ Initialize Terraform

```bash
terraform init
```

### 2️⃣ Validate Configuration

```bash
terraform validate
```

### 3️⃣ Review Execution Plan

```bash
terraform plan
```

### 4️⃣ Deploy Resources

```bash
terraform apply -auto-approve
```

### 5️⃣ Verify Outputs

```bash
terraform output
```

---

## 🔍 Resource Verification

### ✅ Verify VPC Creation

List all VPCs:

```bash
aws ec2 describe-vpcs \
  --query "Vpcs[*].[VpcId,CidrBlock]" \
  --output table
```

Check VPC Name Tag:

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=datacenter-vpc" \
  --query "Vpcs[*].[VpcId,CidrBlock]" \
  --output table
```

Expected Output:

```text
------------------------------------
|          DescribeVpcs           |
+----------------+---------------+
|   vpc-xxxxxx   | 10.0.0.0/16   |
+----------------+---------------+
```

---

### ✅ Verify Subnet Creation

List Subnets:

```bash
aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=datacenter-subnet" \
  --query "Subnets[*].[SubnetId,CidrBlock,VpcId]" \
  --output table
```

Expected Output:

```text
--------------------------------------------------
|              DescribeSubnets                   |
+------------+--------------+-------------------+
| subnet-xxx | 10.0.1.0/24  | vpc-xxxxxxxx      |
+------------+--------------+-------------------+
```

---

### ✅ Verify Terraform State

Check managed resources:

```bash
terraform state list
```

Expected:

```text
aws_vpc.datacenter_vpc
aws_subnet.datacenter_subnet
```

---

### ✅ Verify Infrastructure Consistency

Run:

```bash
terraform plan
```

Expected:

```text
No changes. Your infrastructure matches the configuration.
```

🎉 This confirms that Terraform state and AWS infrastructure are fully synchronized.

---

## 📚 Key Concepts Learned

🔹 Terraform Resource Dependencies (`depends_on`)

🔹 AWS VPC Provisioning

🔹 AWS Subnet Creation

🔹 Terraform Variables and Outputs

🔹 Infrastructure State Management

🔹 Infrastructure as Code (IaC) Best Practices

---

## 🏆 Outcome

Successfully provisioned a custom AWS VPC and Subnet using Terraform with an explicit dependency configuration, ensuring predictable and reliable infrastructure deployment.

✨ **Infrastructure automated, dependencies managed, and resources provisioned successfully using Terraform!** 🚀🌍
