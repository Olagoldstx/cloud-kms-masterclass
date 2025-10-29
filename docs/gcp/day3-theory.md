<p align="center"><img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" width="70%"></p>

# 🟨 Day 3 — GCP Theory: CMEK for Cloud Storage & BigQuery

## 🧭 What you’ll learn
- **Cloud KMS** key ring / key / version model
- **CMEK** for Storage buckets and BigQuery datasets
- Roles, rotation, and audit logs

## 🧠 Analogy: “Library & Master Key”
The **key ring** is a shelf; each **key** is a book; each **version** is a new edition.  
Services borrow the book (CMK) to **wrap/unwrap** data keys.

```mermaid
flowchart LR
  KMS["🏦 Cloud KMS (CMEK)"]
  GCS["📦 Cloud Storage"]
  BQ["📊 BigQuery"]
  GCS -->|"Uses CMEK"| KMS
  BQ -->|"Uses CMEK"| KMS
🔑 Roles
roles/cloudkms.admin — manage rings/keys

roles/cloudkms.cryptoKeyEncrypterDecrypter — use for wrap/unwrap

Service agents for Storage / BigQuery need access to the key

🔁 Rotation
New key versions created on schedule; services pick latest

Old versions remain for decrypt

🧾 Audit
Cloud Audit Logs for KMS + services

Label keys (env=prod, app=…) for governance

✅ Checklist
Same location for bucket/dataset and KMS key

Grant least privilege to service accounts

Track key versions and rotation cadence

yaml
Copy code

---

# 4) Cross-Cloud Day 7 Theory – BYOK (Rich version)

```bash
nano docs/cross-cloud/day7-theory.md
Paste:

markdown
Copy code
<p align="center"><img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" width="70%"></p>

# ☁️ Day 7 — Theory: Cross-Cloud BYOK (AWS ↔ Azure ↔ GCP)

## 🧭 What you’ll learn
- Practical **BYOK** patterns that differ by cloud
- Reusing **AES-256** bytes in **AWS + GCP**; RSA import for **Azure KV**
- Governance: aliases, rotation rhythm, and audit unification

## 🧠 Mental model: “One Master, Three Doors”
You hold the **master key**; each cloud gets a compatible **door key**.

```mermaid
flowchart LR
  YOU["🏠 Your HSM/Offline"]
  AWS["🟧 AWS KMS (AES import)"]
  AZ["🟦 Azure KV (RSA import) / MHSM (AES)"]
  GCP["🟨 GCP KMS (AES import job)"]
  YOU --> AWS
  YOU --> AZ
  YOU --> GCP
🔑 Reality matrix
Cloud	Typical BYOK	Same bytes as others?
AWS KMS	AES-256 import (EXTERNAL)	✅ with GCP
Azure Key Vault	RSA-2048/EC import	❌ (algo differs)
Azure Managed HSM	AES (oct-HSM)	⚠️ advanced
GCP KMS	AES-256 via Import Job	✅ with AWS

🧾 Governance handoff
Aliases/labels aligned: alias/byok/app-prod, app=prod

Rotation cadence (e.g., 90/180 days) per cloud, same calendar

Audit into one SIEM: CloudTrail + Activity Logs + Audit Logs

⚠️ Pitfalls
Expecting identical keys in all three clouds without MHSM

Region/key-version mismatches

Losing imported material ⇒ data unrecoverable
