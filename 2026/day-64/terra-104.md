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

```
resource "aws_kms_key" "state-key" {
    deletion_window_in_days = 30
    enable_key_rotation = true

    tags = local.common_tags
  
}

resource "aws_kms_alias" "state-key" {
    name = "alias/terraform-state-key"
    target_key_id = aws_kms_key.state-key.key_id

}

resource "aws_s3_bucket" "terraform-state" {
    bucket = "rohit-state-bucket-5126"
    force_destroy = false 
    
    lifecycle {
      prevent_destroy = true
    }

    tags = local.common_tags
}

resource "aws_s3_bucket_versioning" "terraform-state" {
    bucket = aws_s3_bucket.terraform-state.id
    versioning_configuration {
      status = "Enabled"
    }
  
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform-state" {
    bucket = aws_s3_bucket.terraform-state.id

    rule {
      apply_server_side_encryption_by_default {
        kms_master_key_id = aws_kms_key.state-key.arn
        sse_algorithm = "aws:kms"
      }
    }
  
}
```

**Add the backend block to your Terraform config:**

```

terraform {
  backend "s3" {
    bucket       = "rohit-state-bucket-5126"
    key          = "terraform.tfstate"
    region       = "eu-north-1"
    encrypt      = true
    
    # Enable S3-native state locking
    use_lockfile = true 
  }
}
```
**terraform init --reconfigure**

**Your local terraform.tfstate should now be empty**

-------------------------------------

## Task 3: Test State Locking

**Terminal 2 should show a lock error with a Lock ID**

```
acquiring state lock. This may take a few moments...
╷
│ Error: Error acquiring the state lock
│ 
│ Error message: operation error S3: PutObject, https response error StatusCode: 412, RequestID: FFMV9J35KDR8A49X, HostID:
│ yrYq85hFxpzTfQZDwZq2WQiynzFRKI+AHVjgRigSlNpQLFEsH81MXn/BsfNO9FrCYaKO49eBEtz58VoL75MBT/UCEl2Yug8L, api error PreconditionFailed: At
│ least one of the pre-conditions you specified did not hold
│ Lock Info:
│   ID:        acef6cbd-6589-17d7-003e-b0c93e03e522
│   Path:      rohit-state-bucket-5126/terraform.tfstate
│   Operation: OperationTypeApply
│   Who:       rohit@rohit
│   Version:   1.15.8
│   Created:   2026-07-14 10:55:46.920442403 +0000 UTC
│   Info:      
│ 
│ 
│ Terraform acquires a state lock to protect the state from being written
│ by multiple users at the same time. Please resolve the issue above and try
│ again. For most commands, you can disable locking with the "-lock=false"
│ flag, but this is not recommended.
╵
```
After the test, if you get stuck with a stale lock:

terraform force-unlock <LOCK_ID>

---------------------------------------

## Task 4: Import an Existing Resource

```
resource "aws_instance" "imported" {
    
}
```
`terraform import aws_instance.imported i-04dcc0263d5600042`

it will import your instance.

## Task 5: State Surgery -- mv and rm

**Sometimes you need to rename a resource or remove it from state without destroying it in AWS.**

**Rename a resource in state:**

terraform state list                              # Note the current resource names
terraform state mv aws_s3_bucket.imported aws_s3_bucket.logs_bucket

Update your .tf file to match the new name. Run terraform plan --> it should show no changes.

**Remove a resource from state (without destroying it):**

terraform state rm aws_s3_bucket.logs_bucket

Run terraform plan --> Terraform no longer knows about the bucket, but it still exists in AWS.

**Document: When would you use state mv in a real project? When would you use state rm?**

to rename an aws resource if required, state rm evry important, if you want to import the instance to get details but do not want to delete them with terraform destroy.

## Task 6: Simulate and Fix State Drift

**State drift happens when someone changes infrastructure outside of Terraform -- through the AWS console, CLI, or another tool.**

Apply your full config so everything is in sync
Go to the AWS console and manually:

Change the Name tag of your EC2 instance to "ManuallyChanged"
Change the instance type if it's stopped (or add a new tag)

Run:
terraform plan
You should see a diff -- Terraform detects that reality no longer matches the desired state.

You have two choices:

Option A: Run terraform apply to force reality back to match your config (reconcile)
Option B: Update your .tf files to match the manual change (accept the drift)
Choose Option A -- apply and verify the tags are restored.

Run terraform plan again -- it should show "No changes." Drift resolved.

**Document: How do teams prevent state drift in production? (hint: restrict console access, use CI/CD for all changes)**
