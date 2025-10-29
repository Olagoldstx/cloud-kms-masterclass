# ☁️ Day 9 Lab — Multi-Cloud Key Governance & Audit Automation

![securethecloud](https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941)

---

## 🎯 Goal
Implement cross-cloud automation that:
- Enforces key rotation
- Aggregates audit logs
- Applies least-privilege RBAC

---

## 🧩 Lab Overview
This lab builds a **Governance-as-Code** pipeline using Terraform and native cloud tools.

terraform/
├── aws/
│ ├── kms.tf
│ └── cloudtrail.tf
├── azure/
│ ├── keyvault.tf
│ └── monitor.tf
└── gcp/
├── kms.tf
└── logging.tf

yaml
Copy code

---

## 🧱 Step 1: Enable Rotation Everywhere

**AWS**
```bash
aws kms enable-key-rotation --key-id alias/prod-cmk
Azure

bash
Copy code
az keyvault key rotate --vault-name cloudlabVault --name cmk-prod
GCP

bash
Copy code
gcloud kms keys update cmk-prod --rotation-period=90d
⚙️ Step 2: RBAC Role Assignments
AWS

bash
Copy code
aws iam attach-user-policy --user-name cloudlab-user \
  --policy-arn arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser
Azure

bash
Copy code
az role assignment create \
 --assignee "<objectId>" \
 --role "Key Vault Crypto User" \
 --scope "/subscriptions/<subid>/resourceGroups/rg/providers/Microsoft.KeyVault/vaults/cloudlabVault"
GCP

bash
Copy code
gcloud kms keys add-iam-policy-binding cmk-prod \
  --location=us-central1 --keyring global-secure \
  --member="user:cloudlab@example.com" \
  --role="roles/cloudkms.cryptoKeyEncrypterDecrypter"
📊 Step 3: Audit Aggregation (SIEM)
Forward all logs to a central SIEM bucket:

AWS → S3 + Lambda Forwarder

Azure → Log Analytics Workspace

GCP → Logging Sink → Pub/Sub → SIEM

Example Terraform resource:

hcl
Copy code
resource "aws_s3_bucket" "siem_logs" {
  bucket = "securethecloud-siem"
  lifecycle_rule {
    id      = "retain"
    enabled = true
    transition {
      days          = 30
      storage_class = "GLACIER"
    }
  }
}
🔁 Step 4: Policy-as-Code Validation
Integrate OPA (Open Policy Agent):

rego
Copy code
package kms.rotation

deny[msg] {
  input.key.rotation_period > 90
  msg = sprintf("Key %s rotation exceeds policy", [input.key.id])
}
✅ Step 5: Validate Everything
Each key rotates successfully

RBAC only allows authorized identities

All encryption/decryption operations logged centrally

🏁 Next: Day 10 — Unified Multi-Cloud KMS Dashboard
