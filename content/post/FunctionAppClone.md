---
title: "Cloning a Azure Function App"
date: 2022-02-10T14:22:48+11:00
draft: false
tags: ["Azure", ".Net", "Azure Function", "Azure Portal", "Azure CLI"]
---

Recently I had a requirement to make a copy of a Function App from the production version to support a POC implementation of an solution upgrade.   

One option was to deploy the Release branch which had the version same as in PROD (we already made updates to that function app post release, so DEV was already a lot of commits ahead). The challenge with this approach was, since we did not had a hotfix release, there were no Pipelines setup for Release branch. So we had to setup a pipeline, give the pipeline service account access to the POC resource group, then actually triggering the deployment.

But we ended up doing option two which was easy and straigh forwards done in two steps.
> Download the function app as a zip file using the *download app content* option in the function app overview portal
  ![Download Function Content](/blogimages/FunctionAppDownload.png)
> Use Azure CLI to deploy the function app using zip deploy, the zip file downloaded in the above step can be directly deployed
```
az functionapp deployment source config-zip \
    -g <RESOURCEGROUPNAME> -n <TARGET-FUNCTIONAPPNAME> \
    --src <FULL PATH TO THE ZIP FILE>
```

Tadaa! you have now made a clone of your existing Azure function app without needing to build a pipeline or deploying from Visual studio.

This is pretty simple activity, I am documenting for anyone new to Azure searching for such an requirement.
