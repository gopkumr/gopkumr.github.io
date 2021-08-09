---
title: ""Authentication in Microsoft Azure - Modern Authentication""
date: 2021-08-01T18:06:02+10:00
draft: true 
tags: ["Azure", "Security", "OAuth", "Authentication"]
---

## Introduction
From the [previous post](/content/post/securingusingazure-part1.md) its clear that there was a need a new generation of authentication mechanism that can satisfy the new generation of application, starting from apps that run just in the browser to apps that run on micro-controllers. This new generation of authentication mechanism called as the modern authentication protocols are built on top of the OAuth protocol and taking inspiration from SAML.

## Authentication
OpenID connect is a protocol built on top of OAuth 2.0 to authenticate users and communicate the identify information to the service provider application. 

[Flow]

When an anonymous user tries to access a service provider web application, the user is redirected to an identity provider which supports OpenID connect.  Identity providers can use a username/password or any other means to identify the user and issue a token called ID token. This token is then passed onto the service provider application which can inspect the token to get the user identity information. User can access the application until the user logs out or the ID token expires, after which users will be redirected to identity provider for authentication.

The ID token is often persisted in a cookie and is sent across in requests, so that the service provider application will know the user is an authenticated user. 
Due to the presence of this ID token, if the user tries to access another application that trusts the same identity provider, user will not be asked to authenticate again, only further consent if required might be asked and new ID token for the second application will be provided and redirect direct to the application, thus implementing a single sign-on behavior.

ID Token contains a AUD claim which represents an audience which is the application that the token targets, hence each application needs a separate token. ID token is purely used for user identification and not for authorization. Authorization like in case of accessing an API should be always done using an access token.

## Authorization
Once the user identity is known, the application or API needs to decide what functionality does the user have access to. The ID token cannot be used for this, as it just says who the user is, to know all the roles or groups that user belongs to, the application required a new token access token. Identity provider gives back the access token for the application or API when the user is authenticated. 
If the user, through the application, accesses multiple APIs, the identity providers will give multiple access tokens for each APIs. 
Access token contains all the roles/groups that user are member as claims. Applications/APIs can access these token value to validate the roles/group and grant access to functionality.
Access tokens are short lived for security, so while the user is using the application/API if the access token expires, the application/API can use another token called Refresh token to get a renewed access token. 

[Flow]

Now that the two flows in identity is discussed, we can look at different application scenarios where identity will be used.