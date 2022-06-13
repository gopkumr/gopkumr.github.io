---
title: "Exposing Azure Storage container via SFTP"
date: 2022-06-13T12:05:53+10:00
draft: false 
tags: ["Azure", "Storage Account", "Blob Storage", "SFTP"]
---
## Context
While most of the application integration patterns are moving towards real-time, near-real-time and stream based solutions, there are still requirement to have batch file based data movement. These requirements are often for reporting or data warehousing scenario or while integrating with a legacy system.
While there are many products that help setup SFTP server, Azure was missing a SaaS offering for hosted SFTP server, like Amazon's AWS Transfer on top of S3.
To host SFTP in Azure the customer has to setup their own SFTP workload either using a VM hosting an SFTP server and mounting the blob storage as a VM disk or hosting the SFTP server as a container on services like ACI and mounting the storage account.
While hosting an SFTP VM is not a complex task, but it adds to the organization's maintenance list, to keep it up and running, securing it and patching updates while maintaining uptime.

## Solution
Microsoft has recently introduced the SFTP service on top of the Azure Blob storage account, which brings in the SFTP SaaS capability to Azure. The feature is in pubic preview during time of writing.
The solution brings in the capability to expose a blob container via SFTP, authenticating users using a password or a SSH key. The users of the SFTP service is setup via the LocalUsers option in SFTP and NOT via Azure Identity management. The storage for the SFTP service is hosted on a container in the storage account. The SFTP service also give options to have user permission mapped for the container, which means when you grant local user access to SFTP container, you can specify permission like READ, WRITE, DELETE, LIST etc.

## Implementing SFTP
Implementing SFTP is straightforward and easy.
**Prerequisite**
- General purpose V2 Storage account
- Hierarchical namespace enabled, i.e. Azure Datalake Storage Gen2 enabled storage account.

**Steps**
- Navigate to the storage account and choose SFTP from the left navigation
- Choose the Enable SFTP option

![Enable SFTP](/blogimages/SFTPEnable.png)

- Once you enable the SFTP option, you can start adding LocalUsers.
- You can have users enabled for password based authentication or key pair.
![Add SFTP User](/blogimages/SFTPAddUser.png)
- The new keypair can be generated within Azure using SSH Key resource or you can import the public key that might have been provided by a third party .
- You can choose the container that would host the SFTP for the user along with the kind of permission that is being granted to the user.
![SFTP User Permission](/blogimages/SFTPUserPermission.png)
- You can also optionally choose a container as a home directory for the user, which is the default directory where the user will land if the container is not specified in the connection (more details below)
- If you have generated the SSH Key in Azure, you will be able to download the private key that can be used to connect to the SFTP.

**Connecting to SFTP**
The host name of the SFTP server would be the storage account blob endpoint `[STORAGEACCOUNT].blob.core.windows.net`. 
The user name to be used depends on which container you want to access.
If the user wants to access the SFTP container that was setup, then user would have to use `[STORAGEACCOUNT].[CONTAINER_NAME].[USERNAME]` as the user name.
If the user ignores the CONTAINER_NAME in the user name like this `[STORAGEACCOUNT].[USERNAME]`, the SFTP connects to the users home directory that was setup during user creation. If the a home directory was not setup, then user name without a container name will be rejected as unauthorized.

e.g Connecting to SFTP Container via powershell
```
sftp sasftppoc.inbound.sftpuser@sasftppoc.blob.core.windows.net
```
e.g Connecting to users home directory via powershell
```
sftp sasftppoc.sftpuser@sasftppoc.blob.core.windows.net
```
