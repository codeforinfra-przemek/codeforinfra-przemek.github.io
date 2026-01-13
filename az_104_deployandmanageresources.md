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




