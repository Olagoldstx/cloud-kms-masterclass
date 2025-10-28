![securethecloud](https://github.com/user-attachments/assets/3bc3c1e6-3aad-4723-89aa-29be88a8eb9f)



# AWS KMS — Beginner Track

**Day 1** covers:
- Creating a symmetric CMK
- S3 SSE-KMS with a customer managed key
- EBS default encryption using your CMK
- Verifying with CloudTrail

Open: `day-by-day/day1-aws-beginner.md`  
Then try: `aws/beginner/iac/` for **CloudFormation** and **Terraform** versions.
3) IaC (optional but included for you)
3a) CloudFormation — aws/beginner/iac/cloudformation/day1-sse-kms.yaml
bash
Copy code
nano aws/beginner/iac/cloudformation/day1-sse-kms.yaml
Paste:

yaml
Copy code
AWSTemplateFormatVersion: '2010-09-09'
Description: Day1 - KMS CMK + S3 SSE-KMS + EBS default encryption

Parameters:
  BucketName:
    Type: String
    Description: S3 bucket name
  AliasName:
    Type: String
    Default: alias/masterclass-cmk

Resources:
  CMK:
    Type: AWS::KMS::Key
    Properties:
      Description: Masterclass Day1 CMK
      KeyUsage: ENCRYPT_DECRYPT
      Origin: AWS_KMS
      Enabled: true
      Tags:
        - Key: project
          Value: cloud-kms-masterclass

  CMKAlias:
    Type: AWS::KMS::Alias
    Properties:
      AliasName: !Ref AliasName
      TargetKeyId: !Ref CMK

  Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
              KMSMasterKeyID: !Ref AliasName

Outputs:
  KeyId:
    Value: !Ref CMK
  BucketOut:
    Value: !Ref Bucket
Deploy:

bash
Copy code
STACK=day1-kms
BUCKET="ckms-${AWS_ACCOUNT_ID}-cf-$(date +%s)"
aws cloudformation deploy \
  --template-file aws/beginner/iac/cloudformation/day1-sse-kms.yaml \
  --stack-name "$STACK" \
  --parameter-overrides BucketName="$BUCKET" \
  --capabilities CAPABILITY_NAMED_IAM
3b) Terraform — aws/beginner/iac/terraform/main.tf
bash
Copy code
nano aws/beginner/iac/terraform/main.tf
Paste:

hcl
Copy code
terraform {
  required_version = ">= 1.3.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

resource "aws_kms_key" "cmk" {
  description             = "Masterclass Day1 CMK"
  key_usage               = "ENCRYPT_DECRYPT"
  is_enabled              = true
  deletion_window_in_days = 7
  tags = { project = "cloud-kms-masterclass" }
}

resource "aws_kms_alias" "alias" {
  name          = "alias/masterclass-cmk"
  target_key_id = aws_kms_key.cmk.key_id
}

resource "aws_s3_bucket" "bucket" {
  bucket = var.bucket_name
}

resource "aws_s3_bucket_server_side_encryption_configuration" "sse" {
  bucket = aws_s3_bucket.bucket.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_alias.alias.name
    }
  }
}

# EBS default encryption (account-level)
resource "aws_ebs_encryption_by_default" "default" {
  enabled = true
}

resource "aws_ebs_default_kms_key" "default" {
  key_arn = aws_kms_key.cmk.arn
}

output "bucket_name" { value = aws_s3_bucket.bucket.bucket }
output "kms_key_id"  { value = aws_kms_key.cmk.key_id }
variables.tf
bash
Copy code
nano aws/beginner/iac/terraform/variables.tf
Paste:

hcl
Copy code
variable "region" {
  type    = string
  default = "us-east-1"
}

variable "bucket_name" {
  type = string
}
outputs.tf
bash
Copy code
nano aws/beginner/iac/terraform/outputs.tf
Paste:

hcl
Copy code
output "bucket_name" { value = aws_s3_bucket.bucket.bucket }
output "kms_key_id"  { value = aws_kms_key.cmk.key_id }
Apply:

bash
Copy code
cd aws/beginner/iac/terraform
terraform init
terraform apply -auto-approve -var="bucket_name=ckms-${AWS_ACCOUNT_ID}-tf-$(date +%s)"
