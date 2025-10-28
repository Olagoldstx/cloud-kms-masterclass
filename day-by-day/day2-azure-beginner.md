# 🟦 Day 2 — Azure Key Vault (Beginner): Encrypt Blob Storage with a Customer-Managed Key (CMK)

![Banner](../securethecloud.jpg)
---

## 🎯 Goals
- Create an **Azure Key Vault (AKV)** and a **symmetric CMK**  
- Configure **Blob Storage Server-Side Encryption with CMK (SSE-CMK)**  
- Assign least-privilege access using RBAC and Access Policies  
- Understand **Key Vault hierarchy** and **Envelope Encryption**  
- Verify encryption and rotation in portal and CLI

---

## 🧠 Analogy — “The Bank with Multiple Vault Doors”
If AWS KMS is a vault with one door per region, Azure Key Vault is a vault with **many doors** — each door opens to a different kind of secret:  
🔑 Keys for encryption 🧾 Secrets for passwords 📜 Certificates for TLS.  
You control who can open which door and for what purpose using **RBAC and Access Policies**.

```mermaid
flowchart LR
  subgraph Vault["🏦 Azure Key Vault"]
    K[(CMK)]
  end
  App["Blob Service or Client App"]
  Data["Blob Data (Encrypted)"]
  App -->|"Encrypt Request"| Vault
  Vault -->|"Return Wrapped DEK"| App
  App -->|"Upload Ciphertext + Encrypted DEK"| Data
  Vault -.audit/logs.-> Vault
⚙️ Prerequisites
bash
Copy code
az --version
# Ensure you have 'az account set' to your subscription
az account show --output table
Optional variables:

bash
Copy code
export RG="rg-ckms"
export LOC="eastus"
export KV="kv-ckms-$RANDOM"
export ST="ckms$RANDOM"
🚀 Step 1 — Create Resource Group (if not exists)
bash
Copy code
az group create -n $RG -l $LOC
🔑 Step 2 — Create Key Vault + Key
bash
Copy code
az keyvault create \
  --name $KV \
  --resource-group $RG \
  --location $LOC \
  --enable-rbac-authorization true

az keyvault key create \
  --vault-name $KV \
  --name masterclass-key \
  --protection software \
  --ops encrypt decrypt wrapKey unwrapKey
🪣 Step 3 — Create Storage Account with SSE-CMK
bash
Copy code
az storage account create \
  --name $ST \
  --resource-group $RG \
  --location $LOC \
  --sku Standard_LRS \
  --kind StorageV2 \
  --encryption-key-source Microsoft.Keyvault \
  --encryption-key-vault "https://$KV.vault.azure.net" \
  --encryption-key-name masterclass-key \
  --https-only true
🧩 Step 4 — Grant Blob Service Access to Key Vault
bash
Copy code
az ad sp list --display-name "Microsoft Storage" --query "[].appId"
# Usually returns a GUID like '12345678-...'

az role assignment create \
  --assignee <StorageAppID> \
  --role "Key Vault Crypto Service Encryption User" \
  --scope $(az keyvault show -n $KV -g $RG --query id -o tsv)
🔍 Step 5 — Verify Encryption
bash
Copy code
az storage account show -n $ST -g $RG --query 'encryption'
Expected:

json
Copy code
{
 "keySource": "Microsoft.Keyvault",
 "keyVaultProperties": {
   "keyName": "masterclass-key",
   "keyVaultUri": "https://kv-ckms-xxxx.vault.azure.net"
 }
}
🧱 Infrastructure as Code (Bicep)
bash
Copy code
nano azure/beginner/iac/bicep/day2-kv-ssek.bicep
bicep
Copy code
param rg string = resourceGroup().name
param location string = resourceGroup().location
param kvName string
param stName string

resource kv 'Microsoft.KeyVault/vaults@2023-02-01' = {
  name: kvName
  location: location
  properties: {
    tenantId: subscription().tenantId
    sku: { family: 'A'; name: 'standard' }
    enableRbacAuthorization: true
  }
}

resource key 'Microsoft.KeyVault/vaults/keys@2023-02-01' = {
  parent: kv
  name: 'masterclass-key'
  properties: {
    keySize: 2048
    kty: 'RSA'
    keyOps: [ 'encrypt', 'decrypt', 'wrapKey', 'unwrapKey' ]
  }
}

resource st 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: stName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {
    encryption: {
      keySource: 'Microsoft.Keyvault'
      keyvaultproperties: {
        keyName: key.name
        keyvaulturi: kv.properties.vaultUri
      }
    }
  }
}
Deploy:

bash
Copy code
az deployment group create -g $RG -f azure/beginner/iac/bicep/day2-kv-ssek.bicep \
  -p kvName=$KV stName=$ST
🧮 Terraform Variant (optional)
bash
Copy code
nano azure/beginner/iac/terraform/main.tf
hcl
Copy code
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name    = "rg-ckms"
  location = "eastus"
}

resource "azurerm_key_vault" "kv" {
  name = "kv-ckms-${random_integer.suffix.result}"
  location = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  tenant_id = data.azurerm_client_config.current.tenant_id
  sku_name = "standard"
  enable_rbac_authorization = true
}

resource "azurerm_key_vault_key" "cmk" {
  name = "masterclass-key"
  key_vault_id = azurerm_key_vault.kv.id
  key_type = "RSA"
  key_size = 2048
  key_opts = ["encrypt","decrypt","wrapKey","unwrapKey"]
}

resource "azurerm_storage_account" "st" {
  name = "ckms${random_integer.suffix.result}"
  resource_group_name = azurerm_resource_group.rg.name
  location = azurerm_resource_group.rg.location
  account_tier = "Standard"
  account_replication_type = "LRS"
  enable_https_traffic_only = true
  customer_managed_key {
    key_vault_key_id = azurerm_key_vault_key.cmk.id
    user_assigned_identity_id = null
  }
}

resource "random_integer" "suffix" {
  min = 10000
  max = 99999
}

data "azurerm_client_config" "current" {}
🔐 Verification and Audit
bash
Copy code
az monitor activity-log list --resource-id $(az keyvault show -n $KV -g $RG --query id -o tsv) --max-events 10 --query "[].{Operation:operationName.localizedValue, Caller:caller}"
Look for Encrypt, Decrypt, WrapKey, UnwrapKey events.

🧹 Cleanup
bash
Copy code
az group delete -n $RG -y
🧩 Quiz
What is the difference between Microsoft-managed keys and customer-managed keys?

How does Azure Key Vault enforce RBAC vs Access Policies?

What does --enable-rbac-authorization true do?

Which Azure service acts like AWS CloudTrail for key operations?

What would happen if you deleted the Key Vault used by the Storage Account?

✅ Next: Day 3 — Google Cloud KMS and BigQuery CMEK
