---
title: "Deploying 'JUST' the modified ARM templates"
date: 2021-09-05T10:00:46+10:00
draft: false 
tags: ["Azure", "AzureDevOps", "ARM Template"]
---
## Problem
The project has a bunch of ARM templates as part of IAC scripts and more often only couple, if not few templates get modified. But when deploying using Azure pipeline all the templates gets deployed. Even though ARM template deployment support incremental mode, if a templates is deployed with exact same properties, the resource gets recreated. The project does not want to recreate all the templates when only a few are changed. Currently there is no out-of-the-box tasks that support this behavior (*or I could not find any*). [Deployment Mode Reference](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes)

## Approach
The lack of out-of-the-box capability to deploy modified ARM templates, force us to write scripts to implement the logic. The approach is to create a YML build pipeline and use git cli to get the modified files in the latest commit and use powershell script to copy the modified files and its related files into the staging directory drop location. The release pipeline would look at the drop location and use Azure CLI to deploy the templates and its parameter files to azure.    

## Implementation

### Build pipeline
Build is a YML based pipeline with Powershell task with inline script.  Below is the script that copies the files to a staging directory
```
 $modifiedFiles=$(git diff --name-only HEAD HEAD~1 -- '***.json')
 foreach ($modifiedFile in $modifiedFiles)       
   {
    $directory = (Get-Item $modifiedFile).Directory
    Copy-Item -Path $directory -Destination $(Build.ArtifactStagingDirectory) -Recurse
   }
```
The script copies the files to the staging directory. The next task can be `PublishBuildArtifact` task to move the files to the drop location.

### Release pipeline
After the build pipeline is executed the files from the latest commit is available in the drop location. In the release pipeline, use an azure CLI task to get the files and deploy the templates into resource group using `az deployment group` command.

```
$editedFiles = Get-ChildItem $(System.DefaultWorkingDirectory)"\$($dropLocation)\" -directory | Select Name;

$editedFiles | ForEach-Object {
      $TemplatePath="$(System.DefaultWorkingDirectory)\$($dropLocation)\$($_.Name)/$($_.Name).json"
      $TemplateParamPath="$(System.DefaultWorkingDirectory)\$($dropLocation)\$($_.Name)\$($_.Name)-parameters.json"

      if(Test-Path $TemplatePath)
        {
           az deployment group create --resource-group $resourceGroupName  --template-file $TemplatePath  --parameters $TemplateParamPath
        }
      }
```
In this script I have resource group name as a variable, so that the same script can be used to deploy to any resource group.

We use Azure CLI task specifically to use the service connection that is configured in Azure devops to connect with Azure platform.

## Conclusion
The above approach satisfies the requirement to deploy only the modified using a custom written script. If there is a better approach please don't hesitate to correct me.
