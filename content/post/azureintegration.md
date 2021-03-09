---
title: "Azure for integration and process automation"
date: 2020-06-21T18:52:19+11:00
draft: false
---

## Problem
Businesses run on multiple applications and services, how well the business runs is often impacted on how efficiently data is distributed to the correct task. Automating this flow of data is a way to streamline the business. The problem here is to choose the right technology  for this data integration and process automation.

## Objective
This article is describing the azure technologies that are available during time of writing to solve the business need.

## Constraints
Time - Businesses does not have time to property integrate between the existing applications and services.
Money - Get the most out using the current infrastructure and technology stack.

## Approach
To get a high-quality produce and service to user is to design and implement strict business processes. The business processes will involve multiple steps, people and software. It may have branches and/or loops, some may run quickly and some may take days/weeks to complete.
The business processes modelled are called workflows and these workflows are implemented to integrated multiple systems. Workflows at a high level does four things
- Accept input values as data or file
- Execute actions that performs a logic to modify the data or trigger another action.
- Checks conditions to decide what actions to execute next.
- Produces an output, a piece of data or file which is a result of the whole workflow execution.  

## Options to consider
There are multiple options within Azure that can cater to the workflow requriement.
1. **LogicApps** - Given a GUI to design workflow using flow chart like controls. Actions can be customized using programming languages like C# and others. Has more than 200 connectors to integrate with external systems or options to write custom connectors in exception cases.
2. **PowerAutomate** - Built on top of LogicApps having all the designing capabilities but targeting the business analysts and non developer roles, hence there is no code editing possible. 
3. **WebJobs** - Part of the AppServices hosting fully code driven service. Run along other services hosted in AppServices, supports programming in C# and other languages. 
4. **Functions** - Best in the lot which just run a piece of code or a function on demand and charges only for the run or server-less. Options to scale based on demand. Supports a variety of programming languages and source control like GitHub or DevOps

Choosing between these services is very straightforward. If the workflow is going to be designed as a flow chart by a non-developer (where no custom connectors or interfaces required) then *PowerAutomate* is your way forward.  

*LogicApp* can give you flexibility to define your flows if a developer is involved, i.e. if the integration to external system are not available through an already available connector in *PowerAutomate* or you need customization over an already available connector.   
The added advantage of the flow chart like design based workflow tool is that the workflow itself exists as a documentation of the flow, that anyone can read and understand the flow just by looking at the workflow design in the browser or UI.

If the business process is being defined as part of an application development that the flow itself is driven by code, the options are *WebJobs* and *Functions*. *Functions* definitely has an edge over *WebJobs* when it comes to scalability and cost of owning. WebJobs can be handy if the current applications are part of AppServices and you want to keep all together or if you need customizations to external interfaces.

But there is nothing stopping to use multiple of the services to solve a specific business scenario. In the mix and match scenarios too there are services that works best together, like *LogicApp* and *Functions*, there are actions in *LogicApp* that can be added to the designer to call a *Function*.