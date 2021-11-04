---
title: "Move azure resources between resource groups"
date: 2021-10-12T17:20:59+11:00
draft: false 
tags: ["Azure", "Azure Portal", "Resource Management"]
---

## Problem
While working with Azure cloud platform, often there will be instances where resources needs moving across resource groups for maintenance reasons or because of re-organising of products. There might even cases where the resource may need to be moved across subscriptions.

## Solution
In Azure resources can be moved across resource groups from the portal UI or Azure CLI or powershell or from the rest APIs. Moving the resource using the portal UI is as easy as going through a wizard like steps and clicking finish at the end of it. The process also validates if the resource can be moved or not, for example an Azure SQL Database cannot be moved without moving the SQL Server instance, and when a SQL Server instance is moved across, all the databases gets moved automatically.

From the UI, there are three ways of getting to the wizard.
1. Azure Resource Mover service
2. Move option at the top of the Resource Group Screen
3. Move option at the top of the Resource Screen.

All the above options lead to the same wizard screen when you can choose the destination subscription and resource group and initiate the move. This will validate if the resource support moving as well as if all the criteria is satisfied for the resource to be moved.

You can find all the information regarding what resources currently support moving [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/move-support-resources). Also the criteria that any resource that supports moving needs to satisfy to be able to move can be found [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/move-limitations/app-service-move-limitations)

The move does not copy over the role assignments to the destination resource, so all the role assignments will have to redone after the move is completed. There is no downtime to the usage of the resource, but the resource will be locked for few hours to any changes even though the move is completed in lesser time.

**Powershell** command [Move-AzResource](https://docs.microsoft.com/en-us/powershell/module/az.resources/move-azresource?view=azps-6.4.0) can be used to execute the move from Powershell

**Azure CLI** command [az resource move](https://docs.microsoft.com/en-us/cli/azure/resource?view=azure-cli-latest#az_resource_move) can be used to execute move from any CLI that has Azure CLI installed.

**Rest Endpoint**  POST https://management.azure.com/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version} [Details Here](https://docs.microsoft.com/en-us/rest/api/resources/resources/move-resources) can be used to trigger the source move passing the payload with resources ids and the target resource group names.

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

The Azure resource mover also bring in the capability of moving the resources across regions more about it can be found [here](https://docs.microsoft.com/en-us/azure/resource-mover/move-region-within-resource-group) 