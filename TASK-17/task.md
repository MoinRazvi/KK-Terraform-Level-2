### To enable secure retrieval of secrets, the Nautilus DevOps team needs to configure access to a secret in AWS Secrets Manager using IAM roles and policies. The objective is to allow EC2 instances to retrieve secrets securely. Please complete the following tasks:
#
Create a secret in AWS Secrets Manager named devops-app-secret with the following secret string:

{"db_user":"admin","db_pass":"supersecret"}

Create an IAM role named devops-app-role with EC2 as the trusted entity.

Attach an inline IAM policy named devops-app-policy that grants permission to retrieve the secret from AWS Secrets Manager.

Use the main.tf file (do not create a separate .tf file) to provision the IAM Role and IAM Policy.

Create the variables.tf file, ensure the following variables are defined in variables.tf file:

KKE_SECRET_NAME for the secret name.<br>
KKE_SECRET_VALUE for the secret value.<br>
KKE_ROLE_NAME for the IAM role name.<br>
KKE_POLICY_NAME for the IAM policy name.<br>
Create the outputs.tf file, and use the following:

KKE_secret_name: The secret name<br>
KKE_role_name: The IAM role name<br>
KKE_policy_name: The IAM policy name<br>

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
