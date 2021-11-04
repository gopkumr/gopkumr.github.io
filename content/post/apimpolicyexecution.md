---
title: "Policy Execution in Azure APIM."
date: 2021-10-27T18:50:46+11:00
draft: false
tags: ["Azure", "API", "API Management", "API Gateway"]
---

## What are APIM Policies?
 APIM policies are statements executed by Azure APIM to modify the behavior of API request, response and exception flows. The logic/conditions written as part of the policies are executed at various stages of API execution like,  *request received (inbound)*, *before request sent to backend service/API (backend)*, *before sending  response to requester (outbound)* and *in case of any exceptions during the request processing (on-error)*. Policies are defined as an XML format with different tag to define the execution stage and the actual policy. 


 Structure of a policy XML (as mentioned in Microsoft documentation)
  ```xml
 <policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies>
  ```

## Different levels of Policies
![APIM Policy Hierarchy](/blogimages/apim-policylevels.png)

Policy statements for each execution stage can be defined at 4 level, Global policies defined at the *All APIs* section, Product level defined at the product details page, API level defined at each specific API section and Operation level defined at the specific operation section. The execution of the higher hierarchy level policy can be controlled using a special tag `<base />` defined at each of the lower levels. The base tag can be used to control the order of execution as well as to skip execution of higher level policies. 

Below is an example to illustrate the use of base tag.

Global Level Policy
```xml
<policies>
    <inbound>
        <ip-filter action="allow">
          <address>10.0.0.120</address>
        </ip-filter>
    </inbound>
</policies>
```

Operation Level Policy
```xml
<policies>
    <inbound>
        <rate-limit calls="20" renewal-period="90" />
        <base />
    </inbound>
</policies>
```

Even though the Global policies are at a higher level in the hierarchy, the ip-filter policy is executed after the rate-limit policy as the operation level policy specifies the `<base />` after the rate-limit policy. This was each lower level policies can control the way the policies are executed in the hierarchy.

You can even skip the execution of upper level policies by removing the base tag. So of the base tag is removed at the operation level then all the policies defined at  Global, Product or API level will be ignored and only the operation level policy will be executed. 

You can view the effective policy on an operation by opening the policy window at the operation level and choosing to view effective policy. This option will look at the hierarchy and show the policies applies on the operation along with the order in which it is applied.

![APIM Effective Policy](/blogimages/apim-effectivepolicy.png)

## Product Level policies
When an API is added to a product, the policies that are defined at the product level gets applied to the API calls. An API can be added to more than one product, in such a case APIM will look at the subscription key passed in the request to apply the corresponding products policies. In the event the API method is triggered using a subscription key that does not belong with a product (e.g. a master subscription key) none of the product policies are applied and the operation is trigger passing through the Global, API and operation level policies. 

One such use case would be an API listed under a starter product where a rate limit policy is applied and also list under an unlimited product where there is no rate limit restriction. 


## Conclusion
APIM policies are powerful yet easy to implement feature that helps in implementing many common use cases using [built in policies](https://docs.microsoft.com/en-us/azure/api-management/api-management-policies). For more advanced and custom scenarios, the policy definition also support [C# syntax expressions](https://docs.microsoft.com/en-us/azure/api-management/api-management-policy-expressions) and a subset of .Net Framework types, so the opportunity to extend the policies are limitless. 