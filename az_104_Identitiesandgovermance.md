---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)

# Microsoft Certified: Azure Administrator Associate (AZ-104): Manage Azure Identities and Governance

## Azure CLI:
```
az group list
rgcli=$(az group list)
echo $rgcli
az group list --query "[].name"
az group list --query "[].name" --output tsv
rgcli=$(az group list --query "[].name" --output tsv)
az network vnet create \
 --name vnet-cli \
 --resource-group $rgcli \
 --address-prefix 10.0.0.0/16
```
## ARM Templates:
<img width="1382" height="744" alt="image" src="https://github.com/user-attachments/assets/7501d0ed-4272-4369-a597-ee4fa543edb4" />
- we can use custome deployment to deploy base on template
- during creation we can copy template or get it from resource group - > settings -> deployments -> template
<img width="1382" height="734" alt="image" src="https://github.com/user-attachments/assets/5201197d-365c-4be1-83d0-d4fac89e2491" />

<img width="1400" height="764" alt="image" src="https://github.com/user-attachments/assets/9f6ca151-bd5a-4d93-9a76-a4ee86391500" />
- biceps file are eay to read
