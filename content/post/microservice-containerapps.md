---
title: "Simplified Microservice Deployment with Azure Container Apps and Dapr"
date: 2024-10-07T21:45:08+10:00
draft: false 
slug: "Microservice Deployment with Azure Container Apps "
tags: ["Azure", "Microservices", "Azure Container Apps", "Dapr", "Security", "Scalability", "Containers", "AKS", "Kubernetes"]
---

## Introduction

In this blog post we look into a scalable and flexible platforms to run microservices on Azure without the complexity of managing infrastructure. **Azure Container Apps** allows you to run containerized microservices and integrating **Dapr (Distributed Application Runtime)** can simplify the communication between services, manage state, and handle pub/sub messaging. This blog also shows how to set up Azure Container Apps, and how to deploy two Dapr-enabled microservices that communicate with each other using .NET.

### What is Dapr?

Dapr is an open-source, portable runtime that simplifies building microservices by providing building blocks for common microservice scenarios, such as service-to-service invocation, pub/sub messaging, state management, and distributed tracing. By combining **Azure Container Apps** with **Dapr**, you get the power of Kubernetes without its complexities.

### Key Benefits of Azure Container Apps + Dapr

- **Simplified Microservice Communication**: Dapr provides built-in service discovery and service-to-service invocation.
- **State Management**: Dapr allows you to manage state across distributed services easily.
- **Pub/Sub Messaging**: Dapr enables pub/sub messaging between microservices with a wide variety of supported message brokers.
- **Serverless Autoscaling**: Azure Container Apps automatically scales your microservices based on load, and Dapr enables handling of event-driven workloads with ease.
- **Observability**: Both Azure Container Apps and Dapr provide built-in observability tools for monitoring and tracing requests across services.

AKS offers more control over the underlying infrastructure and is ideal for more complex Kubernetes workloads. However, if simplicity without sacrificing flexibility is the requirement, then Azure Container Apps and Dapr is a great fit.

### Deploying Two Dapr-enabled .NET Microservices on Azure Container Apps

Below we are deploying two microservices using **Azure Container Apps** and **Dapr**. One microservice will call another service privately using Dapr's service-to-service invocation feature.

Components that we are implementing
![Container App Solution](/blogimages/container-app.svg)

#### Step 1: Prerequisites

- [**Azure CLI**](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- [**Dapr CLI**](https://docs.dapr.io/getting-started/install-dapr-cli/)
- [**Docker**](https://docs.docker.com/get-docker/)
- [**.NET SDK**](https://dotnet.microsoft.com/download)

#### Step 2: Create Two .NET Microservices with Dapr

We will create two simple services: **Products Service** and **Orders Service**. The **Orders Service** will invoke the **Products Service** using Dapr's service invocation.

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

   Modify `program.cs` to add an order minimal api endpoint that invokes the **Products Service** using Dapr's service invocation:
   ```csharp
      app.MapGet("/orders/{id}", async (int id, IHttpClientFactory httpClientFactory) =>
      {
         var httpClient = httpClientFactory.CreateClient();
         var product = await httpClient.GetStringAsync($"http://localhost:3500/v1.0/invoke/products-service/method/products/{id}");
         return new { OrderId = id, Product = product };
      })
      .WithName("GetOrder")
      .WithOpenApi();
   ```

   Note the URL format for invoking Dapr: `http://localhost:3500/v1.0/invoke/{app-id}/method/{method}`. The `app-id` here is `products-service`.

#### Step 3: Dockerize Both Services

1. **`Dockerfile` for Products Service**:
   ```Dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
   WORKDIR /app
   EXPOSE 8080

   FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
   WORKDIR /src
   COPY . .
   RUN dotnet restore
   RUN dotnet build -c Release -o /app/build

   FROM build AS publish
   RUN dotnet publish -c Release -o /app/publish

   FROM base AS final
   ENV ASPNETCORE_HTTP_PORTS=8080
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "ProductsService.dll"]
   ```

2. **`Dockerfile` for Orders Service**:
   ```Dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
   WORKDIR /app
   EXPOSE 8080

   FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
   WORKDIR /src
   COPY . .
   RUN dotnet restore
   RUN dotnet build -c Release -o /app/build

   FROM build AS publish
   RUN dotnet publish -c Release -o /app/publish

   FROM base AS final
   ENV ASPNETCORE_HTTP_PORTS=8080
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "OrdersService.dll"]
   ```
   >*Please note: Update the aspnet and sdk versions based on what framework the projects are targeting*

#### Step 4: Push Docker Images to Azure Container Registry (ACR)

1. **Create an Azure Container Registry**:
   ```bash
   az acr create --resource-group <your-resource-group> --name <your-registry-name> --sku Basic
   ```

2. **Push Docker images**:
   - Tag and push the **Products Service**:
     ```bash
     docker build -t productsservice .
     docker tag productsservice <your-registry-name>.azurecr.io/productsservice:v1
     az acr login --name <your-registry-name>
     docker push <your-registry-name>.azurecr.io/productsservice:v1
     ```

   - Tag and push the **Orders Service**:
     ```bash
     docker build -t ordersservice .
     docker tag ordersservice <your-registry-name>.azurecr.io/ordersservice:v1
     az acr login --name <your-registry-name>
     docker push <your-registry-name>.azurecr.io/ordersservice:v1
     ```

#### Step 5: Set Up Azure Container Apps with Dapr

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

#### Step 6: Test the Application

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

