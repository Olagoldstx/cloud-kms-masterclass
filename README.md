<p align="center">
  <img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" 
       alt="Secure the Cloud Banner" 
       width="70%" 
       style="border-radius: 15px; box-shadow: 0 0 10px rgba(0,0,0,0.25);">
</p>


# 🧭 Cloud KMS Masterclass (AWS · Azure · GCP)
_A multi-cloud encryption journey — from zero to hero._

---

## 🎓 Welcome to the Masterclass
Keys are the **crown jewels of cloud security** — and you’re about to become the **keysmith**.  
This course takes you from KMS beginner to **multi-cloud encryption architect** by mastering:

- 🔐 **AWS KMS**
- 🏦 **Azure Key Vault / Managed HSM**
- ☁️ **Google Cloud KMS / External Key Manager**

Each day includes: deep theory, CLI & IaC, diagrams, cloud-native verification, and a quiz/anki set.


# 🛡️ Cloud KMS Masterclass (AWS · Azure · GCP)
_A multi-cloud encryption journey — from zero to governance._

---

## 🗺️ 10-Day Roadmap · Interactive Portal

<table>
<tr>
<td align="center" width="33%">

### 🟧 Day 1 — AWS  
**S3 + EBS with Customer-Managed Key**  
[📘 Theory](docs/aws/day1-theory.md) • [🔗 Lab](day-by-day/day1-aws-beginner.md)

</td>
<td align="center" width="33%">

### 🟦 Day 2 — Azure  
**Blob Storage Encryption with AKV CMK**  
[📘 Theory](docs/azure/day2-theory.md) • [🔗 Lab](day-by-day/day2-azure-beginner.md)

</td>
<td align="center" width="33%">

### 🟨 Day 3 — GCP  
**BigQuery CMEK + Cloud Storage Encryption**  
[📘 Theory](docs/gcp/day3-theory.md) • [🔗 Lab](day-by-day/day3-gcp-beginner.md)

</td>
</tr>

<tr>
<td align="center">

### 🟧 Day 4 — AWS  
**CMK Rotation + Aliases Automation**  
[📘 Theory](docs/aws/day4-theory.md) • [🔗 Lab](day-by-day/day4-aws-intermediate.md)

</td>
<td align="center">

### 🟦 Day 5 — Azure  
**Managed HSM + Disk Encryption Sets**  
[📘 Theory](docs/azure/day5-theory.md) • [🔗 Lab](day-by-day/day5-azure-intermediate.md)

</td>
<td align="center">

### 🟨 Day 6 — GCP  
**External Key Manager (EKM) Integration**  
[📘 Theory](docs/gcp/day6-theory.md) • [🔗 Lab](day-by-day/day6-gcp-intermediate.md)

</td>
</tr>

<tr>
<td align="center">

### ☁️ Day 7 — Cross-Cloud  
**BYOK (AWS ↔ Azure ↔ GCP)**  
[📘 Theory](docs/cross-cloud/day7-theory.md) • [🔗 Lab](day-by-day/day7-cross-cloud-intermediate.md)

</td>
<td align="center">

### ☁️ Day 8 — Cross-Cloud  
**Envelope Encryption Deep Dive + RBAC**  
[📘 Theory](docs/cross-cloud/day8-theory.md) • [🔗 Lab](day-by-day/day8-crosscloud-intermediate.md)

</td>
<td align="center">

### ☁️ Day 9 — Cross-Cloud  
**Key Governance, RBAC & Audit Automation**  
[📘 Theory](docs/cross-cloud/day9-theory.md) • [🔗 Lab](day-by-day/day9-crosscloud-advanced.md)

</td>
</tr>

<tr>
<td colspan="3" align="center">

### ☁️ Day 10 — Capstone  
**Unified Multi-Cloud KMS Security Dashboard**  
[📘 Theory](docs/cross-cloud/day10-theory.md) • [🔗 Lab](day-by-day/day10-capstone-dashboard.md)

</td>
</tr>
</table>

---

## 📚 Resources & Directories

**Azure Day 5 Theory:** [Managed HSM vs Key Vault + DES](docs/azure/day5-theory.md)
---

## 🧩 Repository Map
```plaintext
cloud-kms-masterclass/
├── README.md
├── day-by-day/
│   ├── day1-aws-beginner.md
│   ├── day2-azure-beginner.md
│   └── day3-gcp-beginner.md
├── docs/
│   ├── aws/
│   ├── azure/
│   ├── gcp/
│   └── cross-cloud/
├── common/
│   ├── diagrams/
│   └── scripts/
└── anki/
🧠 Encryption Mental Model
Think of KMS as a digital vault. The master key (CMK) never leaves the vault.
Workloads get a temporary data key (DEK) to encrypt data; that DEK is then encrypted with your CMK — envelope encryption.

```mermaid
flowchart LR
    subgraph Vault[🔐 Key Management Service]
        CMK[Customer Master Key<br/>Stored in HSM]
    end
    
    DEK[📄 Data Encryption Key<br/>Ephemeral / Wrapped]
    Data[💾 Encrypted Data<br/>Stored in Object Storage]
    Audit[(Audit Logs)]
    
    CMK -->|Wraps & Unwraps| DEK
    DEK -->|Encrypts & Decrypts| Data
    Vault -.->|Audit Trail &<br/>Access Logging| Audit
    
    style Vault fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style CMK fill:#e1f5fe,stroke:#0288d1
    style DEK fill:#fff3e0,stroke:#f57c00
    style Data fill:#e8f5e8,stroke:#388e3c
    style Audit fill:#f5f5f5,stroke:#757575
```

🧱 Automation Stack
🐍 CLI / SDK: AWS CLI · Azure CLI · gcloud

⚙️ IaC: Terraform · CloudFormation · Bicep

🔒 Security: Least Privilege · RBAC · CMK Isolation

🧾 Auditing: CloudTrail · Activity Logs · Audit Logs

🧠 Study Tip
“Encryption without key mastery is illusion — this course makes you the keysmith.” 🔐


docs/ → All textbook theory chapters
day-by-day/ → Hands-on labs
terraform/ → IaC templates
slides/ → Presentation materials
anki/ → Flashcards

git clone https://github.com/Olagoldstx/cloud-kms-masterclass.git
cd cloud-kms-masterclass
nano day-by-day/day1-aws-beginner.md

<p align="center">💥 <b>Let’s roll — your journey to KMS mastery begins here.</b> 🚀</p> ```

“Governance is the bridge between security and accountability.” — Secure the Cloud Dev Labs
