---
title: "An attempt to convert Blazor WebAssembly Project to Blazor Server App"
date: 2020-04-25T18:52:19+11:00
draft: false
---

## Blazor Web-Assembly Project
This starts from my Blazor Web-Assembly project that I create as a replica of the Angular TourOfHeros tutorial. The source code of project is in [GitHub](https://github.com/gopkumr/BlazorTourOfHeroes.git)  

This is an attempt to convert the existing project to a Blazor server app with few changes to the wiring up and hosting configuration. Since this article is written with a pre-release version of Blazor Web-Assembly, there could be changes to the steps after the actual release expected in May 2020.

##Framework updates
The WebAssembly project was create in Jan 2020 when the Blazor Web-Assembly project was in preview, hence i have to change the TargetFramework and few assembly references in the project file.
* Updated Target Framework to `netcoreapp3.1`
* Removed references to Blazor assemblies from .Net Core 3.1.0-Preview build
* Removed `RazorLanguageVersion` specification

##Hosting and Startup changes
We need to added references to couple of Framework assemblies, namely `Microsoft.AspNetCore.App` and `Microsoft.NETCore.App` which provides the BlazorServer components.

To wire up Blazor Server app we first need to configure the dependency injection container. This is done by adding server side Blazor service in startup class's `ConfigureServices` method
`services.AddServerSideBlazor()`
Also add support for Razor pages `services.AddRazorPages()`, because, our Blazor components are Razor pages.

### Configuring the request pipeline
Unlike the Web-Assembly where we just bootstrap the first component and routing is performed at browser end, the Blazor server integrates with ASPNet Core routing and it uses BlazorHub to interact with Blazor components. For this we add the below configurations to the Configure method.

` app.UseStaticFiles();
  app.UseRouting();
  app.UseEndpoints(endpoints =>
    {
     endpoints.MapBlazorHub();
     endpoints.MapFallbackToPage("/_Host");
    });`

All the requests are directed to a Blazor component based on the URL defined using the @page directive, any URL that does not have a match in the routing configured to directed to the fallback page.
The `MapFallbackToPage` is used to define the component to be loaded when the request does not match any URL. The host file is named _host by-convention also this route has a lower priority in the routing table, hence Blazor can be coexist with other ASPNet core pages/controllers without causing conflicts. 

##_Host file
Since the host file is loaded by default in our project *(since we don't have any another routes configured)*, the code that we had in our `index.html` will need to be copied over to the host file along with couple lines of code modified.

* The JavaScript file supporting web assembly needs to be changed to the one 
  supporting Blazor server
  i.e. `<script src="_framework/blazor.server.js"></script>`
* Rendering mode of the initial component to be changed to server-pre-rendered
  `<app>Loading...</app>`
    to
  `<app> <component type="typeof(App)" render-mode="ServerPrerendered" /></app>`
   *We can pretty much bootstrap any component in the type attribute, we use the 
   App component as it is our parent component which calls all the rest of the 
    components.*

And that's pretty much it. Your WebAssembly app should be able to running in a ServerRendered Blazor server mode now. 

Both my WebAssembly and Blazor Server app code are available in [GitHub] (https://github.com/gopkumr/BlazorTourOfHeroes.git)
* WebAssembly is in the `master` branch
* BlazorServer is in the `serverApp` branch *(which i branched out and modified)*
