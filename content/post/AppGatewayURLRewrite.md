---
title: "Configuring Azure Application Gateway for API Management Traffic Routing"
date: 2023-12-02T14:00:48+11:00
draft: false
tags: ["Azure", "App Gateway", "API Management", "URL Rewrite", "Config"]
---

## Introduction
Azure Application Gateway provides a powerful solution for load balancing, SSL termination, and URL-based routing. In this blog post, we will discuss a common scenario 
where we need to forward traffic to two different Azure API Management instances based on the incoming URL, distinguishing between non-production and production environments.

## Problem Statement
Consider a scenario where you have two separate instances of Azure API Management (Sku: any non consumption tier): one for non-production/testing (nonprod) and the other for production (prod). 
The requirement is to route incoming traffic through an Azure Application Gateway, forwarding requests to the appropriate API Management instance based on the path specified in the URL. 
Specifically, requests with the path /nonprod/* should be directed to the non-production API Management instance, while requests without this path should be forwarded to the production instance.

## Solution
Prerequisite: Two separate API Management instances are created - one for non-production and one for production and base URLs (FQDN) for both instances are known.

- **Step 1**: Configure Listener & Backend Pools in Application Gateway  
Create a listener with a public IP address  
Create two backend pools, one for non-production and one for production. Add the respective API Management FQDN  to each.  

- **Step 2**: Create a health probe  
Create a custom health probe to the API Management instance that sends request to /status-0123456789abcdef  
Pick the hostname and port from the backend settings.  

- **Step 3**: Set up Backend Settings   
Create Backend settings, a single setting can be shared by both API Management instance.   
Set the protocol to HTTPs, use the backend services well-known certificate, Override hostname from the backend target.   
Use the health probe from step 2  

- **Step 4**: Create new rule  
Create a path based rule for selecting the listener from step 1.  
In the backend targets select the backend settings and the production backend pool    
Add a path base rule  
 matching the path /nonprod/*:  select the backend settings and non production backend pool  

- **Step 5**: URL Rewrite Configuration  
Navigate the "Rewrites" and add a new rewrite rule set.   
In the ruleset   
Associate with the  the path based rule  
Add the below condition  
 ```
 IF  server_variable:uri_path equals(=) /nonprod/(.+)  
 THEN set URL Path = /{var_uri_path_1}
``` 

![image](https://github.com/gopkumr/gopkumr.github.io/assets/1662197/59b71fd8-fde0-4e22-991b-cd5c9adef30a)  


## Conclusion
Configuring Azure Application Gateway to route traffic based on URL paths to different API Management instances provides a  solution for managing non-production and production environments with same API routes. 
By setting up URL rewrite configurations when a request with nonprod in the path is processed by App Gateway, it matches the path to the non production API Management instance however, when app gateway forwards the requesty
it rewrites the URL without the nonprod in the path and hence matching the API routes. 
For requests without nonprod in the path, the default backend pool is used.
