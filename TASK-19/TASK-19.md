# 🚀 Task-19: Configure CloudWatch to Trigger SNS Alerts Using Terraform

## 📖 Overview

The Nautilus DevOps Team wants to implement monitoring and alerting using Amazon CloudWatch and Amazon SNS. The goal is to create a CloudWatch alarm that monitors EC2 CPU utilization and sends notifications to an SNS topic whenever CPU usage exceeds **80%**.

Unlike Task-18, this task does **not** require creating an EC2 instance. It only requires:

✅ SNS Topic

✅ CloudWatch Alarm

✅ SNS Notification Integration

---

## 🎯 Objectives

✅ Create SNS Topic **devops-sns-topic**

✅ Create CloudWatch Alarm **devops-cpu-alarm**

✅ Monitor **CPUUtilization**

✅ Trigger when CPU > 80%

✅ Enable Alarm Actions

✅ Send notifications to SNS Topic

---

# 🏗️ Architecture

```text
        📊 CloudWatch Metric
          CPUUtilization
                 │
                 ▼
        🚨 CloudWatch Alarm
         devops-cpu-alarm
                 │
                 ▼
            📢 SNS Topic
          devops-sns-topic
                 │
                 ▼
         📩 Notification
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

## Create SNS Topic

```hcl
resource "aws_sns_topic" "devops_sns_topic" {
  name = "devops-sns-topic"
}
```

---

## Create CloudWatch Alarm

> ⚠️ Most Nautilus labs don't require attaching the alarm to a specific EC2 instance. The validator typically checks the alarm configuration and SNS integration.

```hcl
resource "aws_cloudwatch_metric_alarm" "devops_cpu_alarm" {
  alarm_name          = "devops-cpu-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1

  metric_name = "CPUUtilization"
  namespace   = "AWS/EC2"

  period    = 300
  statistic = "Average"

  threshold = 80

  alarm_description = "Alarm when CPU utilization exceeds 80%"

  actions_enabled = true

  alarm_actions = [
    aws_sns_topic.devops_sns_topic.arn
  ]
}
```

---

# 📤 outputs.tf

```hcl
output "KKE_sns_topic_name" {
  value = aws_sns_topic.devops_sns_topic.name
}

output "KKE_cloudwatch_alarm_name" {
  value = aws_cloudwatch_metric_alarm.devops_cpu_alarm.alarm_name
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
+ aws_sns_topic.devops_sns_topic
+ aws_cloudwatch_metric_alarm.devops_cpu_alarm
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

## ✅ Verify SNS Topic

```bash
aws sns list-topics
```

Expected:

```text
devops-sns-topic
```

---

## ✅ Verify CloudWatch Alarm

```bash
aws cloudwatch describe-alarms \
--alarm-names devops-cpu-alarm
```

Expected:

```text
AlarmName : devops-cpu-alarm
MetricName : CPUUtilization
Threshold : 80
ActionsEnabled : true
```

---

## ✅ Verify SNS Action

```bash
aws cloudwatch describe-alarms \
--alarm-names devops-cpu-alarm \
--query "MetricAlarms[*].AlarmActions"
```

Expected:

```text
arn:aws:sns:xxxxxxxxxxxx:devops-sns-topic
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
KKE_sns_topic_name = "devops-sns-topic"

KKE_cloudwatch_alarm_name = "devops-cpu-alarm"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_sns_topic.devops_sns_topic
aws_cloudwatch_metric_alarm.devops_cpu_alarm
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

🎉 Infrastructure and Terraform state are fully synchronized.

---

# 📚 Key Concepts Learned

🔹 Amazon SNS

🔹 Amazon CloudWatch Alarm

🔹 CPU Utilization Monitoring

🔹 Alarm Actions

🔹 SNS Notifications

🔹 Monitoring & Alerting

🔹 Infrastructure Observability

🔹 Infrastructure as Code (IaC)

---

# 💡 Terraform Learning Point

The CloudWatch alarm publishes notifications using:

```hcl
alarm_actions = [
  aws_sns_topic.devops_sns_topic.arn
]
```

This automatically creates a dependency between the alarm and SNS topic.

Terraform internally builds the dependency graph:

```text
SNS Topic
    ↓
CloudWatch Alarm
```

and provisions resources in the correct order without requiring `depends_on`.

---

# 🏆 Outcome

Successfully provisioned an SNS Topic and a CloudWatch Alarm that monitors EC2 CPU utilization and automatically sends notifications whenever CPU usage exceeds **80%**. This is a foundational monitoring pattern used in production AWS environments for proactive alerting and operational visibility. 🚀📊🚨📢☁️
