# Terraform State Management and Remote Backends

## Task 1: Inspect Your Current State


terraform show                                    # Full state in human-readable format
terraform state list                              # All resources tracked by Terraform
terraform state show aws_instance.<name>          # Every attribute of the instance
terraform state show aws_vpc.<name>               # Every attribute of the VPC

Answer:

**How many resources does Terraform track?**

9 resources and 3 data 

**What attributes does the state store for an EC2 instance? (hint: way more than what you defined)**
every attribute.

Open terraform.tfstate in an editor -- find the serial number. What does it represent?

192

In a terraform.tfstate file, the serial number is a monotonically increasing integer that tracks the version iteration of that specific state 
file.Every single time Terraform modifies your infrastructure or successfully
runs an operation that updates the state (such as terraform apply or terraform refresh), this integer increments by 1

----------------------------------------

## Task 2: Set Up S3 Remote Backend

