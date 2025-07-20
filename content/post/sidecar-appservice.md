---
title: "Running Sidecar Containers in Azure App Service: An experiment with Dapr"
date: 2024-11-16T10:43:40+11:00
draft: false 
tags: ["Azure", "Azure App Services", "Docker", "Azure Container Registry", "Sidecar Pattern", "Azure Cache for Redis"]
---

### Running Sidecar Containers in Azure App Service: An experiment with Dapr 

Today I decided to try Azure App Service's new ability to run **sidecar containers**.  
My goal, To create a simple .NET API, run it in Azure App Service, and pair it with Dapr as a sidecar container and use it for state management. Locally, Dapr would connect to a local Redis docker container, and once deployed to Azure, it would seamlessly switch to Azure Cache for Redis.  

---

### Building the Main Application  

I started by creating a .NET 9 API to manage basic employee data. A straightforward app, but the twist was using Dapr for state management.  

The API had two endpoints: one to save a employee data to a state store and another to retrieve it by ID.  

Here's the core of what I built:

Create a new api project
```bash
dotnet new webapi -o api-app
```
Modify the program.cs to add the api endpoints
```csharp  
using Dapr.Client;
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);
var client = new DaprClientBuilder()
                .Build();
builder.Services.AddSingleton<DaprClient>(client);  

string DAPR_STORE_NAME = "statestore";

app.MapGet("/get-state/{id}", async ([FromServices]DaprClient client, string id) =>
{
     var result = await client.GetStateAsync<Employee>(DAPR_STORE_NAME, id);
    return result;
})
.WithName("GetStoredState");


app.MapPost("/set-state", async ([FromServices]DaprClient client, [FromBody]Employee value) =>
{
   await client.SaveStateAsync(DAPR_STORE_NAME, value.Id, value);
   return Results.Created();
})
.WithName("SetStoredState");

app.Run();  

record Employee(string Id, string Name, DateOnly? DOB);
```  

The real magic was in Dapr's state management.  

Locally, I used Redis container running in docker desktop as the state store. I created a simple dapr component YAML file to configure it:  

```yaml  
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: redis:6379
  - name: redisPassword
    value: ""
```  
and created a ```Dockerfile``` to config the dapr image

```yaml

FROM daprio/daprd:latest

EXPOSE 3500
EXPOSE 50001

ARG COMPONENT_PATH  #pass the path to the component yaml file as parameter while building the image
COPY ./${COMPONENT_PATH} /components

ENTRYPOINT ["./daprd"]

CMD ["run","--app-id", "api-app", "--resources-path", "/components", "--dapr-http-port", "3500", "--dapr-grpc-port", "50001"]
```

---

### Running Locally with Docker  

Docker Compose made orchestrating the container easy by running the Dapr and Redis containers together.  

My `docker-compose.yml` looked like this:  

```yaml  
services:  
  dapr-sidecar:  
    image: dapr-sidecar:local
    build:
      context: dapr-statestore
      dockerfile: ./Dockerfile
      args:
        COMPONENT_PATH: "components-local"
    ports:
      - "3500:3500"
      - "50001:50001"
  redis:  
    image: "redis:latest"  
    ports:  
      - "6379:6379"  
```  

Spinning it up was as easy as:  

```bash  
docker-compose up  
```  

Once everything was running, I tested the API locally using dotnet run. Saving and retrieving forecasts worked perfectly, and Dapr seamlessly handled the state management.  
Here are the http requests used to test the two api endpoints
```
### Testing Locally

POST http://localhost:5001/set-state HTTP/1.1
Content-Type: application/json

{
  "Id":"1",
  "Name":"User 1",
  "DOB":"2020-01-01"
}

###

GET http://localhost:5001/get-state/1
```


---

### Deploying to Azure App Service  

With local testing out of the way, it was time for the real challenge: deploying this setup to Azure App Service with the new sidecar feature.  

#### Preparing the Azure Environment  

First, I set up an Azure resource group, deploy and configure Azure Cache for Redis, deploy and configure container registry:  

```bash  
#Create resource group
az group create --name app-service-sidecar-poc-rg --location australiaeast

#Create Azure Cache for redis
az redis create --location australiaeast --name sidecar-poc-cache --resource-group app-service-sidecar-poc-rg --sku Basic --vm-size c0

#Enable EntraID Authentication in Azure Cache for redis
az redis update --name sidecar-poc-cache --resource-group app-service-sidecar-poc-rg  --set "redisConfiguration.aadEnabled"="true"

#Create Container Registry
az acr create --name appsrvdevacr --resource-group app-service-sidecar-poc-rg --sku Basic --admin-enabled true

```  

#### Containerizing and Pushing to Azure Container Registry  

I updated the Dapr component file to point to the Azure Cache for redis

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: sidecar-poc-cache.redis.cache.windows.net:6380
  - name: useEntraID #using apps managed identity to connect to cache
    value: "true"
  - name: enableTLS
    value: "true"
```

and containerized Dapr and pushed it to Azure Container Registry (ACR):  


```bash  
#Login to Azure Container Registry
az acr login --name appsrvdevacr  

#Build the image locally using docker, passing the dapr component config with the azure cache for redis configuration
docker build -t appsrvdevacr.azurecr.io/dapr-sidecar:v1 --build-arg COMPONENT_PATH=components-deploy ./dapr-statestore/
docker push appsrvdevacr.azurecr.io/dapr-sidecar:v1
```  

#### API App Deployment  

Deploying to Azure App Service was as simple as running:  

```bash  
#change to the api project folder
cd api-app
#create webapp, app servie plan and deploy the api
az webapp up --name sidecar-app-dev --os-type linux
#Enable managed identity
az webapp identity assign --name sidecar-app-dev --resource-group app-service-sidecar-poc-rg
```  
Next step is to give the app service permission to pull container from azure container registry and also to access azure cache for redis.

```bash
#Fetch app service identity id
principalId=$(az webapp show --resource-group app-service-sidecar-poc-rg --name sidecar-app-dev  --query "identity.principalId" --output tsv)

#Fetch the azure container registry id
registryId=$(az acr show --resource-group app-service-sidecar-poc-rg --name appsrvdevacr --query id --output tsv)

#Assign the app with AcrPull permission
az role assignment create --assignee $principalId --scope $registryId --role "AcrPull"

#Assign the app data owner permission to the cache
az redis access-policy-assignment create -g app-service-sidecar-poc-rg -n sidecar-poc-cache --object-id $principalId --object-id-alias 'dapr-sidecar' --access-policy-name "Data Owner" --policy-assignment-name "dapr-sidecar-data-owner"
```

#### Sidecar Container Deployment  

Now that the app is deployed and permissions are granted, the final step is to deploy the dapr sidecar container to app service.  

One way to deploy the sidecar container is via a bicep template. 

```bicep
param appServiceName string = 'sidecar-app-dev'
param sidecarName string = 'dapr-sidecar'
param sidecarImage string = 'appsrvdevacr.azurecr.io/dapr-sidecar:v1'


resource sidecar_app_dev_dapr_sidecar 'Microsoft.Web/sites/sitecontainers@2024-04-01' = {
  name: '${appServiceName}/${sidecarName}'
  properties: {
    image: sidecarImage
    isMain: false
    authType: 'SystemIdentity'
    volumeMounts: []
    environmentVariables: []
  }
}
```

Running the command deploys the bicep and deploys the sidecar container to app service
```bash
#Change to the root folder
cd..
#Deploy the bicep
az deployment group create --resource-group app-service-sidecar-poc-rg --template-file ./bicep/sidecar-container.bicep
```

---

### The Moment of Truth  

After deployment, wait for few seconds for the containers to spin-up to test.  

Use the below test file endpoints
```
### Test the deployed code

POST https://sidecar-app-dev.azurewebsites.net/set-state HTTP/1.1
Content-Type: application/json

{
  "Id":"1",
  "Name":"User 1",
  "DOB":"2020-01-01"
}

###

GET https://sidecar-app-dev.azurewebsites.net/get-state/1

```  

Both worked perfectly. Locally, Dapr connected to Redis, and in deployed environment, it switched to Azure Cache for Redis without any hiccups.  

---

### Final Thoughts  

The simplicity of managing sidecars, coupled with the power of tools like Dapr, makes this a game-changer for developers building modern cloud-native applications.
All the code artifacts and CLI commands are available on my github repo: [gopkumr/app-service-sidecar](https://github.com/gopkumr/app-service-sidecar)

References

- [Tutorial: Configure a sidecar for a custom container app – Azure App Service](https://learn.microsoft.com/en-us/azure/app-service/tutorial-custom-container-sidecar)
- [Sidecar pattern – Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/patterns/sidecar)
- [Introducing Sidecars for Azure App Service for Linux: Now Generally Available](https://azure.github.io/AppService/2024/11/08/Global-Availability-Sidecars.html)

   

