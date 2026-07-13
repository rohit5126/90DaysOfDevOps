# Providers, Resources and Dependencies

## Task 1: Explore the AWS Provider

```
terraform {
  required_providers {
    aws = {
        source = "hashicorp/aws"
        version = "6.54.0"
    }
  }
}

provider "aws" {
    region = "eu-north-1"

}
```

**Read the provider lock file .terraform.lock.hcl -- what does it do?**

it is maintained by terraform init. it conatines the version id and hash.

For ~> 5.0, it is functionally equivalent to saying: >= 5.0 AND < 6.0. It will allow updates to versions like 5.1, 5.2, or 5.9, but will block any update to 6.0.0 or higher

>= 5.0 Matches any version 5.0 or higher, including major overhauls (e.g., 6.2, 7.0, 12.1).

= 5.0.0 Locks to that exact version. >No updates or security patches are allowed unless the lock file is manually updated.

-----------------------------------------------------

## Task 2: Build a VPC from Scratch

```

resource "aws_vpc" "main" {
    cidr_block = "10.0.0.0/16"
    instance_tenancy = "default"
    tags = {
      Name = "TerraWeek-VPC"
    }

}

resource "aws_subnet" "public1" {
    vpc_id = aws_vpc.main.id
    cidr_block = "10.0.1.0/24"
    map_public_ip_on_launch = true
    availability_zone = "eu-north-1b"
    tags = {
      Name = "TerraWeek-Public-Subnet"
    }
  
}

resource "aws_internet_gateway" "net" {
    vpc_id = aws_vpc.main.id
    tags = {
      Name = "terra-gw"
    }
  
}

resource "aws_route_table" "route" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.net.id
  }
  tags = {
    Name = "terra-route"
  }
}

resource "aws_route_table_association" "table" {
    subnet_id = aws_subnet.public1.id
    route_table_id = aws_route_table.route.id
  
}

```

**Apply and check the AWS VPC console. Can you see all five resources connected?**

yes all is created.

## Task 3: Understand Implicit Dependencies

**How does Terraform know to create the VPC before the subnet?**

Terraform knows to create the VPC before the subnet because of resource dependency mapping

**What would happen if you tried to create the subnet before the VPC existed?**

If you bypassed Terraform’s dependency system—such as by hardcoding a fake VPC ID or trying to force an out-of-order execution—the deployment would fail during the execution phase.

**Find all implicit dependencies in your config and list them**

gateway_id = aws_internet_gateway.net.id
subnet_id = aws_subnet.public1.id
route_table_id = aws_route_table.route.id

## Task 4: Add a Security Group and EC2 Instance

```

/*
resource "aws_default_vpc" "default" {

}
*/

resource "aws_key_pair" "keypair" {
    key_name = "terra-key"
    public_key = file("terra-key.pub")
  
}

resource "aws_security_group" "security_group" {
    # depends_on = [ aws_vpc.main, aws_subnet.public1 ]
    name = "terra-sg-new"
    vpc_id = aws_vpc.main.id

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
    # depends_on = [ aws_vpc.main, aws_subnet.public1, aws_route_table.route, aws_route_table_association.table]
    vpc_security_group_ids = [aws_security_group.security_group.id]
    subnet_id = aws_subnet.public1.id
    key_name = aws_key_pair.keypair.key_name
    instance_type = "t3.micro"
    ami = "ami-0aba19e56f3eaec05"
    associate_public_ip_address = true
    root_block_device {
      volume_size = "8"
      volume_type = "gp3"
      
    }
    tags = {
        Name = "server-terra"
    }

}
```

## Task 5: Explicit Dependencies with depends_on

```
resource "aws_s3_bucket" "my_s3_bucket" {
    depends_on = [ aws_instance.my-instance ]
    bucket = "rohit-bucket-5126"
}
```

you can visualize the entire dependency tree:

```
terraform graph | dot -Tpng > graph.png
```


if dot not present install it with 
```
 sudo apt install graphviz
```

## Task 6: Lifecycle Rules and Destroy

```
lifecycle {
  create_before_destroy = true
}
```

**the three lifecycle arguments (create_before_destroy, prevent_destroy, ignore_changes)**

prevent_destroy : What it does: When set to true, this argument acts as a safeguard. If you run a terraform destroy command or remove the resource from your configuration, 
Terraform will throw an error and refuse to delete the object

