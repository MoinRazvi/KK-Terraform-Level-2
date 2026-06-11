### The Nautilus DevOps team wants to automate infrastructure provisioning using CloudFormation. As part of the stack setup, they need to create a DynamoDB table.
#
Create a CloudFormation stack named devops-dynamodb-stack.

The stack must create a DynamoDB table named devops-cf-dynamodb-table.

Use the main.tf file (do not create a separate .tf file) to provision a CloudFormation stack and DynamoDB table. Make sure to add a lifecycle block in main.tf to ignore changes to the parameters attribute.

Use the variables.tf file with the following variable names:

KKE_DYNAMODB_TABLE_NAME: Dynamodb table name.<br>
The locals.tf file is already provided and includes the following:<br>

cf_template_body: A local variable that stores the CloudFormation template body.<br>
Use the outputs.tf file to output the following:<br>

KKE_stack_name: CloudFormation stack name

Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
