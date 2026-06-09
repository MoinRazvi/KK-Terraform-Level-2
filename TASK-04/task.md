### The Nautilus DevOps team wants to provision multiple EC2 instances in AWS using Terraform. Each instance should follow a consistent naming convention and be deployed using a modular and scalable setup.
#
Use Terraform to:

Create 3 EC2 instances using the count parameter.

Name each EC2 instance with the prefix datacenter-instance (e.g., datacenter-instance-1).

Instances should be t2.micro.

The key named should be datacenter-key.

Create main.tf file (do not create a separate .tf file) to provision these instances.

Use variables.tf file with the following:

KKE_INSTANCE_COUNT: number of instances.<br>
KKE_INSTANCE_TYPE: type of the instance.<br>
KKE_KEY_NAME: name of key used.<br>
KKE_INSTANCE_PREFIX: name of the instnace.<br>
Use the locals.tf file to define a local variable named AMI_ID that retrieves the latest Amazon Linux 2 AMI using a data source.<br>

Use terraform.tfvars to assign values to the variables.

Use outputs.tf file to output the following:

kke_instance_names: names of the instances created.

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.)
