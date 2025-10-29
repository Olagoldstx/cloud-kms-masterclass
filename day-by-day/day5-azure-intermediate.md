<p align="center">
  <img src="https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941" alt="Secure the Cloud Banner" width="70%">
</p>

# 🟦 Day 5 — Azure (Intermediate): Managed HSM + Disk Encryption Sets (DES)

---

## 🎯 Goals
- Provision **Azure Managed HSM** (FIPS L3) or use **Key Vault** as fallback
- Create a **CMK** and grant a **DES identity** permission to use it
- Create a **Disk Encryption Set** wired to the CMK
- Launch a VM whose **OS disk is encrypted by your DES**
- Verify encryption & audit operations

> Note: Managed HSM has regional quotas/pricing; if unavailable, use standard Key Vault with RBAC (same flow).

---

## 🧠 Analogy — “Hardware Vault + Golden Key”
Managed HSM = a **dedicated hardware vault**. You mint a **golden key (CMK)** there.  
A **Disk Encryption Set** is like a **keycard** referencing that golden key; any VM disk using that keycard is **locked by your CMK**.

---

## ⚙️ Prereqs
```bash
export RG="rg-ckms"
export LOC="eastus"
az group create -n $RG -l $LOC
1) (Option A) Create Managed HSM + key
bash
Copy code
# Create Managed HSM (may take ~20-30 min to become ready)
az keyvault mhsm create -n mhsm-ckms-$RANDOM -g $RG -l $LOC

# Enable RBAC model and get resource ID
MHHSM_ID=$(az keyvault mhsm show -n $(az keyvault mhsm list -g $RG --query "[0].name" -o tsv) -g $RG --query id -o tsv)

# Create a key in Managed HSM
az keyvault key create \
  --hsm-name $(az keyvault mhsm list -g $RG --query "[0].name" -o tsv) \
  --name cmk-des \
  --ops encrypt decrypt wrapKey unwrapKey \
  --kty RSA \
  --size 2048
1b) (Option B) Fallback to Key Vault + key
bash
Copy code
KV="kv-ckms-$RANDOM"
az keyvault create -n $KV -g $RG -l $LOC --enable-rbac-authorization true

az keyvault key create \
  --vault-name $KV \
  --name cmk-des \
  --protection software \
  --ops encrypt decrypt wrapKey unwrapKey
Record the key URL (one of):

bash
Copy code
# Managed HSM key ID
az keyvault key show --hsm-name $(az keyvault mhsm list -g $RG --query "[0].name" -o tsv) -n cmk-des --query key.kid -o tsv

# or Key Vault key ID
az keyvault key show --vault-name $KV -n cmk-des --query key.kid -o tsv
2) Create a User-Assigned Managed Identity for DES
bash
Copy code
UAI="uai-des-$RANDOM"
az identity create -g $RG -n $UAI -l $LOC
UAI_ID=$(az identity show -g $RG -n $UAI --query id -o tsv)
UAI_PRINCIPAL=$(az identity show -g $RG -n $UAI --query principalId -o tsv)
Grant the identity permission to use the key (data-plane):

If using Key Vault:

bash
Copy code
az role assignment create \
  --assignee $UAI_PRINCIPAL \
  --role "Key Vault Crypto User" \
  --scope $(az keyvault show -n $KV -g $RG --query id -o tsv)
If using Managed HSM:

bash
Copy code
# Assign at HSM scope; role may be "Managed HSM Crypto User" depending on RBAC model
az role assignment create \
  --assignee $UAI_PRINCIPAL \
  --role "Managed HSM Crypto User" \
  --scope $MHHSM_ID
3) Create Disk Encryption Set (DES) pointing to the CMK
bash
Copy code
KEY_URL=$(az keyvault key show --vault-name $KV -n cmk-des --query key.kid -o tsv 2>/dev/null || true)
if [ -z "$KEY_URL" ]; then
  KEY_URL=$(az keyvault key show --hsm-name $(az keyvault mhsm list -g $RG --query "[0].name" -o tsv) -n cmk-des --query key.kid -o tsv)
fi

DES="des-ckms-$RANDOM"
az disk-encryption-set create \
  -g $RG -n $DES \
  --location $LOC \
  --key-url $KEY_URL \
  --identity-type UserAssigned \
  --identity-ids $UAI_ID
Grant DES’s own system identity access to the key (Azure auto-creates a principal for the DES resource):

bash
Copy code
DES_ID=$(az disk-encryption-set show -g $RG -n $DES --query id -o tsv)
DES_PRINCIPAL=$(az disk-encryption-set show -g $RG -n $DES --query identity.principalId -o tsv)

# KV model:
az role assignment create \
  --assignee $DES_PRINCIPAL \
  --role "Key Vault Crypto Service Encryption User" \
  --scope $(az keyvault show -n $KV -g $RG --query id -o tsv) 2>/dev/null || true

# M-HSM model (role name may vary by portal label):
az role assignment create \
  --assignee $DES_PRINCIPAL \
  --role "Managed HSM Crypto Service Encryption User" \
  --scope $MHHSM_ID 2>/dev/null || true
4) Create a VM with OS disk encrypted by DES
bash
Copy code
VM="vm-des-$RANDOM"
az vm create \
  -g $RG -n $VM -l $LOC \
  --image Ubuntu2204 \
  --generate-ssh-keys \
  --os-disk-security-type ConfidentialVM_DiskEncryptedWithCustomerKey \
  --disk-encryption-set $DES
✅ Verify
bash
Copy code
az vm show -g $RG -n $VM --query "storageProfile.osDisk.encryption.type" -o tsv
az disk show -g $RG -n ${VM}OsDisk --query "encryption.diskEncryptionSetId" -o tsv
🧹 Cleanup (optional)
bash
Copy code
az group delete -n $RG -y
🧠 Quiz
Why does a Disk Encryption Set need its own identity, separate from the VM?

Difference between Key Vault and Managed HSM for CMK storage?

Which RBAC roles are needed to let DES use the key?

What breaks if you disable or delete the CMK after attaching DES to disks?
