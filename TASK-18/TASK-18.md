# 🚀 Task-18: Create and Configure CloudWatch Alarm Using Terraform

## 📖 Overview

The Nautilus DevOps Team needs to provision an EC2 instance and monitor its CPU utilization using Amazon CloudWatch. If CPU usage exceeds **90%** for one consecutive **5-minute period**, a notification should be sent to the SNS topic **datacenter-sns-topic**.

Since the SNS topic already exists in the Terraform configuration, we directly reference the Terraform resource instead of creating a data source.

---

## 🎯 Objectives

✅ Create EC2 Instance **datacenter-ec2**

✅ Use Ubuntu AMI **ami-0c02fb55956c7d316**

✅ Create CloudWatch Alarm **datacenter-alarm**

✅ Monitor **CPUUtilization**

✅ Trigger when CPU ≥ 90%

✅ Evaluation Period = 1

✅ Period = 300 seconds (5 minutes)

✅ Send notification to SNS Topic **datacenter-sns-topic**

---

# 🏗️ Architecture

```text
             🖥️ EC2 Instance
             datacenter-ec2
                     │
                     ▼
          📊 CloudWatch Metrics
             CPUUtilization
                     │
                     ▼
           🚨 CloudWatch Alarm
            datacenter-alarm
                     │
                     ▼
               📢 SNS Topic
          datacenter-sns-topic
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

> Existing SNS Topic already present in the Terraform configuration.

```hcl
resource "aws_sns_topic" "sns_topic" {
  name = "datacenter-sns-topic"
}

resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-ec2"
  }
}

resource "aws_cloudwatch_metric_alarm" "datacenter_alarm" {
  alarm_name          = "datacenter-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1

  metric_name = "CPUUtilization"
  namespace   = "AWS/EC2"

  statistic = "Average"
  period    = 300
  threshold = 90

  alarm_description = "Alarm when CPU utilization exceeds 90%"

  dimensions = {
    InstanceId = aws_instance.datacenter_ec2.id
  }

  alarm_actions = [
    aws_sns_topic.sns_topic.arn
  ]
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_instance_name" {
  value = aws_instance.datacenter_ec2.tags["Name"]
}

output "KKE_alarm_name" {
  value = aws_cloudwatch_metric_alarm.datacenter_alarm.alarm_name
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
+ aws_instance.datacenter_ec2
+ aws_cloudwatch_metric_alarm.datacenter_alarm
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

# 🔍 Verification Steps

## ✅ Verify EC2 Instance

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=datacenter-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,State.Name]" \
--output table
```

Expected:

```text
running
```

---

## ✅ Verify SNS Topic

```bash
aws sns list-topics
```

Expected:

```text
datacenter-sns-topic
```

---

## ✅ Verify CloudWatch Alarm

```bash
aws cloudwatch describe-alarms \
--alarm-names datacenter-alarm
```

Expected:

```text
AlarmName : datacenter-alarm
MetricName : CPUUtilization
Threshold : 90
```

---

## ✅ Verify Alarm Action

```bash
aws cloudwatch describe-alarms \
--alarm-names datacenter-alarm \
--query "MetricAlarms[*].AlarmActions"
```

Expected:

```text
arn:aws:sns:xxxxxxxxxxxx:datacenter-sns-topic
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
KKE_instance_name = "datacenter-ec2"

KKE_alarm_name = "datacenter-alarm"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_sns_topic.sns_topic
aws_instance.datacenter_ec2
aws_cloudwatch_metric_alarm.datacenter_alarm
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

🔹 Amazon EC2

🔹 Amazon CloudWatch Alarms

🔹 SNS Notifications

🔹 CPU Monitoring

🔹 Terraform Resource References

🔹 CloudWatch Metrics

🔹 Infrastructure Observability

🔹 Infrastructure as Code (IaC)

---

# 💡 Terraform Learning Point

In this task, the SNS topic was already managed by Terraform:

```hcl
resource "aws_sns_topic" "sns_topic"
```

Therefore, the CloudWatch Alarm directly references:

```hcl
aws_sns_topic.sns_topic.arn
```

instead of using a Data Source.

### Senior DevOps Rule

✅ Use **resource references** when Terraform already manages the infrastructure.

```hcl
aws_sns_topic.sns_topic.arn
```

✅ Use **data sources** when the infrastructure exists outside the current Terraform configuration.

```hcl
data.aws_sns_topic.sns_topic.arn
```

Understanding this distinction is critical for building clean, maintainable, and scalable Terraform codebases.

---

# 🏆 Outcome

Successfully provisioned an EC2 instance and configured a CloudWatch Alarm that monitors CPU utilization. The alarm sends notifications to an SNS topic whenever CPU usage exceeds **90%** for a continuous **5-minute period**, enabling proactive monitoring and faster incident response in production environments. 🚀🖥️📊🚨📢☁️
