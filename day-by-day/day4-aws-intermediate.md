<p align="center">
  <img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" alt="Secure the Cloud Banner" width="70%">
</p>

# ☁️ Day 4 — AWS KMS (Intermediate): CMK Rotation & Alias Automation  
_Where keys evolve with time — and your encryption strategy breathes._

---

## 🧠 Concept: “The Clockwork Vault”

Think of AWS KMS as a **living vault** — its gears represent key versions that rotate automatically to keep your secrets fresh.  
Instead of one eternal master key, AWS KMS lets you **rotate the inner mechanism** while keeping the same alias at the door.  

You’re not replacing the vault, you’re **replacing its lock cylinder** — the key alias always points to the latest version.

```mermaid
%%{init: {'theme':'base'}}%%
flowchart LR
  Alias["🔑 Alias: app/secure-key"]
  V1["CMK v1 (2024)"]
  V2["CMK v2 (2025)"]
  V3["CMK v3 (2026)"]
  Alias --> V1
  V1 -.rotated.-> V2
  V2 -.rotated.-> V3
  note right of V3: All share the same alias<br>and same permissions
🎯 Goals
Automate CMK rotation in AWS KMS

Use aliases for seamless version switching

Apply Terraform for key + alias management

Verify rotation events and key policies

Visualize rotation flow with KMS + CloudTrail

🧩 Step 1 — Create KMS Key with Rotation Enabled
bash
Copy code
aws kms create-key \
  --description "App encryption CMK with auto-rotation" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS \
  --region us-east-1
Get the key ARN:

bash
Copy code
aws kms list-keys --region us-east-1
Enable rotation:

bash
Copy code
aws kms enable-key-rotation \
  --key-id <your-key-id> \
  --region us-east-1
🪄 Step 2 — Create Alias for Seamless Rotation
bash
Copy code
aws kms create-alias \
  --alias-name alias/app/secure-key \
  --target-key-id <your-key-id>
Aliases let apps stay stable while new versions rotate in the background.

To verify:

bash
Copy code
aws kms list-aliases --region us-east-1
⚙️ Step 3 — Terraform Automation Blueprint
hcl
Copy code
provider "aws" {
  region = "us-east-1"
}

resource "aws_kms_key" "app_key" {
  description         = "App CMK with auto rotation"
  enable_key_rotation = true
  deletion_window_in_days = 7
  tags = {
    Purpose = "AppEncryption"
  }
}

resource "aws_kms_alias" "app_alias" {
  name          = "alias/app/secure-key"
  target_key_id = aws_kms_key.app_key.key_id
}
Apply:

bash
Copy code
terraform init
terraform apply -auto-approve
🧾 Step 4 — Verify Rotation Status
bash
Copy code
aws kms get-key-rotation-status \
  --key-id <your-key-id>
✅ Expected Output:

json
Copy code
{
  "KeyRotationEnabled": true
}
🧰 Step 5 — Check Rotation History in CloudTrail
Search CloudTrail for:

bash
Copy code
EventName: "EnableKeyRotation"
Or query via CLI:

bash
Copy code
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=EnableKeyRotation \
  --region us-east-1
🧠 Analogy: The Keysmith’s Clock
Imagine your encryption key as a clock tower.

Every 365 days, the gears rotate (key version)

The clock face (alias) stays the same

Visitors (applications) always read the current time — without caring what gears are inside

Rotation is the quiet hum of security maturity.
No downtime, no reconfiguration — just steady evolution.

🔍 Diagram — AWS KMS Rotation Flow
mermaid
Copy code
flowchart LR
  App["🧩 Application"]
  Alias["🔑 Alias: app/secure-key"]
  V1["CMK v1"]
  V2["CMK v2"]
  Vault["🏦 KMS Vault"]

  App -->|"Encrypt/Decrypt via Alias"| Alias
  Alias --> V1
  V1 -.rotate→.-> V2
  V2 --> Vault
  Vault -.rotation logs.-> CloudTrail[(CloudTrail Logs)]
🧠 Quick Quiz
What’s the main benefit of using aliases in AWS KMS?

How often does AWS automatically rotate CMKs when enabled?

What CloudTrail event records a rotation?

What happens to old key versions after rotation?

<p align="center"> 🔐 <b>Next:</b> Day 5 — Azure Key Vault + Managed HSM Integration ⚡ </p> ```
