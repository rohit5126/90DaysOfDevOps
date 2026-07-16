# Terraform Modules: Build Reusable Infrastructure

**You have been writing everything in one big main.tf file. That works for learning, but in real teams you manage dozens of environments with hundreds of resources. Copy-pasting configs across projects is a recipe for disaster.
Today you learn Terraform modules -- the way to package, reuse, and share infrastructure code. Think of modules as functions in programming. Write once, call many times.**


## Task 1: Understand Module Structure

**Document:** What is the difference between a "root module" and a "child module"?

**Root Module:** This is where you execute commands like terraform plan. It sets up the big picture, such as where your files are stored and what cloud platforms you are using.

**Child Module:** This is an external block of code that does one specific job (like building a network or a server). It cannot run on its own; it must be triggered by the root module

## Task 2: Build a Custom EC2 Module

its multiple file so not updating here will be adding it to my github.


Call the security group module:
module "web_sg" {
  source        = "./modules/security-group"
  vpc_id        = aws_vpc.main.id
  sg_name       = "terraweek-web-sg"
  ingress_ports = [22, 80, 443]
  tags          = local.common_tags
}
Call the EC2 module -- deploy two instances with different names using the same module:
module "web_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-web"
  tags               = local.common_tags
}

module "api_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-api"
  tags               = local.common_tags
}

-----------------------------------------------


## Task 5: Use a Public Registry Module

Replace your hand-written VPC resources with:

```
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway = false
  enable_dns_hostnames = true

  tags = local.common_tags
}
```

you can use vpc-id and subnet-id from this module in your main file.

Document: Where does Terraform download registry modules to? Check '.terraform/modules/'.

------------------------------------

## Task 6: Module Versioning and Best Practices



Pin your registry module version explicitly:

version = "5.1.0" -- exact version
version = "~> 5.0" -- any 5.x version
version = ">= 5.0, < 6.0" -- range
Run terraform init -upgrade to check for newer versions

Check the state to see how modules appear:

terraform state list
Notice the module.vpc., module.web_server., module.web_sg. prefixes.

**Document: Write down five module best practices:**

Always pin versions for registry modules
Keep modules focused -- one concern per module
Use variables for everything, hardcode nothing
Always define outputs so callers can reference resources
Add a README.md to every custom module
