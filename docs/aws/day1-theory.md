<p align="center"><img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" width="70%"></p>

# 🟧 Day 1 — AWS KMS Theory: CMK Fundamentals & Envelope Encryption
_“Keys protect data, but KMS protects the keys.”_

## 🧭 What you’ll learn
- CMK vs data keys, AWS-managed vs customer-managed vs imported keys
- **Envelope encryption** (GenerateDataKey / Decrypt)
- Policies, grants, CloudTrail, and least privilege design

## 🧠 Analogy: “Box in a Box”
Data is locked with a **data key** (inner box).  
That data key is locked by the **CMK** (outer box in KMS).  
Apps never handle the CMK — only the **wrapped data key (EDK)**.

```mermaid
sequenceDiagram
  participant App as Application
  participant KMS as AWS KMS (CMK)
  App->>KMS: GenerateDataKey(CMK)
  KMS-->>App: Plaintext DEK + Encrypted DEK (EDK)
  App->>App: Encrypt data with DEK
  App->>KMS: Decrypt(EDK) to recover DEK when needed
🔑 Key types
Type	Control	Notes
AWS-managed CMK	AWS	Created per service (S3/EBS etc.)
Customer-managed CMK	You	Rotation, aliases, policies
Imported (EXTERNAL)	You	You supply key material; no auto-rotation
AWS-owned	AWS	Not visible to you

🧱 Minimal policy pattern
json
Copy code
{
 "Version":"2012-10-17",
 "Statement":[
  {"Sid":"Root","Effect":"Allow","Principal":{"AWS":"arn:aws:iam::764265373335:root"},"Action":"kms:*","Resource":"*"},
  {"Sid":"AppUse","Effect":"Allow","Principal":{"AWS":"arn:aws:iam::764265373335:user/cloudlab-user"},
   "Action":["kms:Encrypt","kms:Decrypt","kms:GenerateDataKey"],"Resource":"*"}
 ]}
🧪 CLI quick demo
bash
Copy code
# request a data key from alias/app/prod
aws kms generate-data-key --key-id alias/app/prod --key-spec AES_256 --region us-east-1 > /tmp/dk.json
PLAINTEXT=$(jq -r .Plaintext /tmp/dk.json | base64 -d)
echo "secret" > msg.txt
# encrypt locally with DEK (example using OpenSSL)
openssl enc -aes-256-cbc -pbkdf2 -salt -in msg.txt -out msg.enc -pass file:<(printf "%s" "$PLAINTEXT")
# store jq -r .CiphertextBlob (the EDK) with the ciphertext
🧩 Service patterns
Service	Uses KMS to…
S3	SSE-KMS for object encryption with CMK wrapping
EBS	Per-volume DEKs wrapped by CMK
RDS	DB storage encrypted; snapshots inherit
Lambda	Env vars / secrets encryption

🧾 Audit & compliance
CloudTrail logs: Encrypt, Decrypt, GenerateDataKey, policy/alias changes

Map to NIST 800-57, PCI DSS 3.6.x controls

✅ Checklist
Use aliases (not raw key IDs)

Enable rotation where applicable

Least privilege on key policy + IAM

Centralize CloudTrail to SIEM

