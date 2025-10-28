# 🧭 Day 1 — AWS KMS Theory and Architecture  
_A foundation in managed cryptography for the cloud architect._

---

## 1️⃣  The Big Idea — “Locks, Keys & Trust”

Imagine every object in the cloud as a **treasure chest**.  
When you store it (in S3 or EBS), AWS wraps it with a **lock**.  
But who owns the **master key**?  
→ **You do**, through **AWS Key Management Service (KMS).**

KMS keeps the **Key Encryption Keys (KEKs)** sealed in a hardware vault (HSM).  
It issues **Data Encryption Keys (DEKs)** to your workloads, uses them to lock the data, then destroys them.  
The master key never leaves the vault.

```mermaid
flowchart LR
  subgraph Vault["🏦 AWS KMS HSM Vault"]
    KEK[(CMK / KEK)]
  end
  App[App / Service] -->|requests DEK| Vault
  Vault -->|GenerateDataKey| DEK[(DEK)]
  DEK -->|Encrypts| Data[(Your Data)]
  DEK -->|Wrapped by KEK| EncDEK[Wrapped DEK]
  Data -->|Stored with| EncDEK
  Vault -.Rotate/Audit.-> Vault
2️⃣ Key Hierarchy and Terminology
Layer	Purpose	Lifetime	Example
CMK (Customer Managed Key)	Root KEK stored in HSM	Long-term	alias/masterclass-cmk
DEK (Data Encryption Key)	Encrypts actual data	Ephemeral	Random AES-256
Envelope Encryption	Wrap DEK with CMK	Per object	Used in SSE-KMS
Grants & Policies	Who can use the key	Continuous	IAM & KMS grant docs

3️⃣ Shared Responsibility View
AWS KMS	You (the customer)
Operates HSM fleet, FIPS 140-2 L3 validated	Create and manage CMKs, aliases
Secure storage of key material	Define IAM/KMS policies and grants
Key rotation infrastructure	Choose rotation intervals
Regional replication control	Determine cross-region strategy

4️⃣ Common Integration Patterns
S3 SSE-KMS — Encrypt each object with a DEK wrapped by your CMK

EBS Default Encryption — All volumes use the CMK automatically

RDS / Secrets Manager / Lambda — Services invoke KMS API to encrypt data before storage

CloudTrail + CloudWatch — Track GenerateDataKey, Decrypt, DisableKey events

5️⃣ Envelope Encryption Explained
mermaid
Copy code
sequenceDiagram
  participant App as App
  participant KMS as AWS KMS
  participant S3 as S3 Storage
  App->>KMS: GenerateDataKey(CMK)
  KMS-->>App: {Plaintext DEK, Encrypted DEK}
  App->>S3: Upload(Encrypt(Data, Plaintext DEK), Encrypted DEK)
  S3->>KMS: Decrypt(Encrypted DEK)
  KMS-->>S3: Plaintext DEK
  S3->>App: Return(Decrypted Data)
6️⃣ Security Practices Checklist
 Use CMKs instead of AWS-managed keys for sensitive data

 Enable automatic rotation (every 365 days)

 Restrict access with kms:ViaService and kms:EncryptionContext

 Audit with CloudTrail and AWS Config

 Prefer envelope encryption over client-side DIY

7️⃣ Mini-Case Study
Scenario: You run a healthcare SaaS storing images in S3.
Requirement: HIPAA encryption and key rotation.
Solution:

Create alias/hipaa-cmk in KMS.

Enable S3 bucket SSE-KMS with that alias.

Schedule rotation.

Forward CloudTrail logs to Security Hub.

Result: Full data at-rest encryption with centralized auditing.

8️⃣ Reflection & Quiz
What’s the difference between SSE-S3 and SSE-KMS?

Why can’t you download the CMK material even as root?

What events prove KMS use in CloudTrail?

How does envelope encryption help with key rotation?

How would you use kms:EncryptionContext for fine-grained control?

🧩 References & Further Reading
AWS KMS Developer Guide

NIST SP 800-57 Pt 1 – Key Management Guidelines

AWS Encryption SDK Docs

CloudTrail Data Events Reference

## 🧱 Textbook Chapters

| Cloud | Chapter | Description |
|--------|----------|-------------|
| **AWS** | [Day 1 – Theory](docs/aws/day1-theory.md) | CMK fundamentals and envelope encryption |
| **Azure** | (coming next) | Key Vault and SSE with CMK |
| **GCP** | (coming next) | Cloud KMS and BigQuery CMEK |
| **Cross-Cloud** | (future) | BYOK, EKM and Multi-Region Strategies |
