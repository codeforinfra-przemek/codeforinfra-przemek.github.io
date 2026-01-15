# Microsoft Certified: Azure Administrator Associate (AZ-104): Deploy and Manage Azure Compute Resources

## Introduction to Azure Virtual Machines

<img width="1391" height="796" alt="image" src="https://github.com/user-attachments/assets/32372477-eeec-4e27-9638-d6199a94c196" />
<img width="1391" height="773" alt="image" src="https://github.com/user-attachments/assets/92ebe093-2dfa-44ee-855a-a55d961d68ba" />
<img width="1380" height="760" alt="image" src="https://github.com/user-attachments/assets/897ba6b1-3165-42e7-a288-8b22dae658aa" />

`az vm create --help`
```
az vm create `
  --name vm-prod-01 `
  --resource-group rg-az104 `
  --image Ubuntu2204 `
  --security-type TrustedLaunch `
  --generate-ssh-keys
```
<img width="1387" height="781" alt="image" src="https://github.com/user-attachments/assets/0d13d4f0-efef-4c7d-b7a3-04825f6185d2" />
<img width="1388" height="799" alt="image" src="https://github.com/user-attachments/assets/07a57c21-6168-4f54-a9e3-254bb253f4bd" />
```
$rg = "rg-az104-02"
$location = "eastus"

New-AzVm -ResourceGroupName $rg -Name "vm-prod-01" -Location $location -Image Canonical:UbuntuServer:18.04-LTS:latest -Size Standard_D2s_V3
Get-AzVMDiskEncryptionStatus -ResourceGroupName $rg -VMName "vm-prod-01"
New-AzKeyVault -Name 'psaz104keyvault' -ResourceGroupName $rg -Location $location -EnabledForDiskEncryption
$keyvault = Get-AzKeyVault -VaultName 'psaz104keyvault' -ResourceGroupName $rg
Set-AzVMDiskEncryptionExtension -ResourceGroupName $rg -VMName "vm-prod-01" -DiskEncryptionKeyVaultUrl $keyvault.VaultUri -DiskEncryptionKeyVaultId $keyvault.ResourceId -SkipVmBackup -VolumeType All
Y
```
<img width="1384" height="794" alt="image" src="https://github.com/user-attachments/assets/1b10c5f6-f393-4122-8532-c414c279cdaa" />
<img width="1385" height="769" alt="image" src="https://github.com/user-attachments/assets/3fb11f67-95f7-422e-b4be-2a26143f3c5d" />
<img width="1360" height="743" alt="image" src="https://github.com/user-attachments/assets/3b2c018d-23c2-4eb5-98ad-610f0a4ce8ae" />
<img width="1270" height="701" alt="image" src="https://github.com/user-attachments/assets/b92cdd8b-a041-4439-b3e2-14865ce498cd" />
<img width="1394" height="782" alt="image" src="https://github.com/user-attachments/assets/07a4bcb8-09ac-4014-9c5a-71a40e509a80" />

## Creating App Service Plan

<img width="1391" height="804" alt="image" src="https://github.com/user-attachments/assets/f32e79ed-a567-4ad9-95f5-96c78cfb1b8f" />
<img width="1397" height="775" alt="image" src="https://github.com/user-attachments/assets/ff155702-b7e5-4ff2-9cb8-dee28e4d3b07" />
<img width="1399" height="773" alt="image" src="https://github.com/user-attachments/assets/b0a214e0-5b2d-4391-bb0b-d3293f412f8b" />

## Introduction to Azure Containers

```
$acrName = az acr list -o tsv --query '[].name'
az acr build `
  --image sample/hostnameapp:v1 `
  --registry $acrName `
  --file ./Dockerfile .
az acr run `
  --registry $acrName `
  --cmd '$Registry/sample/hostnameapp:v1' /dev/null

````
<img width="1371" height="766" alt="image" src="https://github.com/user-attachments/assets/3690bd3b-5ced-494a-82de-36e619c005bf" />

### Build and Run a Container Using Azure ACR Tasks

```
# az acr create --resource-group 425-b0a83ada-build-and-run-a-container-using-azure \
#   --name acrbuildcontainer11 --sku Basic --admin-enabled true
sub=$(az account show --query id -o tsv)
rg=$(az group list --query '[0].name' -o tsv)
acrName="acrbuildcontainer11"
location=$(az group show --subscription "$sub" -n "$rg" --query location -o tsv)
az acr create \
  --subscription "$sub" \
  --resource-group "$rg" \
  --location "$location" \
  --name "$acrName" \
  --sku Basic \
  --admin-enabled true

cloud [ ~ ]$ cd $HOME
cloud [ ~ ]$ cd clouddrive
cloud [ ~/clouddrive ]$ echo "FROM hello-world" > Dockerfile
# az acr build --image sample/hello-world:v1 --registry acrbuildcontainer11 \
#  --file Dockerfile .
az acr build \
  --subscription "$sub" \
  --registry "$acrName" \
  --image sample/hello-world:v1 \
  --file Dockerfile \
  .
az acr run \
  --subscription "$sub" \
  --registry "$acrName" \
  --cmd '$Registry/sample/hello-world:v1' \
  /dev/null
```
### Create Web App from Docker Container in Azure

```
rg=$(az group list --query '[0].name' -o tsv)
acr="acrbuildcontainer11"
sub=$(az account show --query id -o tsv)

az acr create \
  --subscription "$sub" \
  --resource-group "$rg" \
  --location "$location" \
  --name "$acr" \
  --sku Basic \
  --admin-enabled true

cd $HOME
cd clouddrive  # this way we enter fileshare we created to work on this.
git clone --branch js-docker https://github.com/linuxacademy/content-AZ-104-Microsoft-Azure-Administrator.git ./js-docker
cd js-docker/
az acr build --image js-docker:v1 --registry $acr --file Dockerfile .
#there was problem with dockerfile so we change it to:
FROM node:18-alpine

WORKDIR /src
COPY . .
RUN npm install

EXPOSE 8080
CMD ["node", "./app.js"]
az acr run \
  --subscription "$sub" \
  --registry "$acr" \
  --cmd '$Registry/sample/hello-world:v1' \
  /dev/null

```
### Using Container App:

<img width="1378" height="768" alt="image" src="https://github.com/user-attachments/assets/e15bb518-e16e-442d-b11d-68d21cd400ed" />

