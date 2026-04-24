---
title: Homarr - Proxmox intergration
description:
draft: false
tags:
  - proxmox
  - VM
created: 25/18/2025 08:18
updated: 23/07/2026 21:07
---
# 1. Create Proxmox group:
Navigate to : `Datacentre -> Permissions -> Groups`:
![[homarr - proxmox intergration group.png]]

# 2. Create Group Permission:
Navigate to : `Datacentre -> Permissions -> Add -> Group permissions`:
![[homarr - proxmox group permissions.png]]

# 3. Create User:
navigate to : `Datacentre -> Permissions -> Users`
![[homarr - proxmox intergration user.png]]

# 4. Create API Token:
navigate to : `Datacentre -> Permissions -> API Tokens`
![[homarr - proxmox intergration token.png]]

Take note of:
token ID: `homarr_User@pve!homarr_token`
Secret: `12345678-aaaa-bbbb-cccc-1234567890AB`

>[!warning] 
>Token is only shown onces. after you close it you won't be able to see it again.

# 5. Create Token Permissions:
Navigate to : `Datacentre -> Permissions -> Add -> API Tokens Permissions`
![[homarr - intergration token perm.png]]

# 6. Integrate with Homarr:
In the Homarr webpage, go to:  `Profile(icon at top right) -> management -> Intergrations -> New Intergrations -> Proxmox`

based on the above token ID: `homarr_User@pve!homarr_token`
fill in as follow:
![[homarr - intergration homarr.png]]

# 7. Dealing with CA certificate extraction failed:
Head **back to proxmox**. navigate to `Datacenter -> pve -> System -> Certificates -> View Certificate`:

![[homarr - proxmox intergration cert.png]]
**Click on view raw and cop the data into a file.**
NOTE: Ensure you choose the one that says **`CA`**.
# 8. Upload the cert to homarr:
in homa, go to: `Profile(icon at top right) -> management -> Tools -> Certificates -> Add certificate`

![[homarr - intergration proxmox upload.png]]

# 9. Retry Test intergradation.
Finally retry the test integration in step 6.

Lastly to use the **in homerr**:
in edit mode: `<<plus_symbol>> -> New Item -> System Health Monintoring`.
Click on the 3 dots and choose `edit item -> proxmox`. save changes and done.
 

# Resource: 
https://homarr.dev/docs/integrations/proxmox/