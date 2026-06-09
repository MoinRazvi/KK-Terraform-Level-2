# 🚀 Task-07: Stream Kinesis Data to CloudWatch Using Terraform

## 📖 Overview

In this task, we provision an Amazon Kinesis Data Stream and configure CloudWatch monitoring to improve observability of streaming workloads. The goal is to detect write throughput throttling events and generate alerts whenever the stream exceeds its provisioned write capacity.

This setup helps operations teams proactively identify ingestion bottlenecks before they impact downstream applications.

---

## 🎯 Objectives

✅ Create a Kinesis Data Stream named **xfusion-kinesis-stream**

✅ Configure the stream with **1 shard**

✅ Enable shard-level monitoring metrics

✅ Create a CloudWatch Alarm named **xfusion-kinesis-alarm**

✅ Monitor **WriteProvisionedThroughputExceeded**

✅ Trigger the alarm when the metric exceeds **1**

✅ Output the stream name and alarm name

---

## 🏗️ Architecture

```text
                Producers
                     │
                     ▼
        🌊 Kinesis Data Stream
        (xfusion-kinesis-stream)
                     │
                     ▼
         📊 CloudWatch Metrics
                     │
                     ▼
      🚨 CloudWatch Alarm
      (xfusion-kinesis-alarm)
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

## Create Kinesis Data Stream

```hcl
resource "aws_kinesis_stream" "xfusion_kinesis_stream" {
  name             = "xfusion-kinesis-stream"
  shard_count      = 1
  retention_period = 24

  shard_level_metrics = [
    "IncomingBytes",
    "IncomingRecords",
    "WriteProvisionedThroughputExceeded"
  ]
}
```

---

## Create CloudWatch Alarm

```hcl
resource "aws_cloudwatch_metric_alarm" "xfusion_kinesis_alarm" {
  alarm_name          = "xfusion-kinesis-alarm"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "WriteProvisionedThroughputExceeded"
  namespace           = "AWS/Kinesis"
  period              = 60
  statistic           = "Sum"
  threshold           = 1

  dimensions = {
    StreamName = aws_kinesis_stream.xfusion_kinesis_stream.name
  }

  alarm_description = "Triggers when write throughput exceeds provisioned limits."

  treat_missing_data = "notBreaching"
}
```

---

# 📤 outputs.tf

```hcl
output "kke_kinesis_stream_name" {
  value = aws_kinesis_stream.xfusion_kinesis_stream.name
}

output "kke_kinesis_alarm_name" {
  value = aws_cloudwatch_metric_alarm.xfusion_kinesis_alarm.alarm_name
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
+ aws_kinesis_stream.xfusion_kinesis_stream
+ aws_cloudwatch_metric_alarm.xfusion_kinesis_alarm
```

---

### 4️⃣ Deploy Infrastructure

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify Kinesis Stream

```bash
aws kinesis describe-stream-summary \
--stream-name xfusion-kinesis-stream
```

Expected:

```text
StreamName : xfusion-kinesis-stream
OpenShardCount : 1
StreamStatus : ACTIVE
```

---

## ✅ Verify Shard Count

```bash
aws kinesis describe-stream-summary \
--stream-name xfusion-kinesis-stream \
--query "StreamDescriptionSummary.OpenShardCount"
```

Expected:

```text
1
```

---

## ✅ Verify Enhanced Monitoring

```bash
aws kinesis describe-stream \
--stream-name xfusion-kinesis-stream
```

Verify:

```text
WriteProvisionedThroughputExceeded
IncomingBytes
IncomingRecords
```

are listed under:

```text
EnhancedMonitoring
```

---

## ✅ Verify CloudWatch Alarm

```bash
aws cloudwatch describe-alarms \
--alarm-names xfusion-kinesis-alarm
```

Expected:

```text
AlarmName : xfusion-kinesis-alarm
MetricName : WriteProvisionedThroughputExceeded
Threshold : 1
```

---

## ✅ Verify Alarm Configuration

```bash
aws cloudwatch describe-alarms \
--alarm-names xfusion-kinesis-alarm \
--query "MetricAlarms[*].[AlarmName,MetricName,Threshold]"
```

Expected:

```text
-------------------------------------------------------
| xfusion-kinesis-alarm | WriteProvisionedThroughputExceeded | 1 |
-------------------------------------------------------
```

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
kke_kinesis_stream_name = "xfusion-kinesis-stream"

kke_kinesis_alarm_name = "xfusion-kinesis-alarm"
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_kinesis_stream.xfusion_kinesis_stream
aws_cloudwatch_metric_alarm.xfusion_kinesis_alarm
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

🔹 Amazon Kinesis Data Streams

🔹 Enhanced Shard-Level Monitoring

🔹 CloudWatch Metrics

🔹 CloudWatch Alarms

🔹 Throughput Monitoring

🔹 Infrastructure as Code (IaC)

🔹 Terraform Resource Dependencies

🔹 AWS Observability Best Practices

---

# 🏆 Outcome

Successfully provisioned a Kinesis Data Stream and integrated CloudWatch monitoring to detect write throughput throttling events. This observability setup enables proactive monitoring of streaming workloads and helps ensure reliable data ingestion within AWS environments. 🚀🌊📊🚨
