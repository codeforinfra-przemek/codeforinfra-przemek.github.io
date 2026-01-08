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

## How to use biceps:
<img width="1382" height="734" alt="image" src="https://github.com/user-attachments/assets/5201197d-365c-4be1-83d0-d4fac89e2491" />

<img width="1400" height="764" alt="image" src="https://github.com/user-attachments/assets/9f6ca151-bd5a-4d93-9a76-a4ee86391500" />
- biceps file are eay to read

example:
```
param location string = resourceGroup().location

var sku = 'S1'
var linuxFxVersion = 'php|7.4'

resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: 'AwesomeAppServicePlan'
  location: location
  sku: {
    name: sku
  }
  kind: 'linux'
  properties: {
    reserved: true
  }
}

resource webAppPortal 'Microsoft.Web/sites@2022-03-01' = {
  name: 'AwesomeApp'
  location: location
  kind: 'app'
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: linuxFxVersion
      ftpsState: 'FtpsOnly'
    }
    httpsOnly: true
  }
  identity: {
    type: 'SystemAssigned'
  }
}
``` from https://github.com/pluralsight-cloud/Microsoft-Certified-Azure-Administrator-Associate/blob/main/Azure%20Administration/Discovering%20Azure%20Bicep/deploy.bicep
How to use this:
```
 /home/cloud> code .
 /home/cloud> $rg = Get-AzResourceGroup
 /home/cloud> New-AzResourceGroupDeployment `
  -Name DemoDeployment1 `
  -ResourceGroupName $rg.ResourceGroupName `
  -TemplateFile deploy.bicep
```
and additionally:
```
az bicep --help
cat deploy.bicep
az bicep build --file deploy.bicep --outfile tst.json
az bicep decompile --file tst.json
rgName=$(az group list --query "[].name" -o tsv)
az deployment group create --name demoDeployment1 --resource-group $rgName --template-file tst.bicep 
az resource list --resource-group $rgName --output table 
```
## Subscription:

<img width="1406" height="674" alt="image" src="https://github.com/user-attachments/assets/2a8d317a-5b4b-49f9-b81c-d6a72f4929e1" />
<img width="1391" height="752" alt="image" src="https://github.com/user-attachments/assets/bc7a7258-e4d4-449c-b91b-bee50406428a" />
<img width="1250" height="771" alt="image" src="https://github.com/user-attachments/assets/3f970762-02d2-49bd-ad15-3fd48caeafcd" />

### Cost:
- Azure cost: https://azure.microsoft.com/en-us/pricing/calculator/?ef_id=_k_EAIaIQobChMI9_r2o7_7kQMVJKeDBx1NnTzMEAAYASAAEgI54vD_BwE_k_&OCID=AIDcmmeauvx05c_SEM__k_EAIaIQobChMI9_r2o7_7kQMVJKeDBx1NnTzMEAAYASAAEgI54vD_BwE_k_&gad_source=1&gad_campaignid=1635079395&gbraid=0AAAAADcJh_vgGnrfTzhsAn711Vdd9Eyla&gclid=EAIaIQobChMI9_r2o7_7kQMVJKeDBx1NnTzMEAAYASAAEgI54vD_BwE

- Cost Managment + Biling -> Create Budget or Cost Analyse

- Advisoe - we can setup recomendation every week for costs

### Tagging:
<img width="1382" height="758" alt="image" src="https://github.com/user-attachments/assets/bc93f012-1b7d-4d0d-a537-a4c5414a2875" />
- tag are no inherited
- we can multiple tags,

### Locking Resources
- Delete(can be modify by not deleted) or Read-only(only view)
- we can move resources between resources group (localization of resources is not tide to resource group, so we can in one resource grou east and west us)
- 

### Azure Policy
<img width="1365" height="796" alt="image" src="https://github.com/user-attachments/assets/f61a1f4c-62da-42c0-bf8a-38713266f263" />

### MGmt Groups
<img width="1344" height="775" alt="image" src="https://github.com/user-attachments/assets/4eaf8564-b52e-4f26-9a2e-95cb0d970e7d" />
<img width="1363" height="725" alt="image" src="https://github.com/user-attachments/assets/0a1b399f-4114-49e8-883c-2245e8df03d6" />



