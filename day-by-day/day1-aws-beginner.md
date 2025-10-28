![securethecloud](https://github.com/user-attachments/assets/2a717544-7b0f-41a8-b05e-8ff1be77ac7b)


# Day 1 — AWS KMS (Beginner): S3 + EBS with Customer Managed Key

## 🎯 Goals
- Create a **symmetric CMK** (customer managed key) in **AWS KMS**
- Enable **SSE-KMS** on an **S3 bucket**
- Turn on **EBS default encryption** with your CMK
- Prove encryption & log key usage with **CloudTrail**
- Understand **envelope encryption** visually

## 🧠 Analogy
Think of KMS as a **bank vault**. The **master key (CMK/KEK)** never leaves the vault.  
You generate a **box key (DEK)** to lock your file. You store the **box (ciphertext)** alongside the **wrapped DEK**.  
When you open the box, the vault unwraps the DEK for you—**you never hold the master key**.


flowchart LR
  subgraph KMS["🏦 AWS KMS"]
    KEK[(CMK/KEK)]
  end
  ```mermaid
  A[Plaintext file] -->|Generate DEK| B((DEK))
  B -->|Encrypt| C[Ciphertext]
  B -->|Encrypt with KEK| D[Encrypted DEK]
  C --> E[(S3 Object)]
  D --> E
  KEK -.rotation/audit.-> KEK
```

⚙️ Prereqs
AWS CLI v2, jq installed (sudo apt-get install -y jq if needed)

Active profile with permissions to create KMS, S3, EC2 settings

Default region: us-east-1

Optional env (edit if you want):

bash
Copy code
export AWS_REGION=us-east-1
export AWS_ACCOUNT_ID=764265373335
🚀 Step 1 — Create a CMK and alias
bash
Copy code
aws kms create-key \
  --description "Masterclass Day1 CMK" \
  --key-usage ENCRYPT_DECRIPT \
  --origin AWS_KMS \
  --tags TagKey=project,TagValue=cloud-kms-masterclass \
  --region "$AWS_REGION" > /tmp/k.json

# (typo guard) If you see an error on ENCRYPT_DECRIPT, use ENCRYPT_DECRYPT:
# --key-usage ENCRYPT_DECRYPT

KEY_ID=$(jq -r '.KeyMetadata.KeyId' /tmp/k.json)

aws kms create-alias \
  --alias-name alias/masterclass-cmk \
  --target-key-id "$KEY_ID" \
  --region "$AWS_REGION"

echo "CMK KeyId: $KEY_ID"
Verify:

bash
Copy code
aws kms describe-key --key-id alias/masterclass-cmk --region "$AWS_REGION" \
  --query 'KeyMetadata.[KeyId,KeyState,KeyManager,Arn]'
🪣 Step 2 — S3 bucket with SSE-KMS (your CMK)
bash
Copy code
BUCKET="ckms-${AWS_ACCOUNT_ID}-$(date +%s)"
aws s3api create-bucket --bucket "$BUCKET" \
  --region "$AWS_REGION" \
  --create-bucket-configuration LocationConstraint="$AWS_REGION"

aws s3api put-bucket-encryption --bucket "$BUCKET" \
  --server-side-encryption-configuration '{
    "Rules":[{"ApplyServerSideEncryptionByDefault":{
      "SSEAlgorithm":"aws:kms","KMSMasterKeyID":"alias/masterclass-cmk"}}]
  }'

# Upload a test object
echo "hello kms day1" > /tmp/hello.txt
aws s3 cp /tmp/hello.txt s3://$BUCKET/test/hello.txt

# Inspect encryption headers
aws s3api head-object --bucket "$BUCKET" --key "test/hello.txt" \
  --query '[ServerSideEncryption, SSEKMSKeyId]'
Expected:

css
Copy code
["aws:kms", "arn:aws:kms:us-east-1:764265373335:key/...."]
💽 Step 3 — EBS default encryption with your CMK
bash
Copy code
aws ec2 enable-ebs-encryption-by-default --region "$AWS_REGION"
aws ec2 modify-ebs-default-kms-key-id \
  --kms-key-id alias/masterclass-cmk \
  --region "$AWS_REGION"

# Verify defaults
aws ec2 get-ebs-default-kms-key-id --region "$AWS_REGION"
aws ec2 get-ebs-encryption-by-default --region "$AWS_REGION"
🔍 Step 4 — Prove usage in CloudTrail
bash
Copy code
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceName,AttributeValue=alias/masterclass-cmk \
  --max-results 10 | jq '.Events[] | {EventTime,EventName,Username}'
Look for events like GenerateDataKey, Decrypt, etc.

🧪 Troubleshooting
AccessDenied: your IAM user/role needs KMS, S3, and EC2 permissions (AdministratorAccess or scoped policies).

Bucket region mismatch: ensure --region matches bucket’s region.

SSE-KMS not set: rerun put-bucket-encryption and re-upload.

KMS alias not found: confirm describe-key for alias/masterclass-cmk.

🧹 Clean Up (avoid charges)
bash
Copy code
# Remove S3 objects and bucket
aws s3 rm s3://$BUCKET --recursive
aws s3api delete-bucket --bucket "$BUCKET"

# Schedule key deletion (min 7 days)
aws kms schedule-key-deletion --key-id "$KEY_ID" --pending-window-in-days 7 --region "$AWS_REGION"
📝 Quiz (answer in your own words)
Why use SSE-KMS instead of SSE-S3 for sensitive buckets?

What is the difference between a CMK (KEK) and a DEK?

Which command showed evidence of encryption at rest for your test object?

Which service logs KMS usage for audits?

✅ Next: Day 2 (Azure Key Vault + Blob with CMK)
