---
title: "Authentication in Microsoft Azure - A bit of history"
date: 2021-07-24T19:16:16+10:00
draft: false 
tags: ["Azure", "Security", "OAuth", "Authentication"]
---

## Introduction
Authentication has been an important component in the world of IT from the time companies required their employees to prove their identity to use the company's computing resources whether it was to execute its business processes or accessing email or file. During the earlier days employees used to login to their computers using a username and password, which was stored in a central server like an active directory (in case of Microsoft tech stack). With the active directory credentials employees where able to use to login to both their windows computers as well as the email application both of which were in the same network. This approach worked well for many years until the softwares and services that the companies used where no longer within their network.   
While active directory protocols like NTLM or Kerberos could work across external networks via technologies like VPN it was complex to setup and maintain such an infrastructure while keeping all the connection secure and stable. Also with growing number of users/services and the pace at which the growth occurred, these technologies were not designed to scale at that pace. Hence new Authentication mechanisms were needed.

## New Generation of Authentication
The new generation of authentication were developed enabling accessing to employees to access services external to the company network as well as external users/services to  company's resources securely. 
This new generation of authentication mechanism differentiate two parties in the use case
- Identity provider: who provides the user profiles and authentication services.
- Service provider: Who provides the secured resource to authenticated user/service. 
  
With this new approach the service provider is only concerned about providing the resource, leaving all the authentication complexities offloaded to the identity provider.  This provides companies as service providers time to focus purely on their business and deliver value to their customers also identity provider can be enhanced or replaced with no impact to the service provider.

Main NewGen Authentication mechanisms are
- WS-Fed
- SAML
- OAuth
- OpenID Connect

**WS_Fed & SAML**
>Protocols build for authentication of web applications and uses SAML tokens as a proof of authenticated user. When an anonymous user tries to access a secured web application via his browser, he is redirected to a identity provider. User then proves his identity by entering username/password or any other means. The identity provider validates and generates a SAML token and the user is then redirected to the web application, which validates the token and lets the user access the application. From this moment onwards its the user via his browser communicate directly with the web application passing the token as a proof of his identity, until he logs off. The SAML token is most often stored in a browser cookie which is submitted to the web application each time a request is sent.

**OAuth & OpenID Connect**
>OAuth is not an authentication protocol, rather it is a access delegation protocol. When a web application/service provider wants to authenticate a user, it redirects the user to an identity provider who then authenticates using username/password or any other means. The identity provider also gets the users consent to provide identity information to the web application/service provider and generates token, which is then used to access the web application/service provider. The problem with OAuth was that there was no standard in the way user identity was represented, each identity provider has its own way of representing user identity, so often the web application/service providers has to maintain different logic to access user information for different identity providers.

OpenID Connect solves this very problem with OAuth. OpenID connect is an authentication extension built on top of OAuth 2.0. When using OpenID Connect the identity provider provides a ID Token, Access Token and a userinfo endpoint after a user is authenticated. ID Token contains the identity information about the user which the web application/service provider can use to identify the user. Access token contains the user authorization information that can be used to provide access to specific resources within the service provider. The userinfo endpoint is used to retrieve the user information passing the access token when the service provider does not have the ID Token.

## Next step
With different kinds of service providers like web applications, REST APIs and different consumers like Services, Mobile Apps and Single page apps, there are different scenarios where authentication need to be implemented. The next article will be to look into OpenID Connect/OAuth to solve each of these scenarios.

