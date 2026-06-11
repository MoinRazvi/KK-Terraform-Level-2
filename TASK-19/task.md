### The Nautilus DevOps team is expanding their AWS infrastructure and requires the setup of a CloudWatch alarm and SNS integration for monitoring EC2 instances. The team needs to configure an SNS topic for CloudWatch to publish notifications when an EC2 instance’s CPU utilization exceeds 80%. The alarm should trigger whenever the CPU utilization is greater than 80% and notify the SNS topic to alert the team.
#
Create an SNS topic named devops-sns-topic.

Create a CloudWatch alarm named devops-cpu-alarm to monitor EC2 CPU utilization with the following conditions:

Metric: CPUUtilization<br>
Threshold: 80%<br>
Actions enabled<br>
Alarm actions should be triggered to the SNS topic.<br>
Ensure that the SNS topic receives notifications from the CloudWatch alarm when it is triggered.<br>

Update the main.tf file (do not create a different .tf file) to create SNS Topic and Cloudwatch Alarm.

Create an outputs.tf file to output the following values:<br>

KKE_sns_topic_name for the SNS topic name.<br>
KKE_cloudwatch_alarm_name for the CloudWatch alarm name.<br>


Notes:<br>
The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
