---
title: "Cloud Resume Challenge - Azure Serverless "
date: 2021-06-28T10:18:48+11:00
draft: false
tags: ["developer", "blazor", "visual studio", "serverless", "azure", "azure functions", "c#", "dotnetcore"]
---

I recently came across the site https://cloudresumechallenge.dev/ and decided to give it a try using Azure services. To start simple I decided to ignore the DB, CDN part etc and just have the the UI and the middler layer of the app. Below is the high level architecture.

![Blog Arch ](/blogimages/ResumeApp_arch.png)

The front end of the app will be hosted a static web site in Azure Blob storage. Backend will be an Azure function that will feed the resume data to the frontend over HTTP, the azure function will be a HTTP triggered function. 
Currently the resume data in JSON format hardcoded in the Azure Function code. As an upgrade to the app, the JSON data can be moved to a CosmosDB instance and put an Azure CDN in front of the UI to deliver content fast to users.

## Development Environment
- **Frontend**: Blazor WebAssembly
- **Backend**: Azure Function using C# (Function Auth)
- **IDE**: VS Code
- **IAC**: Azure ARM Template
- **Source Control**: Github
- **CI/CD**: Github Actions

## The App project
The project is divided into 3 parts.  
**Core**: A dotnet standard 2.0 library project that contains all the entities used in the application. It also contains the interfaces and implementation of services, the classes that contains all the logic to get the data ready for the caller.
Created using ```dotnet new classlib --framework netstandard2.0 ```  
The one service class that currently resides in the core project is the ResumeService, which calls the Function backend and deserialized the JSON to its class instance and return. This service is injected in the frontend to complete the rendering the data.

**Frontend**: Blazor WebAssemly project, which contains all the UI rendering logic. The projects refers to the Core project to get the resume data.
Created using ``` dotnet new blazorwasm --framework net5.0```  
For now the function will respond to HTTP calls with a JSON representing the resume data. 

**Backend**: Azure Function project
Created using ```func init backend --dotnet```
```func new --name backend --template "HTTP trigger" --authlevel "function"```


## Infrastructure as code (IAC)
The infrastructure for deploying the application is coded using Azure Resource manager template for repeatable deployment, this makes it east to setup an azure environment quickly and without human errors. Also the infrastructure can be version controlled reviewed and provisioned as part of the CI/CD pipeline. 

referred the Azure ARM template quick start github (here)[https://github.com/Azure/AzureStack-QuickStart-Templates] to get started with the azure services.

The ARM template has the Azure Static website and Azure Function configured.

## CI/D
There are two github actions workflow, one is to deploy the infrastructure and another to deploy the code. The infrastructure is deployed manually as we don't want to spin up new infrastructure element automatically on any event. The
source code gets deployed on any commit on the ```master``` branch in the github repository.

### Deploying Infrastructure
To execute the workflow on Azure, we need to register github actions as an Azure AD and use the client credentials authentication flow to get access to create resources. Also, while creating a service principle for github actions, give permission on to one resource group to have maximum restrictions on. 

use the below command to create an Azure service principle and copy the returned JSON into Github secrets of your repo and call it AZURE_CREDENTIALS. substitute the subscription id, resource group name and the app name.

```az ad sp create-for-rbac --name ResumeAppGithub --role contributor --scopes /subscriptions/{subscriptionId}/resourceGroups/{MyResourceGroup} --sdk-auth```

In the YAML file to deploy the ARM template we use the *azure/arm-deploy@v1* which taken in subscriptionid and resourcegroupname as input, so store those too in the repo secret so that you don't have to expose the values into the code.

You can check the ARM template [here](https://github.com/gopkumr/Serverless-Resume/blob/28f4704aa0fd55ddd2c14395d2f5d73a2a1e134b/infra/resumeapp-infra.json) and the YAML file to deploy it [here](https://github.com/gopkumr/Serverless-Resume/blob/28f4704aa0fd55ddd2c14395d2f5d73a2a1e134b/.github/workflows/infra.yml)

### Deploying the backend azure function
Deploying the function app is easy as the github action can already login to Azure as an app. So all we are have to do is publish the app and upload it to the function service using the *Azure/functions-action@v1* action.

You can check the yaml file [here](https://github.com/gopkumr/Serverless-Resume/blob/2edc1f129863a8c4f2b77fe350fe801f3d0f41ca/.github/workflows/function.yml). 

### Deploying the frontend static web app
Deploying Blazor project follows two steps, one is to publish the project and next is to upload all the published files to the static web app. Both these tasks can be done by using the * Azure/static-web-apps-deploy@v1* action, which uses microsoft/oryx to build the Blazor project and copy over the content to the azure static app website.

You can check te yaml file [here](https://github.com/gopkumr/Serverless-Resume/blob/3a3fb515c5c1666cae89246c2b12c5a088e051cb/.github/workflows/staticwebapp.yml)

## Conclusion
The app currently implements just an azure function that returns a hardcoded JSON data that represents the resume and a Blazor app thats deployed on azure static web app to render the resume. 
The next revision of the app would be to add a Azure CDN service to cache the resume UI and also to return the content faster to whoever accesses it from where ever in the world. Azure function returns a hard coded JSON string which needs a change and have the resume stored as a document in the document in cosmos DB.

The full source code, IAC templates and the actions YAML files can be found in the [github repository](https://github.com/gopkumr/Serverless-Resume)