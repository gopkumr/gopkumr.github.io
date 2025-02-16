---
title: "Building Multi-Agent Application with Semantic Kernel Agent Framework"
date: 2025-02-15T21:45:08+10:00
draft: false 
slug: "Multi-Agent Application SK Agent Framework"
tags: ["Azure", "AI", "Azure AI", "OpenAI", "Semantic Kernel", "AI Agents", "Azure AI Foundry", "App Development", "dotnet", "csharp"]
---

## Introduction

Recently, I explored Microsoft’s **Semantic Kernel Agent Framework**, an experimental extension (at the time of writing) designed to build AI agents and integrate applications using the same features as Semantic Kernel. 

Here’s my take on it, including what I built, the key parts I used, and how it made my life easier.

## What I wanted to create

I wanted to create a multi-agent application that seamlessly integrates business functions with AI agents. Here are the key capabilities I was looking for:

*   **Defining Agents**: Multiple agents, each with specific skills and capabilities.
    
*   **Enabling Communication**: Agents that can communicate together, leveraging Semantic Kernel skills for task execution.
    
*   **Defining Coordination**: A set of rules to decide which agent is selected to fulfill a user query and how to identify if the assigned task is completed.
    

## Components from the Agent Framework I used

*   **ChatCompletionAgent**: Represents an AI agent built around the AutoCompletion API. This class helps configure a name for the agent, instructions that define the agent’s responsibilities, and a Semantic Kernel instance to connect to your preferred LLM. Integration with Kernel brings capabilities like adding skills to call application functions.
    
*   **AgentGroupChat**: Represents multi-agent interaction with built-in group chat history and options to add agent selection and conversation termination strategies.
    
*   **KernelFunctionSelectionStrategy**: A prompt-based strategy that defines the rules for selecting an agent from the group chat to execute a user query. The strategy is fed into a Kernel connected to an LLM model to execute the strategy and select the agent, enabling coordination.
    
*   **TerminationStrategy**: Another prompt-based strategy that defines the rules for deciding if the user query has been answered and if the agent conversation can be terminated. It also allows defining the maximum number of iterations the agents converse before terminating the conversation to avoid long or endless chats.
    
*   **SemanticKernel Kernel & Plugins**: Supports connecting with LLM services like OpenAI/Azure OpenAI, Ollama, and adding plugins to call into application code.
    

## Here is the high-level application design

![Application High-Level Design](/blogimages/sk-agent-app.png)
*Figure 1: Application design*

The application was designed as a modular monolith and consisted of the following components:

*   **User Interface (Frontend)**: Confined to the UX namespace, any kind of chat interface can be built using a tech stack such as Blazor or a console app (I built mine using [Spectre.Console](https://spectreconsole.net/).)
*   **Backend (Semantic Kernel with Agent Framework)**: The AI namespace consists of all AI-related components that depend on Microsoft.SemanticKernel, Microsoft.SemanticKernel.Agents, Microsoft.SemanticKernel.Connectors to implement kernels connected to deployed models, plugins, agents, group chats, and Azure.AI.Projects to use models deployed via Azure AI Foundry project.
    
*   **Backend (Business Logic)**: The business logic namespace contains the rest of the application logic that performs the business/functional requirements, such as implementing business logic or connecting to other dependencies using different SDKs like a database or a message broker.
    

## What Worked Well

*   **Seamless Integration with Azure OpenAI and Ollama**: Semantic Kernel makes integrating with different LLM deployments easy.
    
*   **Composable Architecture**: The agent framework allowed easy addition of agents, and combining it with SK Kernel makes it a perfect choice to give the agent capabilities like function calling using plugins, which was a key requirement in my use case.
    
*   **Effective Collaboration**: The agent framework’s group chat feature makes it easy to implement chat between agents. Fine-tuning the selection and termination strategy allowed me to implement scenarios where agents worked together to solve a two-step problem, which was fascinating.

## Challenges Encountered

*   **Fine-tuning agent selection and conversation termination prompts**: I had to run and fine tune the prompts for agent selection and conversation termination to get the LLM to choose the correct agent for the task. After multiple rounds of fine-tuning, I could get the agents together to solve a 2-step query but not beyond. I am sure further tuning could help resolve this.
*   **Memory management**: Two LLMs working together and using function calling huge chat history. Which at times causes the API calls to agents to fail, potentially due to token limit. Chat history reduction techniques will need to be implemented to avoid this.    

## Conclusion

The **Semantic Kernel Agent Framework** is a cool way to build smart, multi-agent apps. It’s still experimental, but it shows how modular, AI-driven agents can be orchestrated for complex workflows. If you’re looking to build AI-powered assistants, this framework offers a structured way to create skill-enabled agents and integrate them with your app.