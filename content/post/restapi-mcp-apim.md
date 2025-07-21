---
title: "Exposing REST APIs as MCP Servers with Azure API Management: Two Approaches"
date: 2025-07-20T19:00:51+10:00
draft: false
slug: "Exposing REST APIs as MCP Servers with Azure API Management"
tags: ["AI", "MCP", "Model Context Protocol", "Azure API Management", "AI Gateway"]
---

# Exposing REST APIs as MCP Servers with Azure API Management: Two Approaches

## Introduction

The Model Context Protocol (MCP) has changed the way how AI applications interact with external data sources and services, and the need to securely expose REST APIs as MCP servers has becomes more critical. Azure API Management provides an enterprise-grade solution that facilitates this transformation and along with its robust security, monitoring, and governance capabilities.

In this post, I explore two approaches to expose REST APIs as MCP servers using Azure API Management:

1. **Custom MCP Server Application**: Building a dedicated MCP server application and exposing it through API Management
2. **API Management MCP Server (preview)**: Leveraging Azure API Management's preview feature for out of the box MCP server creation

Both approaches can leverage Azure API Management's policy engine to implement security, rate limiting, and other enterprise requirements.

## Approach 1: Custom MCP Server Application with Azure API Management

The first approach involves creating a custom MCP server application and then exposing it through Azure API Management. This method provides maximum flexibility and control over the MCP implementation.

### Building the Custom MCP Server

Create an MCP server using any supported language and esstentially create "tool" wrapping the API functionality. Below is a sample i created using dotnet and the ModelContextProtocol library

### Tool
````csharp
[McpServerToolType]
public class WeatherPredictionTool(WeatherService weatherService)
{
    private readonly WeatherService _weatherService = weatherService;

    [McpServerTool, Description("Calls the Weather Prediction API and returns a weather forecast for a given city.")]
    public string ForcastWeather(string city)
    {
        return _weatherService.ForecastWeather(city)
             .ContinueWith(task =>
             {
                 if (task.IsFaulted)
                 {
                     return $"Error fetching weather data: {task.Exception?.Message}";
                 }
                 var forecasts = task.Result;
                 return string.Join("\n", forecasts.Select(f => $"{f.Date}: {f.TemperatureC}°C, {f.Summary}"));
             }).Result;
    }
}
````

### program.cs
````csharp
builder.Services
     .AddMcpServer()
     .WithTools<WeatherPredictionTool>()
     .WithHttpTransport();

var app = builder.Build();
app.MapMcp("mcp");

await app.RunAsync();
````

### Deploying to Azure API Management

Once your custom MCP server is built and deployed to a suiatable hosting service (App Service or Container Apps) you'll need to configure endpoints in Azure API Management:

1. **Deploy API**: Deploy an API with the backend as the MCP server exposing POST /message and GET /sse operations. Use bicep to deploy the API and Operations

### Sample bicep 
````bicep
resource mcpApi 'Microsoft.ApiManagement/service/apis@2023-05-01-preview' = {
  parent: apimService
  name: 'mcp'
  properties: {
    ...
    path: 'mcp'
    protocols: [
      'https'
    ]
    serviceUrl: mcpServerServiceUrl
  }
}

resource mcpApiPolicy 'Microsoft.ApiManagement/service/apis/policies@2023-05-01-preview' = {
  parent: mcpApi
  name: 'policy'
  properties: {
    format: 'rawxml'
    value: loadTextContent('mcp-policy.xml')
  }
}

resource mcpSseOperation 'Microsoft.ApiManagement/service/apis/operations@2023-05-01-preview' = {
  parent: mcpApi
  name: 'mcp-sse'
  properties: {
    displayName: 'MCP SSE Endpoint'
    method: 'GET'
    urlTemplate: '/sse'
    ...
  }
}

resource mcpMessageOperation 'Microsoft.ApiManagement/service/apis/operations@2023-05-01-preview' = {
  parent: mcpApi
  name: 'mcp-message'
  properties: {
    displayName: 'MCP Message Endpoint'
    method: 'POST'
    urlTemplate: '/message'
    ...
  }
}
````

2. **Apply Policies**: Implement security and transformation policies.

### Sample API Management Policy with just rate limiting, this could any supported policy.

````xml
<policies>
    <inbound>
        <base />
        <!-- Rate Limiting -->
        <rate-limit-by-key calls="100" renewal-period="60" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
````

## Approach 2: One-Click MCP Server (Preview Feature)

Azure API Management's one-click MCP server feature (currently in preview) allows you to instantly convert existing REST APIs into MCP-compatible servers without writing custom code. 
```Please note: Since this feature is preview while writing this post, the feature and functionality may change when it gets general availaiblity.```

### Enabling the Preview Feature

To use the one-click MCP server feature:

1. Navigate to your Azure API Management instance
2. Go to the "Deployment + Infrastructure" section and open "Service Updates" page 
3. Edit Update Group and enable "AI Gateway Early", wait for around couple of hours or so to recieive the update
4. Navigate to the portal using https://portal.azure.com/?Microsoft_Azure_ApiManagement=mcp and open APIM

### Configuring One-Click MCP Server

#### Prerequisite
Import the REST API into API Management using OpenAPI spec or deploying API, Operations separately.

#### Deploying MCP Server
1. Navigate to API Management (make sure you are using the URL that has this  query string "Microsoft_Azure_ApiManagement=mcp")
2. Open APIs section and open the MCP Servers page 
![MCP Server Page](/blogimages/apim-mcp-1.png)
3. Click on the Create MCP Server page and select the imported API and all the operations that you want to expose as tools
![MCP Server create](/blogimages/apim-mcp-2.png)
4. Hit Create and you have the MCP server availalble with an /sse endpoint ready to use
5. Clicking on the MCP Server name, directs you to a page where you can configure policies for the MCP server

Regardless of which approach you choose, you can implement APIMs robust security, monitoring and caching capabilities .


## Conclusion

Azure API Management provides powerful capabilities for exposing REST APIs as MCP servers through two distinct approaches. The custom MCP server application offers maximum flexibility and control, making it ideal for complex scenarios with specific requirements. The one-click MCP server feature provides a rapid path to MCP adoption with minimal development effort.

Whether you're just starting with MCP or looking to scale existing implementations, Azure API Management provides the tools and flexibility needed in AI-driven landscape.
