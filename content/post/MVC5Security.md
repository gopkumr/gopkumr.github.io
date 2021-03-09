---
title: "ASP.Net MVC 5 and Security"
date: 2017-10-14T10:51:44+11:00
draft: false
---

## Security?

Security is one of the most important cross-cutting concern for any web application. All applications (except for static web sites) require to identify a user and restrict the users from viewing or performing actions on pages.

## Authentication

Authentication is the method by which an application identifies a user. By identifying a user, the application can decide whether the user is a valid user to access the application.

## Authorization

Authorization is the way the application decides if the identified user can view a particular page or perform a particular action.

ASP.Net MVC 5 has introduced ASP.Net Identity which supports implementing security concerns in MVC applications. I will start with an empty ASP.Net MVC Project and then try to add security step by step.

## A bit of history
Microsoft introduced Membership Provide with ASP.Net 2.0 back in the days to handle all security-related concerns. ASP.Net Membership provider supported Forms Authentication with SQL Server database which stored user names, password, and other user data.

Membership provider supported database schema which uses SQL Server. Any additional profile data is stored in separate tables and accessed using the Profile provider API. The membership provider providers interfaces to write customer providers to handle data storage differently.

While Membership provider was service the purpose during its time, modern web applications got different ways to handle membership, authentication, and authorization.

## The fresh look
Modern applications have a different approach to security with multiple ways of handling authentication and authorizations. Membership systems now have to support web, mobile and restful API applications, also social logins and cloud-based authentication platforms.

Microsoft introduced ASP.Net Identity as part of the MVC framework to cater to all the requirements of the modern day applications whether it is web, mobile, store or hybrid applications.

Introduction of ASPNet Identity brings in role providers, social logins, claim based and is based on OWIN specification, which makes it a component that can be hosted in any OWIN based hosting providers and it is independent of System.Web library.  ASPNet identity is distributed via Nuget and hence it gets introduced only in a project that requires it.

## Now to get into the details

**Dependencies**

To introduce ASPNet Identity to any ASP.Net project, you need to install the below NuGet packages.
- Microsoft.AspNet.Identity.Core
  
    Core package of the identity that can be used to write implementations to target user data storage to different persistence types like a NoSQL database.
- Microsoft.AspNet.Identity.OWIN
  
    The package that introduces OWIN middleware supporting identity like login/logout, cookie creation, etc.
- Microsoft.AspNet.Identity.EntityFramework
  
    The package that introduces EF implementation of ASPNet Identity to persist the identity data into SQL Server.

**User registration, Login/out**

Once the packages are introduced, the UserStore and PasswordStore interfaces can be implemented to alter storage. Using the UserManager, SignInManager and AuthenticationManager classed provided, user registration and login/logout can be handled.

A startup class needs to be created and decorated with OwinStartup assembly attribute and cookie authentication middleware from OWIN can be added to app builder to enable cookie creation. The startup class "configure" method can be used to configure other middlewares like the social login middleware.

**Roles, Claims, and Identity in request context**

The OWIN middleware give options to use the default middleware to generate identity or override the existing providers to add functionality to the identity of the user in the request context.
The default CookieAuthenticationProvider can be overridden to add custom logic to add claims or roles to the Identity user instance and inject claims identity into context.

This injected user identity instance can be accessed across the layers and required authorization logic can be added either through custom ClaimsAuthorize authorization filter or by accessing the ClaimsIdentity instance directly from the HttpContext principle and checking the presence of a particular claim.

There are many references available that details the way ASPNet identity is implemented below are a few links.

- [Introduce ASPNet Identity to existing projects.](https://docs.microsoft.com/en-us/aspnet/identity/overview/getting-started/adding-aspnet-identity-to-an-empty-or-existing-web-forms-project)
- [Claim based security MVC](https://www.codeguru.com/csharp/.net/net_security/asp.net-mvc-and-claim-based-security.html)