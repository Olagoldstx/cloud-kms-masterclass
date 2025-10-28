# 🛡️ **Cloud KMS Masterclass (AWS · Azure · GCP)**  
_A multi-cloud encryption journey — from zero to hero._

---

## ☁️ **Welcome to the Masterclass**
> “Keys are the crown jewels of cloud security — and you’re the locksmith.” 🔑  

This course transforms you from a KMS beginner to a **multi-cloud encryption architect** by mastering:
- **AWS KMS**
- **Azure Key Vault / Managed HSM**
- **Google Cloud KMS / EKM**

Each lab fuses:
- 🧠 Deep theory + analogies  
- ⚙️ CLI & Terraform / IaC  
- 🪄 Mermaid diagrams (visual encryption flows)  
- 🧩 Real-world cross-cloud integration  
- 🎯 Quizzes + flashcards for muscle memory  

---

## 🎯 **Learning Objectives**

| Level | Goals |
|-------|-------|
| **Beginner (Days 1 – 3)** | Understand KMS basics, CMKs, symmetric vs asymmetric keys, encrypt data at rest across S3, Blob, GCS |
| **Intermediate (Days 4 – 6)** | Master CMK rotation, grants, BYOK, and service-linked encryption (EBS, RDS, BigQuery CMEK) |
| **Advanced (Days 7 – 10)** | Implement HSM, EKM, automation, and cross-cloud CMEK policies for zero-trust data control |

---

## 🧭 **Repository Map**

cloud-kms-masterclass/
├── aws/
│ ├── beginner/
│ ├── intermediate/
│ └── advanced/
├── azure/
│ ├── beginner/
│ ├── intermediate/
│ └── advanced/
├── gcp/
│ ├── beginner/
│ ├── intermediate/
│ └── advanced/
├── common/
│ ├── diagrams/
│ ├── scripts/
│ └── resources/
└── day-by-day/
├── day1-aws-beginner.md
├── day2-azure-beginner.md
└── day3-gcp-beginner.md

pgsql
Copy code

---

## 🔒 **The Encryption Mental Model**

> Think of KMS as a **bank vault**:
> - The **master vault key (CMK/KEK)** never leaves the vault.  
> - You mint **temporary box keys (DEKs)** to lock your data.  
> - You store each encrypted box (ciphertext) alongside a wrapped DEK.  
> - When you need to open the box, KMS unwraps the DEK behind the scenes.

```mermaid
flowchart LR
  subgraph KMS_VAULT["🏦 Cloud KMS Vault"]
    KEK[(CMK/KEK)]
  end
  A[Plaintext Data] -->|Generate DEK| B((Data Encryption Key))
  B -->|Encrypt| C[Ciphertext]
  B -->|Wrap with KEK| D[Encrypted DEK]
  C --> E[(S3 / Blob / GCS Object)]
  D --> E
  KEK -.rotation/audit.-> KEK
⚙️ Hands-On Labs
Day	Focus	Tools
Day 1 – AWS KMS	Create CMK, S3 SSE-KMS, EBS Default Encryption	AWS CLI + CloudFormation + Terraform
Day 2 – Azure Key Vault	Generate CMK, enable SSE with CMK for Blob Storage	Azure CLI + ARM/Bicep
Day 3 – GCP Cloud KMS	Create key ring, key, CMEK for GCS and BigQuery	gcloud + Deployment Manager
Day 4–6 – Intermediate	Rotation / BYOK / Audit	IaC + Cross-Cloud Automation
Day 7–10 – Advanced	EKM / HSM / Multi-Region Encryption	Terraform + Automation + Dashboards

🧰 Prerequisites
✅ Installed tools

bash
Copy code
git --version
aws --version
az --version
gcloud --version
terraform --version
✅ Cloud accounts with permission to create:

AWS KMS / S3 / EBS / RDS

Azure Key Vault / Storage / Managed HSM

GCP KMS / GCS / BigQuery

✅ Default regions:

AWS → us-east-1

Azure → eastus

GCP → us-central1

📚 Cross-Cloud Concept Map
mermaid
Copy code
graph TD
  subgraph AWS
    A1[S3 Bucket]
    A2[EBS Volume]
    A3[KMS Key]
  end
  subgraph Azure
    B1[Blob Storage]
    B2[Managed Disk]
    B3[Key Vault Key]
  end
  subgraph GCP
    C1[GCS Bucket]
    C2[BigQuery Dataset]
    C3[Cloud KMS Key]
  end

  A3 <--> B3 <--> C3
  A1 --> A3
  B1 --> B3
  C1 --> C3
  A2 --> A3
  B2 --> B3
  C2 --> C3
  A3 -.sync.-> C3
  B3 -.policy.-> A3
🧩 Next Steps
🎬 Clone this repo:

bash
Copy code
git clone https://github.com/Olagoldstx/cloud-kms-masterclass.git
cd cloud-kms-masterclass
✏️ Open each lab with nano and follow along.

💾 Commit your progress:

bash
Copy code
git add .
git commit -m "feat: complete day1 aws kms lab"
git push
🏁 Your Journey Starts Now
“Encryption without key mastery is illusion — this course makes you the keysmith.” 🔐

Let’s roll 🚀
