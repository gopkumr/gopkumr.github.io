---
title: "Securely calling Azure API from PowerApp Portal"
date: 2022-06-19T19:23:15+10:00
draft: false 
tags: ["PowerPlatform", "Azure APIM", "Powerapps Portal", "Security", "OAuth"]
---

## Context
Powerapps Portal gives a quick and easy way to build public facing websites. Data in the portal is mostly fetched from Microsoft Dataverse using Powerplatform FetchXML or the portal's Web API. These operations are secured using portal's application session, as explained [here](https://docs.microsoft.com/en-us/power-apps/maker/portals/web-api-overview). Often there are requirements to consume an externally hosted API, in this particular example an API hosted in Azure behind an API Management. With Javascript the only option to trigger an API, implementing a secret based authentication is out of scope. But there is an alternate approach.

## Approach

### Portal Configuration
The Powerapps portal gives an option to use OAuth implicit grant flow to obtain an ID token and use the token to securely call external APIs' where API can validate the token.

Powerapps portal comes with implicit flow enabled by default, but if required it can be enabled/disabled using the below website settings
```
Connector/ImplicitGrantFlowEnabled   True
```
Further, the website settings can be used to setup the token expiry time and client id too, that  supports securing the call.  

```
ImplicitGrantFlow/TokenExpirationTime    300   seconds
ImplicitGrantFlow/RegisteredClientId     portalname 
ImplicitGrantFlow/{portalname}/RedirectUri  http://portalname.powerappsportal.com
```   

Once the setup is done, we can use the portal api token endpoint in javascript to fetch the token. The client id can be passed as query string to the token URL to fetch the token with aud and appid claim as configured in the website settings.
```
<portal url>/_services/auth/token?client_id={portalname}
```
### Azure API Management Configuration
This example shows how Azure APIM is setup to validate the requests coming from the powerapps portal, however this approach can be used elsewhere which support OAuth token validation.

Below are the steps to configure APIM policies to validate the requests originated in the powerapps portal, as it is configured in the above section.   

#### CORS Setup
Since the calls are triggered from Javascript, the browsers validate for cross-origin requests. So APIM will have to enable cross-origin for the particular powerapps portal hosts.
```
<cors terminate-unmatched-request="true">
    <allowed-origins>
        <origin>portal url</origin>
    </allowed-origins>
    <allowed-methods">
        <method>GET/POST</method>
    </allowed-methods>
</cors>
```

#### Validate JWT Token
The ID token that is generated from the powerapps portal can use validated using the Validate JWT policy in APIM. The policy can be configured to validate the audience and issuer claims. The policy also validates the signature of the token to reject any tampering. 
Unfortunately, at the time of writing this article, the powerapps portal does not expose an openid configuration endpoint like AAD or any other identity provider, that can be used to validate the token. However, the portal exposes the public key that can be used. To use the public key in a policy below are the steps that needs to be done.
- Download the public key via ```<portal url>/_services/auth/publickey``` 
- Convert the public key in to modulus and exponent, sites like [this](http://superdry.apphb.com/tools/online-rsa-key-converter) can be used or library like BouncyCastle has this capability. 
- Modulus and exponent should be configured in the policy.
```
<validate-jwt
    header-name="Authorization">
  <issuer-signing-keys>
    <key n="<modulus>" e="<exponent>" />
  </issuer-signing-keys>
  <audiences>
    <audience>Client Id configured in portal</audience>
  </audiences>
  <issuers>
    <issuer>portal endpoint</issuer>
  </issuers>
</validate-jwt>
```

## Conclusion
The above setup lets the powerapps portal and the API hosted in Azure communicate securely. This  protects the API from being misused by anonymous users or an attacker. 

There is more that can be done, like the API can still is vulnerable to replay or DDOS attack. The portal lets you input a nonce value while generating a token, which along with rate-limit APIM policy can restrict replay and DDOS attacks.  

The API can be added to an APIM subscription and logic and limits can be addressed based on subscription. 

There are many possibilities, and options can be chosen based on the security requirements.