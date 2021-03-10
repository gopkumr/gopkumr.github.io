---
title: "Hosting Blazor WebAssembly on ASP.Net Core WebAPI"
date: 2020-06-09T18:52:19+11:00
draft: false
tags: ["blazor","WASM", ".Net","WebAPI", "c#"]
---

## Background
My WebAssembly project has now been configured to be a PWA (refer the previous article in series). It time to introduce hosting. Since the WebAssembly project handles the client side, I want it to be unchanged but be hosted it in a project that can be used as backend for the UI, hence chose WebAPI.

## The Changes
- Create a new solution and add the already created Blazor WebAssembly project
- Add a new ASPNet core web project and choose WebAPI template and call it the .Server project
- Add reference of the WebAssembly Project to the .Server project.
- Install package Microsoft.AspNetCore.Components.WebAssembly.Server to the .Server project. This package contains the runtime server for Blazor application.
- In the startup class add configuration to the request pipeline to handle Blazor and its routing.

```c#
// This methods serves the WebAssembly framework files when a request is made to root path. 
//This method also take path parameter that can be used if the WebAssembly project is only served 
//from part of the project, giving options to combine web assembly project with a web application

app.UseBlazorFrameworkFiles();
```
```c#
//This configuration helps in serving the static files like 
//Javascript and CSS that is part of the Blazor WebAssembly

 app.UseStaticFiles();
```
```c#
//Add the below configuration to the end of the UseEndpoint configuration, 
//this will serve the index.html file from the WebAssembly when the WebAPI route 
//does not find a match in the routing table

 endpoints.MapFallbackToFile("index.html");
```
Your ASPNet Core hosted WebAssembly project is ready to be published and deployed. Pretty easy!

Next would to move some of the data source maintained in the WebAssembly to the WebAPI Server and use an HTTP call to retrieve it.

Source Code: https://github.com/gopkumr/BlazorTourOfHeroes.git 
Branch: Perf
