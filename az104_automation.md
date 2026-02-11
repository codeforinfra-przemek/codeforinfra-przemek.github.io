---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)

# Users:
```
az version
az account show --output table
az account list --output table

az ad user list --query "[?contains(displayName, 'John')].{upn:userPrincipalName,name:displayName}" -o table
az ad user list --query "[0].userPrincipalName" -o tsv
cloud [ ~ ]$ UPN="az104.user1@YOURDOMAIN.onmicrosoft.com"
PW="P@ssw0rd-ChangeMe!234"

az ad user create \
  --display-name "AZ104 User1" \
  --user-principal-name "$UPN" \
  --password "$PW" \
  --force-change-password-next-sign-in true
bash: !234: event not found
Insufficient privileges to complete the operation.
az ad group create \
  --display-name "AZ104-Admins" \
  --mail-nickname "az104-admins"
az ad group show --group "AZ104-Admins" --query id -o tsv
GROUP_ID=$(az ad group show --group "AZ104-Admins" --query id -o tsv)
echo "$GROUP_ID"
az ad group delete --group "$GROUP_ID"
```


