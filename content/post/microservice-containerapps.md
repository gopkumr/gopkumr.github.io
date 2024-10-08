---
title: "Simplified Microservice Deployment with Azure Container Apps and Dapr"
date: 2024-10-07T21:45:08+10:00
draft: false 
slug: "Microservice Deployment with Azure Container Apps "
tags: ["Azure", "Microservices", "Azure Container Apps", "Dapr", "Security", "Scalability", "Containers", "AKS", "Kubernetes"]
---

### Introduction

In this blog post we look into a scalable and flexible platform to run microservices on Azure without the complexity of managing infrastructure. **Azure Container Apps** allows you to run containerized microservices and integrating **Dapr (Distributed Application Runtime)** can simplify the communication between services, manage state, and handle pub/sub messaging. This blog also shows how to set up Azure Container Apps, and how to deploy Dapr-enabled microservices that communicate with each other.

### What is Dapr?

Dapr is an open-source, portable runtime that simplifies building microservices by providing building blocks for common microservice scenarios, such as service-to-service invocation, pub/sub messaging, state management, and distributed tracing. Dapr is deployed as a separate process or container to provide isolation and encapsulation and this pattern is known as [sidecar pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/side-car). By combining **Azure Container Apps** with **Dapr**, you get the power of Kubernetes without its complexities.

### Key Benefits of Azure Container Apps + Dapr

- **Simplified Microservice Communication**: Dapr provides built-in service discovery and service-to-service invocation.
- **State Management**: Dapr allows you to manage state across distributed services easily.
- **Pub/Sub Messaging**: Dapr enables pub/sub messaging between microservices with a wide variety of supported message brokers.
- **Serverless Autoscaling**: Azure Container Apps automatically scales your microservices based on load, and Dapr enables handling of event-driven workloads with ease.
- **Observability**: Both Azure Container Apps and Dapr provide built-in observability tools for monitoring and tracing requests across services.

AKS offers more control over the underlying infrastructure and is ideal for more complex Kubernetes workloads. However, if simplicity without sacrificing flexibility is the requirement, then Azure Container Apps and Dapr is a great fit.

### Deploying Dapr-enabled .NET Microservices on Azure Container Apps

Below we are deploying two microservices using **Azure Container Apps** and **Dapr**. One microservice will call another service privately using Dapr's service-to-service invocation feature.

Components we are implementing
![Container App Solution](/blogimages/container-app.svg)

#### Step 1: Prerequisites

- [**Azure CLI**](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- [**.NET SDK**](https://dotnet.microsoft.com/download)
- **Azure Subscription** with sufficient permission
- [**Docker**](https://docs.docker.com/get-docker/) to run locally
- [**Dapr CLI**](https://docs.dapr.io/getting-started/install-dapr-cli/) to run locally.

#### Step 2: Create .NET Web APIs

We will create two simple APIs: **Products Service** and **Orders Service**. The **Orders Service** will invoke the **Products Service** using Dapr's service invocation.

1. **Create the Products Service**:
   ```bash
   dotnet new webapi -n ProductsService
   cd ProductsService
   ```

   Modify `program.cs` to add a product minimal API endpoint:
   ```csharp
      app.MapGet("/products/{id}", (int id) =>
      {
         return new { Id = id, Name = $"Product {id}", Price = id * 10 };
      })
      .WithName("GetProduct")
      .WithOpenApi();
   ```
 
2. **Create the Orders Service**:
   ```bash
   dotnet new webapi -n OrdersService
   cd OrdersService
   ```
   Install Dapr SDK packages to support invoking product service via Dapr Service invocation
   ```bash
    dotnet add package Dapr.AspNetCore
   ```

   Modify `program.cs`
   - Add Dapr Client to Dependency scope
   ```csharp
   using Dapr.Client;

   builder.Services.AddDaprClient();
   ```
   - Add an order minimal api endpoint that invokes the **Products Service** using Dapr's service invocation:
   ```csharp
      app.MapGet("/orders/{id}", async (int id, [FromServices]DaprClient client) =>
      {
         var product = await client.InvokeMethodAsync<object>(HttpMethod.Get,"products-service", $"products/{id}");
         return new { OrderId = id, Product = product };
      })
      .WithName("GetOrder")
      .WithOpenApi();
   ```

   Note the parameters passed for invoking Dapr: 
    - `HttpMethod.Get`: Http method implemented in Product API in Step 1.  
    - `{app-id}`: The `app-id` of the product service. Set as `products-service` later in the post.
    - `products/{id}`: API Path to get a product as implemented in Step 1.

#### Step 3: Dockerize Both APIs

1. **Create an Azure Container Registry**:
   ```bash
   az acr create --resource-group <your-resource-group> --name <your-registry-name> --sku Basic
   ```
2. **Push Docker Images to Azure Container Registry (ACR)**:

   Starting dotnet 7 `dotnet publish` command can publish projects directly to docker container registry. By default it pushes to the local docker daemon. 
   We will use it to push to the  Azure Container Registry created in Step 1.
   - Login to Azure Container Registry:
   ```bash
    az acr login --name <your-registry-name>
   ```
   - Publish and push the **Products Service**:
   ```bash
    dotnet publish -c Release -r linux-x64 
    -p:PublishProfile=DefaultContainer \
    -p:ContainerRepository=productsservice \
    -p:ContainerImageTag=v1 --no-self-contained \
    -p:ContainerRegistry=<your-registry-name>.azurecr.io \
   ```

   - Publish and push the **Orders Service**:
   ```bash
    dotnet publish -c Release -r linux-x64 
    -p:PublishProfile=DefaultContainer \
    -p:ContainerRepository=orderssservice \
    -p:ContainerImageTag=v1 --no-self-contained \
    -p:ContainerRegistry=<your-registry-name>.azurecr.io \
   ```

#### Step 4: Set Up Azure Container Apps with Dapr enabled

1. **Create an Azure Container Apps Environment**:
   ```bash
   az containerapp env create --name my-environment --resource-group <your-resource-group> --location <your-location>
   ```
2. **Setting up a service principle**:
   For the Azure Container App to pull down an image from the Container registry, a service principle with permission `acrpull` is required. 
    ```bash
    ACR_REGISTRY_ID=$(az acr show --name <your-registry-name> --query "id" --output tsv)
    PASSWORD=$(az ad sp create-for-rbac --name <service-principle-name> --scopes $ACR_REGISTRY_ID --role acrpull --query "password" --output tsv)
    USER_NAME=$(az ad sp list --display-name <service-principle-name> --query "[].appId" --output tsv)
    echo "Service principal ID: $USER_NAME"
    echo "Service principal password: $PASSWORD"
    ```

3. **Deploy Products Service with Dapr**:
   ```bash
   az containerapp create --name products-service \
     --resource-group <your-resource-group> \
     --environment my-environment \
     --image <your-registry-name>.azurecr.io/productsservice:v1 \
     --registry-username <service-principal-ID> \
     --registry-password <service-principal-password> \
     --target-port 8080 \
     --ingress 'internal' \
     --registry-server <your-registry-name>.azurecr.io \
     --min-replicas 1 --max-replicas 3 \
     --enable-dapr true --dapr-app-id products-service --dapr-app-port 8080
   ```
   [Read more](https://learn.microsoft.com/en-us/cli/azure/containerapp?view=azure-cli-latest#az-containerapp-create) for other options.

4. **Deploy Orders Service with Dapr**:
   ```bash
   az containerapp create --name orders-service \
     --resource-group <your-resource-group> \
     --environment my-environment \
     --image <your-registry-name>.azurecr.io/ordersservice:v1 \
     --registry-username <service-principal-ID> \
     --registry-password <service-principal-password> \
     --target-port 8080 \
     --ingress 'external' \
     --registry-server <your-registry-name>.azurecr.io \
     --min-replicas 1 --max-replicas 3 \
     --enable-dapr true --dapr-app-id orders-service --dapr-app-port 8080
   ```
   [Read more](https://learn.microsoft.com/en-us/cli/azure/containerapp?view=azure-cli-latest#az-containerapp-create) for other options.

Here, both services are Dapr enabled, and the **Orders Service** invokes the **Products Service** privately using Dapr.
> *If you receive errors about missing parameters when you run az containerapp commands in Azure CLI run the below command to install the extension*
> ```bash
>   az extension add --name containerapp --upgrade
> ```

#### Step 5: Test the Order Service

Retrieve the external URL for the **Orders Service**:
```bash
az containerapp show --name orders-service --resource-group <your-resource-group> --query properties.configuration.ingress.fqdn
```

Call the Orders API:


```bash
curl http://<orders-service-url>/orders/1
```

The **Orders Service** will use Dapr to call the **Products Service** and return the order details, including the product information.

### Conclusion

Combining Azure Container Apps with Dapr provides a simplified yet powerful way to deploy and manage microservices. Dapr adds capabilities such as service discovery, pub/sub messaging, and state management while Azure Container Apps takes care of autoscaling and infrastructure management. This integration reduces complexity while enhancing functionality, making it ideal for microservice and event-driven architectures. AKS gives more fine-grained infrastructure control. However, for most cases, Azure Container Apps with Dapr offers a scalable, cost-effective, and simple solution for deploying microservices in Azure.

