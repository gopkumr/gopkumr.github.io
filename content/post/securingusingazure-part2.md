---
title: "Identity in Microsoft Azure - Modern Authentication"
date: 2021-08-01T18:06:02+10:00
draft: false 
tags: ["Azure", "Security", "OAuth", "Authentication"]
---

## Introduction
Continuing from the [previous post](/content/post/securingusingazure-part1), the new generation of authentication mechanism was created to satisfy the new generation of application, starting from apps that run just in the browser to apps that run on micro-controllers. This new generation of authentication mechanism called as the modern authentication protocols are built on top of the OAuth protocol and taking inspiration from SAML.
In the below article the term IDP refers to the Identity provider, the external service that is responsible for authenticating a user and issuing authorization tokens. This service is both trusted by the client app as well as the resource api.

## Authentication
OpenID connect is a protocol built on top of OAuth 2.0 to authenticate users and communicate the identify information to the service provider application. 

![OpenID connect](/blogimages/security/OID.png)

When an anonymous user tries to access a service provider web application, the user is redirected to an identity provider which supports OpenID connect.  Identity providers can use a username/password or any other means to identify the user and issue a token called ID token. This token is then passed onto the service provider application which can inspect the token to get the user identity information. User can access the application until the user logs out or the ID token expires, after which users will be redirected to identity provider for authentication.

The ID token is often persisted in a cookie and is sent across in requests, so that the service provider application will know the user is an authenticated user. 
Due to the presence of this ID token, if the user tries to access another application that trusts the same identity provider, user will not be asked to authenticate again, only further consent if required might be asked and new ID token for the second application will be provided and redirect direct to the application, thus implementing a single sign-on behavior.

ID Token contains a AUD claim which represents an audience which is the application that the token targets, hence each application needs a separate token. ID token is purely used for user identification and not for authorization. Authorization like in case of accessing an API should be always done using an access token.

## Authorization
Once the user identity is known, the application or API needs to decide what functionality does the user have access to. The ID token cannot be used for this, as it just says who the user is, to know all the roles or groups that user belongs to, the application required a new token access token. Identity provider gives back the access token for the application or API when the user is authenticated. 
If the user, through the application, accesses multiple APIs, the identity providers will give multiple access tokens for each APIs. 
Access token contains all the roles/groups that user are member as claims. Applications/APIs can access these token value to validate the roles/group and grant access to functionality.
Access tokens are short lived for security, so while the user is using the application/API if the access token expires, the application/API can use another token called Refresh token to get a renewed access token. 

![AccessToken](/blogimages/security/AccessToken.png)

 Almost all the applications would talk to a resource to get required data and would need an access token to do so. An access token cannot be issued unless the user's identity is  established, hence in almost all scenarios, the flows described above are used together. Below are different scenarios where different authentication and authorizations flows are used.

## Authentication and authorization flows

### Authorization code flow
The flow that is described under the *Authorization* section above is the authorization code flow. The flow in which the client application receives a authorization code after the IDP authenticates the user. This code is used along with the client apps secret to get the access token from the IDP's token end point. This access token is used to get the data from resource api.

Auth code flow was primarily intended to be used by web application deployed on a server, hence the auth code and a client secret has to be submitted over https to get the access token. But this was a challenge to apps like single page javascript based apps which executed completely on a browser with no supported server. At the time Javascript calls to a domain other than the one loaded it was prohibited (*before CORS enabling was a thing*). Hence the next flow.

## Implicit Grant flow
This flow is like the authentication code flow, except that after authenticating the user, the IDP returns the Access token instead of the auth code. This was the client app can get hold of the access token and make direct calls to the API.

![Implicit Grant](/blogimages/security/ImpliciteGrant.png)

This flow had security issues, where the client app could be injected with a malicious cross site scripted page and the response from the IDP could go directly to a malicious app. Once the malicious app gets the access token, it can perform actions as the user. 
Also since the app runs in an untrusted environment, it cannot be trusted with a secret like a client secret key etc, as anyone can inspect the app get to know the secret values. 

Because of these reasons, this flow is not recommended for client side running without a server behind. Hence the next flow.

## Authorization code flow with Proof key for code exchange (PKCE)
To protect the authorization code flow from the above mentioned vulnerabilities, another layer of security is introduced. The flow is similar to the authorization code flow except that the client app generates a verifier code and produce a challenge word cryptographically and is submitted as part of the initial authentication redirection to IDP.
IDP then stores this challenge and the auth code is sent back. The client app then posts the auth code and the verifier code that was generated previously (not the challenge) to the token end point, which the IDP validates and send back the access token.
The introduction of the verifier and its challenge, which is runtime generated by the client app stops the malicious actors from using a CSRF the request and getting hold of the access token. 
 
![PKCE](/blogimages/security/OID-PKCE.png)

## Client Credentials flow
The above described flows enable a resource owner to use a client application to access the resource (API) by authenticating/Authorizing himself and letting the client application perform the resource retrieval on behalf of the resource owner. But there are instances where there is no user involved, where the resource owner is an application/machine scenarios such as an IOT device pushing sensor data to the server or a demon service connecting to an API to get a persisted state or data. In such a scenario, the service/machine authenticates itself using a secret that is agreed between the service and IDP, this secret can be a key or a certificate. Once the service/machine sends the secret to the IDP, the IDP generates an access token with all the permissions that are configured for the service/machine and responds back.

![Client Credential](/blogimages/security/OID-ClientCredential.png)

## Conclusion
The above mentioned flows covers majority of the interactions between the resource owner, client application, IDP and the API, but there are other flows which is variations of these used in difference scenarios and different IDPs implement different scenarios. Also, there are new ways to authentication/authorization being implement as newer scenarios/devices/user cases emerges.
Azure AD is such an IDP which implements authentication/authorization flows and can be used by applications hosted in Microsoft Azure or elsewhere. 