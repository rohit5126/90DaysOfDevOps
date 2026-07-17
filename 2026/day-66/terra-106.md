# Provision an EKS Cluster with Terraform Modules

You built Kubernetes clusters manually in the Kubernetes week. Today you provision one the DevOps way -- fully automated,
repeatable, and destroyable with a single command. You will use Terraform registry modules to create an AWS EKS cluster with a managed node group, 
connect kubectl, and deploy a workload.

This is what infrastructure teams do every day in production.

## Task 1: Project Setup

variables.tf
```
variable "vpc-cidr" {
    default = "10.0.0.0/16"
  
}

variable "pub-cidr-1" {
    default = "10.0.1.0/24"
  
}

variable "pub-cidr-2" {
    default = "10.0.2.0/24"
  
}

variable "pvt-cidr-1" {
    default = "10.0.3.0/24"
  
}

variable "pvt-cidr-2" {
    default = "10.0.4.0/24"
  
}

variable "cluster-name" {
    default = "terraweek"
}

variable "cluster-version" {
    default = "1.29"
}

variable "tags" {
    default = {
        Name = terra-week
        Environment = Dev
    }
  
}

variable "instance_types" {
    default = "t3.micro"
  
}

```

## Task 2: Create the VPC with Registry Module

```
data "aws_availability_zones" "available" {  
}

module "vpc" {
    source = "terraform-aws-modules/vpc/aws"
    version = "5.8.1"

    name = "${var.cluster-name}-vpc"
    cidr = var.vpc-cidr

    azs = slice(data.aws_availability_zones.available.names, 0, 2)
    private_subnets = [var.pvt-cidr-1, var.pvt-cidr-2]
    public_subnets = [var.pub-cidr-1, var.pub-cidr-2]

    enable_nat_gateway = true
    single_nat_gateway = true
    enable_dns_hostnames = true

    public_subnet_tags = {
        "kubernetes.io/role/elb" = 1
    }

    private_subnet_tags = {
        "kubernetes.io/role/internal-elb" = 1
    }
  
}
```

## Task 3: Create the EKS Cluster with Registry Module

```
module "eks" {
    source = "terraform-aws-modules/eks/aws"
    version = "20.8.5"

    cluster_name = var.cluster-name
    cluster_version = var.cluster-version

    cluster_endpoint_public_access = true
    enable_cluster_creator_admin_permissions = true

    vpc_id = module.vpc.vpc_id
    subnet_ids = module.vpc.private-subnets

    eks_managed_node_groups = {
        general_nodes = {
            min_size = 1
            max_size = 4
            desired_size = 2

            instance_types = [var.instance_types]
            Terraform = "true"

        }
    }

    tags = var.tags
  
}
```

## Task 4: Apply and Connect kubectl



First craete an **IAM account** in aws and give it **admin policy** so that eks will treat it as admin.

now configur your local using **aws configure** and **add access key and password** for IAM account.

now **configure kubectl** using `kubectl configure`

`aws eks update-kubeconfig --name terraweek-eks --region <your-region>`

now run **kubectl get nodes** and **kubectl cluster-info**.

## Task 6: Destroy Everything

This is the most important step. EKS clusters cost money. Clean up completely.

First, remove the Kubernetes resources (so the AWS LoadBalancer gets deleted):

`kubectl delete -f k8s/nginx-deployment.yaml`

Wait for the LoadBalancer to be fully removed (check EC2 > Load Balancers in AWS console)

Destroy all Terraform resources:

`terraform destroy`

This will take 10-15 minutes.

Verify in the AWS console:

EKS clusters: empty
EC2 instances: no node group instances
VPC: the terraweek VPC should be gone
NAT Gateways: deleted
Elastic IPs: released
