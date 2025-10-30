# ☁️ Day 10 Lab — Unified Multi-Cloud KMS Security Dashboard (Capstone)

![securethecloud](https://github.com/user-attachments/assets/0ce41038-66c2-4146-a1ab-674790ecf941)

---

## 🎯 Lab Goal
Build a unified dashboard that aggregates AWS KMS, Azure Key Vault, and GCP KMS audit events into a single pane of glass.

---

## 🧱 1. Environment Setup
```bash
mkdir -p dashboard/{aws,azure,gcp}
cd dashboard
🟧 2. AWS Collector
bash
Copy code
aws logs create-export-task \
  --task-name "KMSMetricsExport" \
  --log-group-name "/aws/kms/events" \
  --destination "securethecloud-siem" \
  --from "$(date -d '-1 hour' +%s)000" \
  --to "$(date +%s)000"
🟦 3. Azure Collector
bash
Copy code
az monitor activity-log list --status Succeeded \
  --query "[].{time:eventTimestamp,operation:operationName}" -o json > azure_kms.json
🟨 4. GCP Collector
bash
Copy code
gcloud logging read 'resource.type="kms_key" AND severity>=NOTICE' \
  --format json > gcp_kms.json
☁️ 5. Merge and Normalize Logs
bash
Copy code
jq -s '[.[][]]' aws/*.json azure/*.json gcp/*.json > unified_kms_logs.json
⚙️ 6. Visualize with Grafana (or Power BI / Looker)
Import as datasource → create a panel with query:

sql
Copy code
SELECT cloud, key_id, count(*) as operations
FROM unified_kms_logs
GROUP BY cloud, key_id
Then color-code:

🟢 = healthy rotation

🟡 = approaching threshold

🔴 = overdue or key inactive

🧩 7. Automate Rotation Reminders
Add a Lambda/Logic App/Cloud Function trigger for stale keys:

bash
Copy code
# AWS Lambda trigger
aws kms list-keys | jq '.Keys[].KeyId' | while read key; do
  aws kms enable-key-rotation --key-id $key
done
📊 8. Validate
✅ All three clouds’ logs ingested
✅ Dashboard displays correlated key usage
✅ Alerts generated for violations or delays

🏁 You’ve built the final layer of governance visibility — a real-time, multi-cloud KMS dashboard.
