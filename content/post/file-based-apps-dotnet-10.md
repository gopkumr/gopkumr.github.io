---
title: "Exploring File-Based C# Scripts in Azure DevOps Pipelines"
date: 2025-12-03T17:39:11+11:00
draft: false
tags: ["dotnet", "c#14", "dotnet10","Azure DevOps"]
---

# Exploring File-Based C# Scripts in Azure DevOps Pipelines

What if you could replace your Bash or PowerShell scripts with C#? With .NET 10's **File-Based Apps**, you can run a `.cs` file directly—no project files, no build step.

In this post, we'll explore how to use C# scripts in Azure DevOps pipelines, using a real example: deploying an AI agent with a single `.cs` file.

---

## Why C# for Pipeline Scripts?

Pipeline scripts are typically written in Bash or PowerShell. But what if you already know C#? With .NET 10's file-based apps, you can now use C# as a scripting language—similar to how Node.js or Python work.

Previously, this required community tools like [dotnet-script](https://github.com/dotnet-script/dotnet-script). Now it's built into the platform.

A file-based app is simply a `.cs` file you run directly:

```bash
dotnet run deploy-agent.cs
```

No `.csproj`, no `dotnet build`, no output folders. Just your script and `dotnet run`.

---

## What Makes C# Scripts Useful in Pipelines?

### Inline Package References

The key feature for pipeline scripts is the `#:package` directive. You can reference NuGet packages directly in your code:

```csharp
#:package Azure.AI.Agents.Persistent@1.2.0-beta.7
#:package Azure.Identity@1.17.0

using Azure.AI.Agents.Persistent;
using Azure.Identity;

// Your deployment logic here...
```

The runtime downloads and references these packages automatically. No `PackageReference`, no restore step.

### Full C# Language Support

You get the complete C# language: async/await, LINQ, strong typing, and IntelliSense in your editor. This is a step up from Bash scripts where complex logic can become hard to maintain.

### Linux Pipeline Agents

On Linux-based pipeline agents, you can even use shebang syntax:

```csharp
#!/usr/bin/env dotnet run
Console.WriteLine("Running on Linux agent");
```

---

## Example: Deploying an AI Agent from a Pipeline

Let's look at a practical example. This C# script deploys an AI agent to Azure AI Foundry:

```csharp
// deploy-agent.cs
#:package Azure.AI.Agents.Persistent@1.2.0-beta.7
#:package Azure.Identity@1.17.0

using Azure.AI.Agents.Persistent;
using Azure.Identity;

var endpoint = "https://your-resource.services.ai.azure.com/api/projects/your-project";
var _agentsClient = new PersistentAgentsClient(endpoint, new AzureCliCredential());

var agentResponse = await _agentsClient.Administration.CreateAgentAsync(
   model: "gpt-4o",
   name: "DeployedAgent",
   instructions: "You are a deployed agent. Respond always with your deployment context.",
   description: "This is a deployed agent created via file-based app.",
   cancellationToken: CancellationToken.None);

Console.WriteLine($"Agent deployed with ID: {agentResponse.Value.Id}");
```

### Why This Works Well in Pipelines

- **Self-contained**: Dependencies are declared in the file itself
- **No build artifacts**: No obj/bin folders to clean up
- **Easy to review**: One file in your PR shows all changes
- **Runs anywhere**: Works locally and in the pipeline with just `dotnet run`

---

## The Azure DevOps Pipeline

Here's how to run the C# script in Azure DevOps:

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - master

pool:
  name: 'Azure Pipelines'

variables:
  - name: dotnetVersion
    value: '10.0.x'

stages:
  - stage: DeployAgent
    displayName: 'Deploy AI Agent'
    jobs:
      - job: RunDeployScript
        displayName: 'Run Deploy Agent Script'
        steps:
          # Install .NET 10 SDK
          - task: UseDotNet@2
            displayName: 'Install .NET SDK'
            inputs:
              packageType: 'sdk'
              version: '$(dotnetVersion)'

          # Azure CLI login and run the deploy script
          - task: AzureCLI@2
            displayName: 'Run Deploy Agent Script'
            inputs:
              azureSubscription: 'your-azure-service-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # The script uses AzureCliCredential which leverages the Azure CLI session
                dotnet run ./deploy-agent.cs
              workingDirectory: '$(Build.SourcesDirectory)'
```

### What's Happening Here

1. **Install .NET 10**: The `UseDotNet@2` task sets up the SDK
2. **Azure CLI login**: The `AzureCLI@2` task authenticates, and `AzureCliCredential` in our script picks up that session
3. **Run the script**: `dotnet run ./deploy-agent.cs` handles package restore and execution in one step

No separate build task. No artifact publishing. Just source to execution.

## Conclusion

If you know C#, you can now use it for pipeline scripts. With .NET 10 file-based apps, you get type safety, IntelliSense, and the Azure SDK—all in a single `.cs` file that runs with `dotnet run`.


---

## Resources
- [Tutorial: Build File-Based C# Programs](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/file-based-programs)
- [What's New in C# 14](https://learn.microsoft.com/dotnet/csharp/whats-new/csharp-14)
- [Azure AI Agents SDK](https://learn.microsoft.com/azure/ai-services/agents/)

