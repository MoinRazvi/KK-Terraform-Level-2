### The Nautilus DevOps team is implementing lifecycle policies to manage object storage efficiently in AWS. They want to create an S3 bucket with a specific lifecycle rule that transitions objects to infrequent access (IA) storage after 30 days and deletes them after 365 days.
#
Create an S3 bucket named nautilus-lifecycle-2645.

Enable the S3 Versioning on the bucket.

Add a lifecycle rule named nautilus-lifecycle-rule with:

Transition to STANDARD_IA storage class after 30 days.
Expiration of objects after 365 days.
Use the main.tf file (do not create a separate .tf file) to provision the S3 bucket.

Use the variable name KKE_bucket_name in the outputs.tf file to output the created bucket name.


Notes:

The Terraform working directory is /home/bob/terraform.

Right-click under the EXPLORER section in VS Code and select Open in Integrated Terminal to launch the terminal.

Before submitting the task, ensure that terraform plan returns No changes. Your infrastructure matches the configuration.
