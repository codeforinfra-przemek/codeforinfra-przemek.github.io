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
