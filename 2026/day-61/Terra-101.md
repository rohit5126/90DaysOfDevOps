You have been deploying containers, writing CI/CD pipelines, and orchestrating workloads on Kubernetes. But who creates the servers, networks, and clusters underneath? Today you start your Infrastructure as Code journey with Terraform -- the tool that lets you define, provision, 
and manage cloud infrastructure by writing code.

## Task 1: Understand Infrastructure as Code

**What is Infrastructure as Code (IaC)? Why does it matter in DevOps?**

Infrastructure as Code (IaC) is a DevOps practice where you manage and provision IT infrastructure (servers, networks, databases) 
using machine-readable definition files rather than manual configuration

IaC is a foundational pillar of modern DevOps because it bridges the gap between development and operations teams and solves critical scalability challenges.

**What problems does IaC solve compared to manually creating resources in the AWS console?**

* **Human Error:** Eliminates manual typos, missed configurations, and forgotten security settings.
* **Configuration Drift:** Prevents development, staging, and production environments from deviating.
* **Lack of Auditability:** Tracks all infrastructure changes via Git version control history.
* **Deployment Bottlenecks:** Replaces slow, manual clicking with fast, automated deployments.
* **Slow Disaster Recovery:** Recreates entire cloud architectures instantly during outages.
* **Wasted Costs:** Automates the teardown of temporary environments to save money.

**How is Terraform different from AWS CloudFormation, Ansible, and Pulumi?**

* **Terraform:** Multi-cloud infrastructure provisioning tool using its own specific configuration language (HCL).
* **AWS CloudFormation:** AWS-native infrastructure provisioning tool that uses standard JSON or YAML.
* **Ansible:** Configuration management tool designed to set up software inside servers rather than building the servers themselves.
* **Pulumi:** Multi-cloud infrastructure provisioning tool that lets you use standard programming languages like Python or TypeScript.

**What does it mean that Terraform is "declarative" and "cloud-agnostic"?**

Declarative means you define what the final infrastructure should look like, not how to build it. You specify the target state (e.g., "I want 3 servers"), and Terraform automatically figures out the steps, creation order, and dependencies to make it happen.

Cloud-agnostic means the tool is not locked into a single cloud provider. You can use the exact same Terraform workflow, commands, and syntax to manage resources across AWS, Azure, Google Cloud, or SaaS platforms simultaneously.

## Task 2: Install Terraform and Configure AWS

```
# Linux (amd64)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform


Install and configure the AWS CLI:

aws login

rohit@rohit:~/terraform_practice$ aws login
Attempting to open your default browser. If the browser does not open, open the following URL.

//
aws configure
# Enter your Access Key ID, Secret Access Key, default region (e.g., ap-south-1), output format (json)

aws sts get-caller-identity
```

## Task 3: Your First Terraform Config -- Create an S3 Bucket

```
resource "aws_s3_bucket" "my_s3_bucket" {
    bucket = "rohit-bucket-5126"
}
```
terrafomr init
terraform plan
terraform apply 

What did terraform init download? What does the .terraform/ directory contain?

it downloads the providers plugins and Remote Modules.
it is stored in .terraform dir


## Task 4: Add an EC2 Instance

first create a ssh-keygen in same dir.

```
resource "aws_default_vpc" "default" {

}

resource "aws_key_pair" "keypair" {
    key_name = "terra-key"
    public_key = file("terra-key.pub")
  
}

resource "aws_security_group" "security_group" {
    name = "terra-sg-new"
    vpc_id = aws_default_vpc.default.id

    ingress {
        cidr_blocks = [ "0.0.0.0/0" ]
        protocol = "tcp"
        from_port = 22
        to_port = 22

    }  

    ingress {
        cidr_blocks = [ "0.0.0.0/0" ]
        protocol = "tcp"
        from_port = 80
        to_port = 80

    } 

    egress {
        cidr_blocks = [ "0.0.0.0/0" ]
        protocol = "-1"
        from_port = 0
        to_port = 0
        
    }

}

resource "aws_instance" "my-instance" {
    key_name = aws_key_pair.keypair.key_name
    security_groups = [aws_security_group.security_group.name]
    instance_type = "t3.micro"
    ami = "ami-0aba19e56f3eaec05"
    root_block_device {
      volume_size = "8"
      volume_type = "gp3"
      tags = {
        name = "server-terra"
      }
    }

}
```

How does Terraform know the S3 bucket already exists and only the EC2 instance needs to be created?

because of terraform state file.When you run terraform apply, Terraform does not just blindly send creation requests to AWS. 

---------------------------

## Task 5: Understand the State File

```
terraform show                          # Human-readable view of current state

terraform state list                    # List all resources Terraform manages

$ terraform state list 
aws_default_vpc.default
aws_instance.my-instance
aws_key_pair.keypair
aws_s3_bucket.my_s3_bucket
aws_security_group.security_group

terraform state show aws_s3_bucket.<name>   # Detailed view of a specific resource

terraform state show aws_instance.<name>

```

**Answer these questions in your notes:**

**What information does the state file store about each resource?**

The state file contains everything Terraform needs to know about your infrastructure. At its core, it maps the abstract names in your
Technically, it is a structured JSON file containing metadata, schemas, and resource state.


**Why should you never manually edit the state file?**

You should never manually edit the Terraform state file because it acts as the exact database mapping your code to your live cloud infrastructure.
Even a minor manual adjustment can break your automation

**Why should the state file not be committed to Git?**

You should never commit your terraform.tfstate file to Git for two critical reasons: security vulnerabilities and team collaboration conflicts.

------------------------------------------------

## Task 6: Modify, Plan, and Destroy

```
terraform destroy
```

