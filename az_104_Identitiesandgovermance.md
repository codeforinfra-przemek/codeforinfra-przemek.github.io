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
