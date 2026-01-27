---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

[← Back to index](/)

# Microsoft Certified: Azure Administrator Associate (AZ-104): Monitor and Maintain Azure Resources

## Monitoring and Alerting

<img width="1383" height="776" alt="image" src="https://github.com/user-attachments/assets/6de5a8e8-3e79-4269-9043-b67cd22b7402" />
<img width="1397" height="794" alt="image" src="https://github.com/user-attachments/assets/4442357c-65c0-4a79-ad11-bffc29b9eb0c" />
<img width="1409" height="819" alt="image" src="https://github.com/user-attachments/assets/775dfc5e-199c-41e8-ba76-126798e99557" />
<img width="1398" height="821" alt="image" src="https://github.com/user-attachments/assets/982cc1a1-2240-4dde-a60c-e637c4b3b8a7" />
<img width="685" height="368" alt="image" src="https://github.com/user-attachments/assets/225890e0-a283-4a65-baa1-e82a1c274b41" />
<img width="704" height="415" alt="image" src="https://github.com/user-attachments/assets/401f7d00-341b-4037-a08a-0bfcc80c7482" />
```
rsg=897-b2b74174-configure-an-azure-monitor-alert-rule
rule_scope=/subscriptions/28e1e42a-4438-4c30-9a5f-7d7b488fd883/resourceGroups/897-b2b74174-configure-an-azure-monitor-alert-rule/providers/Microsoft.Storage/storageAccounts/pslabcet3g
action_group_name='mygroupename1'
az monitor action-group create \
 --name $action_group_name \
 --short-name $action_group_name \
 --resource-group $rsg \
 --action sms prz 1 555555555
az monitor activity-log alert create \
  --name "myalertrule1" \
  --resource-group $resource_group_name \
  --scope $rule_scope \
  --condition "category=Administrative and operationName=Microsoft.Storage/storageAccounts/write" \
  --description "my description" \
  --action-group $action_group_name

az monitor activity-log alert list
```
<img width="1394" height="800" alt="image" src="https://github.com/user-attachments/assets/c276956a-3b7a-4a7d-b369-7916dd0ed9e1" />
<img width="1406" height="793" alt="image" src="https://github.com/user-attachments/assets/fb57d8e5-5ba2-4f9c-852f-65ed53cbea8d" />
<img width="1396" height="805" alt="image" src="https://github.com/user-attachments/assets/c58b1b7f-3d9a-48c4-bf0e-427336d8e939" />
<img width="1388" height="787" alt="image" src="https://github.com/user-attachments/assets/f2bd5f6a-79c5-4442-b513-9a62ed06a0a5" />
<img width="1363" height="773" alt="image" src="https://github.com/user-attachments/assets/b80ada32-1476-4679-9a6c-7548994da6e1" />
<img width="1381" height="817" alt="image" src="https://github.com/user-attachments/assets/31173766-e555-4b40-a7f3-e7d88d1afc42" />
<img width="1391" height="824" alt="image" src="https://github.com/user-attachments/assets/6e7a3a2d-881b-4f08-8273-ed8b74bbe9cb" />
<img width="1382" height="809" alt="image" src="https://github.com/user-attachments/assets/a4375ff1-162d-402a-a973-9d9fcbcc9a91" />
<img width="1372" height="780" alt="image" src="https://github.com/user-attachments/assets/1b587dae-0712-483c-b1bb-3809aae3a852" />
<img width="1288" height="641" alt="image" src="https://github.com/user-attachments/assets/dd416af1-7d6b-42b3-8616-59ee8de4d462" />
