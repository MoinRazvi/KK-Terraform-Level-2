# 🚀 Task-12: Integrate SNS with SQS for Messaging Using Terraform

## 📖 Overview

In this task, we implement a common AWS messaging pattern where an Amazon SNS Topic publishes messages and an Amazon SQS Queue receives them through a subscription.

This architecture enables asynchronous communication between services and is widely used in event-driven systems, microservices, and notification pipelines.

---

## 🎯 Objectives

✅ Create SNS Topic **nautilus-sns-topic**

✅ Create SQS Queue **nautilus-sqs-queue**

✅ Subscribe SQS Queue to SNS Topic

✅ Configure SQS policy to allow SNS messages

✅ Output SNS Topic ARN

✅ Output SQS Queue URL

---

# 🏗️ Architecture

```text
                📢 SNS Topic
           nautilus-sns-topic
                     │
                     ▼
             🔔 Subscription
                     │
                     ▼
               📬 SQS Queue
           nautilus-sqs-queue
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
resource "aws_sns_topic" "nautilus_topic" {
  name = "nautilus-sns-topic"
}
```

---

## Create SQS Queue

```hcl
resource "aws_sqs_queue" "nautilus_queue" {
  name = "nautilus-sqs-queue"
}
```

---

## Allow SNS to Send Messages to SQS

```hcl
resource "aws_sqs_queue_policy" "queue_policy" {
  queue_url = aws_sqs_queue.nautilus_queue.id

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [
      {
        Effect = "Allow"

        Principal = {
          Service = "sns.amazonaws.com"
        }

        Action = "sqs:SendMessage"

        Resource = aws_sqs_queue.nautilus_queue.arn

        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.nautilus_topic.arn
          }
        }
      }
    ]
  })
}
```

---

## Subscribe SQS Queue to SNS Topic

```hcl
resource "aws_sns_topic_subscription" "sns_sqs_subscription" {
  topic_arn = aws_sns_topic.nautilus_topic.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.nautilus_queue.arn
}
```

---

# 📤 outputs.tf

```hcl
output "kke_sns_topic_arn" {
  value = aws_sns_topic.nautilus_topic.arn
}

output "kke_sqs_queue_url" {
  value = aws_sqs_queue.nautilus_queue.url
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
+ aws_sns_topic.nautilus_topic
+ aws_sqs_queue.nautilus_queue
+ aws_sqs_queue_policy.queue_policy
+ aws_sns_topic_subscription.sns_sqs_subscription
```

---

### 4️⃣ Deploy Infrastructure

```bash
terraform apply -auto-approve
```

Expected:

```text
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```

---

# 🔍 Verification Steps

## ✅ Verify SNS Topic

```bash
aws sns list-topics
```

Expected:

```text
nautilus-sns-topic
```

---

## ✅ Verify SQS Queue

```bash
aws sqs list-queues
```

Expected:

```text
nautilus-sqs-queue
```

---

## ✅ Verify Subscription

```bash
aws sns list-subscriptions-by-topic \
--topic-arn $(terraform output -raw kke_sns_topic_arn)
```

Expected:

```text
Protocol: sqs
Endpoint: arn:aws:sqs:...
```

---

## ✅ Test Message Delivery

### Publish Message to SNS

```bash
aws sns publish \
--topic-arn $(terraform output -raw kke_sns_topic_arn) \
--message "Terraform SNS-SQS Test"
```

Expected:

```text
MessageId: xxxxxxxx
```

---

### Receive Message from SQS

```bash
aws sqs receive-message \
--queue-url $(terraform output -raw kke_sqs_queue_url)
```

Expected:

```text
Terraform SNS-SQS Test
```

✔️ This confirms SNS is successfully delivering messages to SQS.

---

## ✅ Verify Terraform Outputs

```bash
terraform output
```

Expected:

```text
kke_sns_topic_arn = "arn:aws:sns:..."

kke_sqs_queue_url = "https://sqs....amazonaws.com/..."
```

---

## ✅ Verify Terraform State

```bash
terraform state list
```

Expected:

```text
aws_sns_topic.nautilus_topic
aws_sqs_queue.nautilus_queue
aws_sqs_queue_policy.queue_policy
aws_sns_topic_subscription.sns_sqs_subscription
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

🔹 Amazon SNS

🔹 Amazon SQS

🔹 SNS-to-SQS Subscriptions

🔹 Queue Policies

🔹 Event-Driven Architecture

🔹 Asynchronous Messaging

🔹 Terraform Resource Dependencies

🔹 Infrastructure as Code (IaC)

---

# 🏆 Outcome

Successfully provisioned an SNS Topic and SQS Queue, configured the required queue policy, and established an SNS subscription to deliver messages automatically to the queue. This pattern is commonly used in scalable event-driven architectures and decoupled microservices environments. 🚀📢📬☁️
