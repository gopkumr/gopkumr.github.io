---
title: "Azure API Management - What is it all about?"
date: 2021-08-25T12:44:46+10:00
draft: true 
tags: ["tag1", "tag2"]
---
## Introduction
Every thing about APIM, creating, hosting api, policies and different kind, subscruotion keys, products to distribute, Dev portal and testing it, monitoring and improve performance, app insight and key matrics. Infrastructure and pricing tiers and multi instances. VNet configs and application gateway and cost effective deployment. Configuration in repository. CICD with ARM templates. Managed Identities to interact with backend APIs.

## Creating API Management
To have an initial setup we are going to create the below resoources
Create Resource group
 Log analystics
 Application insights
 API M
Postal> search API management and create APIM. Pricing tiers, we'll use dev tier for learning purpose. Add an application insights too to the API management, you will have to create it a separate resource.  We also need to create a log analytics workspace too.

It take 30-40mins to create and once created will have a sample API already for us to refer to. We shall create an own API using the Add API option in the API Management

As for the example we will create an API management for the Open Weather Map API to get current weather and a 16 day forecast for passed zipcode and country or city name

We will use a function app to contain all the logic to call the open weather apis. Lets create a function app with based on .Net core 3.1 runtime. 
Lets enable the application insights to the same as we created to use with API management.

Create a function with HTTP trigger called CurrentWeather and the add code to call the open weather api data and return the data as it is. Also from teh integration tab of the function only enable the GET verb to make it clear to the consumers that the function is used to get data. *all the code references are available in the github repo*

Now we head back to the API management and import a function app and choose the function that we created. Which will create entries for the function app. We can test the flow of the call using the Test tab in the API Management. Another way to test is to run from the developer portal. You can straight away publish the portal and visit the portal and see the Current weather api published. We can test the API in the API management provided we register and get a subscription key to call the API. This is one layer of security in the API where the users are denied access to the api unless they have registered and got a  subscription key.

You will be able to view all the subscptions keys from the Subsciprions blade in the API management screen. You can switch off the suscription key requirement for a particular api using the settings tab of the API in the API Management and uncheck the Subscription required check box.

Developer portal
Create a new product and call it Weathers and with subscription and choose the open weather api functions that we added. The products is how the APIs are grouped togerther to be displayed in the developer portal.

Products
Access control in developer portal.  using Product  access control to give access to different groups and also set if the product required subscription or not. 

Creating products is a way to group the APIs to add control on all those apis by managing the it at a single product level, features like access control, subscription , approval flows for subscripts etc.

Policies
API policies like a rate limit, authentication, request/response trasformation etc can be implemented at a product level which will affect all the APIs or at an API level for indovidual APIs.

Scope of a policy, at a global level, product level, API level and operation level.

Mock for Apis policy
While the backend API is not yet available, an API management can mock an API. To do that in the API, we can define a response with a sample response header and body and then add a policy called  mock-response.

Throttling
How many calls an API can take in a duration of time limit can be added using the rate-limit-by-key policy. In the policy we can define the number of calls and the time frame the limit should be imposed. 

authentication policy
There are different way of authentication like basic authentication, authentication by client certificate.

check header
policy to validate the header values being passed

ip filter
Used to allow or deny any called from an IP or a range of IP to the API management end point

set quote usage
We can limit the API usage by number of calls or by bandwidth for a subscription by using this policy

https://docs.microsoft.com/en-us/azure/api-management/api-management-policies

revision
Used to handle upgrades without impacting current subscribers. Add a new revision using the revisions tab. Once you add a revision, you can choose it in the revisions tab,  all the changes that we make in APIM will impact only the new revisions and not the old revisions. Once you make the new revision as current, all the consumers will be moved to the new revision. But if any consumers want the old revision, they can still call it by passing rev=<revision> in query string, until the revision is taken offline.

version
Version is different from revision. You can only have one revision online at a time, but you can have mulitple version online with multiple cusumers using differetn versions. You can click on the API and add a new version and the changes make will only reflected in the new version added. While creating a version we can choose if we want to have version being passed as a header value or as a query string and this is used to identify which version if the API is being requested. The new version created will not be part of the product and it might need to added explictly.

Managed Identites 
To authenticate with azure functions we will use managed identities and use function keys to authorization. You can assign a system generated managed identities.
And in the azure function, in the authetication/authorization blade, enable app service authentiation and enable login with azure active diretory. And create an app for the azure function.
And also add a managed identity authentication to the inbound policy and give the function app url in the resource attribute


