---
title: "Infrastructure as C#"
date: 2020-07-26T18:52:19+11:00
draft: false
---

## Introduction
After attempting the .Net tutorial on deploying a simple WebAPI based microservice to Azure Kubernetes Service (AKS), wanted a better way to represent my infrastructure than the YAML.xml file. This was partly because of me being novice in YAML format and partly to have a way to abstract the infrastructure in order to make it repeatable and it should be not just confined to AKS. 
The first solution to this problem was to use a framework like Terraform to define my infrastructure as code. But this will lead me learn new language and language constructs like loops, conditions etc. 
The search was over pretty soon after I found a framework called Pulumi, that lets me write my infrastructure in many of the populate programming language including C#. So i decided to convert the .Net tutorial YAML into a pulumi project and see how well it runs.

.Net tutorial link: https://dotnet.microsoft.com/learn/aspnet/deploy-microservice-tutorial/azure-tools

## Implementation
To start with, use the default WeatherForecast web api project. Add a dockerfile from the dotnet tutorial link and create docket image. Publish the docker image to the docker hub repository. All the steps till here is exactly similar to what is mentioned in the tutorial.

Install Pulumi from https://www.pulumi.com/. In windows, it is as simple as extracting the zip file and adding the bin folder to the PATH environment variable.

Using Pulumi CLI, create a new kubernetes-csharp project.

```
pulumi new kubernetes-csharp
```

give the appropriate project name, description, stack (i choose to name my stack dev) and a C# project will be create for your use.

Stack in pulumi represents different configurations that you require for each environments that you would deploy to. You can have dev, test, prod stacks based on your project's environment requirements. Each stack is represented my separate classes and hence separates the configuration between environments.

The stack class representation uses similar class and property name as the YAML file, hence it is not very difficult to translate, but pulumi helps is giving the infrastructure config a object oriented approach,so we can reuse many repeated properties like the label.

Use the pulumi API to define the deployment class configuration and the service class configuration. All the values in the configuration is directly from the YAML file. I used a variable to store the app label so that i can use it across both deployment and service configuration. 

To run the pulumi code, first create the required Azure resources like the resource group and the AKS cluster. You can follow the exact same steps as in the tutorial or use the portal via the browser. Using the az aks get-credential command to download and save the SSH keys in the local computer, which pulumi will use to deploy to the cluster.

```
pulumi up 
```

Use Pulumi Up commend to deploy to the cluster, and the response of the execute will be the IP address of the load balancer as written in the code. The best part of pulumi is that you can modify the code and do a pulumi up again it will deploy only the difference or only the changes to the cluster.

You can now test the route using ```http://<IP Address>/WeatherForecast```

Once all the testing is completed, you can delete the deployed code using pulumi destroy command.

```
pulumi destroy
```

All the code written for this attempt is available in github: https://github.com/gopkumr/pulimi-kubernetes.git

