### The Nautilus DevOps team needs to create an AMI from an existing EC2 instance for backup and scaling purposes. The following steps are required:
#
They have an existing EC2 instance named devops-ec2.

They need to create an AMI named devops-ec2-ami from this instance.

Additionally, they need to launch a new EC2 instance named devops-ec2-new using this AMI.

Update the main.tf file (do not create a different or separate.tf file) to provision an AMI and then launch an EC2 Instance from that AMI.

Create an outputs.tf file to output the following values:

KKE_ami_id for the AMI ID you created.<br>
KKE_new_instance_id for the EC2 instance ID you created.<br>

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
