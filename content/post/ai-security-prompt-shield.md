---
title: "Understanding and Mitigating Prompt Injection Attacks with Prompt Shield in Azure AI Studio"
date: 2024-07-13T21:45:08+10:00
draft: false 
slug: "AI Security against prompt injection"
tags: ["Azure AI", "AI Studio", "Artificial Intelligence", "AI Security", "Security", "Prompt Injection", "Prompt Shield"]
---
## Understanding and Mitigating Prompt Injection Attacks with Prompt Shield in Azure AI Studio

### Introduction

In the fast-changing landscape of Generative AI and its applications, keeping AI models secure and reliable is very important. Prompt injection is one of the prominent attack identified against AI Implementations. Azure AI Studio offers a solutions to tackle these threats and is called Prompt Shield. This blog will explain what prompt injection attacks are, their possible effects, and how Azure AI Studio's Prompt Shield can protect against them.

### What is a Prompt Injection Attack?

A prompt injection attack is an attack where an attacker manipulates the input prompt to an AI model to produce malicious or unintended outputs. This can lead to various consequences, such as generating inappropriate content, leaking sensitive information, or executing harmful instructions. These attacks exploit the model's response behavior by inserting malicious content into the prompt, tricking the model into following the attacker's intent.

The impact of prompt injection attacks can be significant:

-  Leakage of sensitive or confidential information.
-  Generation of harmful, inappropriate, or misleading content.
-  Execution of unintended commands in automated systems.

The attack can be initiated in two ways
- User Prompt Attack: The user deliberately exploit the system to get the AI Model to perform restricted actions. 
- Document Attack: The document submitted to the model contains hidden instructions to get the AI Model execute unintended commands.

### Mitigating Prompt Injection Attacks with Prompt Shield in Azure AI Studio

>Note: Some of the features discussed here are in preview at the time of writing.

Azure AI Studio provides solutions to address prompt injection attacks through its Prompt Shield feature. Azure AI Studio content filtering is built on top of the Azure AI Content Safety services. Here's how it is configured:

To implement Prompt Shield in Azure AI Studio, follow these steps:

1. **Enable Content Filtering which includes Prompt Shield**: In the Azure AI Studio, navigate to the Content Filtering section and enable the Prompt Shield feature in input filtering.

![Prompt Shield in Content Filtering](/blogimages/prompt-shield.png)

2. **Apply filter to model deployments**: Apply the chosen filters including the prompt shield to model deployment in the project. Alternatively, content filter created  can be chosen when a model is deployed.


### Conclusion

Prompt injection attacks pose a serious threat to the security and reliability of AI systems. Azure AI Studio's Prompt Shield provides a robust framework to mitigate these attacks. By implementing Prompt Shield, organizations can protect their AI models from malicious prompts, ensuring safe and trustworthy AI operations.
