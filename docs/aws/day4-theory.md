<p align="center">
  <img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" alt="Secure the Cloud Banner" width="70%">
</p>

# 🟧 Day 4 — AWS KMS Theory: CMK Rotation & Alias Automation  
_“Keys age, data doesn’t. Rotation keeps trust fresh.”_

---

## 🧭 Overview

AWS KMS allows you to control the **entire lifecycle of encryption keys** — from creation, labeling, rotation, and alias mapping, to archival and deletion.

By **rotating CMKs**, you ensure that:
- Compromised key versions don’t endanger historical data.
- Compliance frameworks (PCI DSS, FedRAMP, NIST 800-57) stay satisfied.
- Key material evolves while access policies and aliases stay stable.

---

## 🧠 Analogy — “Passport Renewal for Encryption Keys”

Imagine each key as a **passport**:
- Your **CMK** = the person’s identity.
- A **key version** = each renewed passport.
- The **alias** = your official name on every document.

When you renew your passport (rotate the key), all your existing IDs (aliases) still point to the same identity — only the internal passport number changes.

That’s AWS KMS rotation.

---

## 🔐 Key Components

| Concept | Description |
|----------|--------------|
| **Customer Managed Key (CMK)** | Your top-level KMS key object with attached metadata, policies, and alias. |
| **Key Material** | The actual cryptographic bytes — rotated when you enable rotation or reimport. |
| **Alias** | A stable reference name (`alias/app-prod`), always pointing to the active CMK. |
| **Key Policy** | Defines IAM access permissions for using or managing the key. |
| **Automatic Rotation** | Every 365 days (for symmetric CMKs) — AWS generates a new key version behind the scenes. |

---

## 🧩 Key Lifecycle Diagram

```mermaid
%%{init: {'theme':'base'}}%%
flowchart LR
  Create[Create CMK]
  Alias[Alias assigned (e.g., alias/app-prod)]
  Rotate[Automatic rotation (new key version)]
  Use[Applications use key via alias]
  Archive[Retire/deactivate old key version]

  Create --> Alias --> Use --> Rotate --> Use --> Archive
  Rotate -. "Every 365 days (auto)" .-> Use
⚙️ Automatic Rotation (Symmetric CMKs)
You can enable rotation with one command:

bash
Copy code
aws kms enable-key-rotation --key-id <your-key-id>
Check rotation status:

bash
Copy code
aws kms get-key-rotation-status --key-id <your-key-id>
Output:

json
Copy code
{
  "KeyRotationEnabled": true
}
Every 365 days, AWS creates a new key version, transparently used for all future encryptions, while old versions decrypt existing data.

🧩 Envelope encryption means you never need to re-encrypt everything — data keys are re-encrypted with the new CMK version during normal lifecycle events.

🧰 Manual Rotation (Reimported Material)
For imported CMKs (origin=EXTERNAL), automatic rotation is disabled.
To rotate manually:

Create a new CMK or reimport new material.

Update your alias to the new CMK.

Optionally disable the old key after a validation period.

bash
Copy code
aws kms create-key --description "CMK v2 for app-prod"
aws kms update-alias --alias-name alias/app-prod --target-key-id <new-key-id>
Now all apps using alias/app-prod automatically use the new CMK.

🧩 Alias & Automation Flow
mermaid
Copy code
%%{init: {'theme':'base'}}%%
sequenceDiagram
  participant Dev as DevOps / App
  participant Alias as Alias (alias/app-prod)
  participant KMS as AWS KMS
  participant CMK1 as CMK v1
  participant CMK2 as CMK v2

  Dev->>Alias: Encrypt(data)
  Alias->>KMS: Resolve to CMK v1
  KMS-->>Dev: CiphertextBlob(v1)

  Note over KMS,CMK1: Rotation event (auto/manual)

  Alias->>KMS: Now points to CMK v2
  KMS-->>Dev: CiphertextBlob(v2)
🧩 Rotation Governance & Audit
Control	Description
AWS CloudTrail	Logs all rotation events, key usage, and alias updates.
KMS Rotation Events	Captured under EnableKeyRotation, DisableKeyRotation, UpdateAlias.
Compliance Mapping	Rotation every 365 days satisfies NIST 800-57 and PCI DSS 3.6.4.
Change Management	Use aliases to avoid direct key ID dependency in apps.

🧩 Recommended Patterns
Alias-based access only → never hardcode CMK IDs.

Enable auto-rotation for all symmetric CMKs.

Manual rotation every 12 months for imported keys.

Audit alias history via CloudTrail for chain-of-custody validation.

Document version lineage (use tags like rotation-cycle:2025Q1).

⚠️ Pitfalls to Avoid
Mistake	Why it’s risky
Disabling rotation globally	Violates key hygiene policies.
Rotating without alias	Apps still using old CMK IDs will break.
Manual rotation without testing decrypt	Data may become unreadable.
Mixing imported and AWS-managed rotation	AWS-managed keys rotate automatically, EXTERNAL do not.

🧠 Quick Review
What’s the difference between alias and CMK ID in AWS KMS?

Why does auto rotation not apply to EXTERNAL keys?

How often does AWS rotate symmetric CMKs?

What’s the key benefit of using alias/app-prod instead of the raw CMK ID?

How can CloudTrail help verify compliance after rotation?

🔐 Next: Day 5 — Azure Managed HSM + Disk Encryption Sets (Intermediate)
