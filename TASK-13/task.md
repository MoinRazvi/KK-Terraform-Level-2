### The Nautilus DevOps team is setting up IAM-based access control for internal AWS resources. They need to create an IAM Role and an IAM Policy using Terraform and attach the policy to the role.
#
Create an IAM Role named datacenter-role.

Create an IAM Policy named datacenter-policy that allows listing EC2 instances.

Attach the policy to the role

Create the main.tf file (do not create a separate .tf file) to provision a Role, policy and attach it.

Use the variables.tf file with the following:

KKE_ROLE_NAME: name of the role.<br>
KKE_POLICY_NAME: name of the policy.<br>
Use terraform.tfvarsfile to input the role and policy names.<br>

Use outputs.tf file to output the following:

kke_iam_role_name: name of the role created.<br>
kke_iam_policy_name: name of the policy ceated.<br>

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
