# ☁️ Day 8 Lab — Cross-Cloud Envelope Encryption with RBAC

![securethecloud](https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941)

---

## 🧪 Lab Objectives
- Create envelope keys in each cloud
- Use Terraform to automate key wrapping
- Apply RBAC at each layer
- Test re-encryption + audit verification

---

## 🔧 1. Terraform Structure
/terraform
├── aws/
│ └── kms.tf
├── azure/
│ └── keyvault.tf
└── gcp/
└── kms.tf

vbnet
Copy code

Each defines:
- CMK / Key Vault Key / CMEK  
- IAM roles / RBAC bindings  
- Outputs exposing ARNs, URIs, or KeyPaths

---

## 🧱 2. AWS Envelope Key Creation
```bash
aws kms create-key --description "CrossCloudEnvelopeKey"
aws kms create-alias --alias-name alias/env-key --target-key-id <KeyId>
aws kms generate-data-key --key-id alias/env-key --key-spec AES_256 > aws_key.json
🟦 3. Azure Wrap
bash
Copy code
az keyvault key import --vault-name cloudlabVault \
  --name env-key --pem-file ./aws_wrapped.pem
🟨 4. GCP Final Wrap
bash
Copy code
gcloud kms keys versions import \
 --import-job crosswrap-job \
 --location=us-central1 --keyring global-secure \
 --key cmk-final --algorithm rsa-oaep-4096-sha256 \
 --input-file aws_wrapped.pem
🔍 5. Verify Access Controls
bash
Copy code
aws kms get-key-policy --key-id alias/env-key
az keyvault key show --vault-name cloudlabVault --name env-key
gcloud kms keys get-iam-policy cmk-final --keyring global-secure --location=us-central1
📜 6. Validate Audits
AWS: CloudTrail → GenerateDataKey, Decrypt

Azure: Activity Logs → WrapKey, UnwrapKey

GCP: Audit Logs → CryptoKeyVersion.UseToEncrypt

✅ Cleanup
bash
Copy code
terraform destroy
🔒 Confirm deletion of envelope keys before proceeding to Day 9.
