---
title: "a sneak peek into Blazor WebAssembly"
date: 2020-01-12T18:52:19+11:00
draft: false
tags: ["blazor","WASM", ".Net", "c#"]
---

# an attempt to create tour of heroes' using Blazor

## preface  
WebAssembly is an exciting piece of software, along with HTML, CSS and JavaScript WebAssembly (or WASM) is the fourth language that modern browsers can run natively, WASM is run in the browser in the same security sandbox as the JavaScript frameworks run. WASM also lets you invoke JavaScript and vice versa, making it coexist with JavaScript, More on WebAssembly here: https://webassembly.org/ and Blazor is an open-source implementation of WASM by Microsoft and it has made web development even more exciting by letting run the ever loved C# in the browser. Lets dive right into writing some code, you can read more about Blazor right from its creators here: http://blazor.net/.

## introduction  
Blazor WebAssembly supports writing single page applications (SPA). Like other SPAs,. the app package gets downloaded to client browser and is executed there, but blazor is written in WebAssembly as opposed to JavaScript.

What is better than the TourOfHeros app from Angular tutorial to get started with Blazor, refer the project goal here: https://angular.io/tutorial.

## setup  
Below are the tooling that is used for developing the app.  
**IDE**: Visual Studio 2019 16.0.10  
**Framework**: ASP.NET Core 3.1  
At the time of writing this article (Jan 2020) Blazor WebAssembly is in Preview, so the code might be different when the final product releases (Expected May 2020).


## coding  
We will start by creating a new Blazor App project. And the template that Visual Studio creates for you has all that is required for you to get started (as always) 
I am not going to detail each line of code, all the code written is in GitHub: https://github.com/gopkumr/BlazorTourOfHeroes

![VS Blazor project](/blogimages/blazor1.png)

The key points the Tour of Heroes project tries to cover is what we will touch up on below

**components**  
Components or pages in the web app is represented by the razor pages in Blazor and they are the .razor file in the pages folder (pretty easy ah!). So in the Blazor project we create a .razor file for each component or page in the app. So for our project they would be the

**App.Razor**  
As in Angular App page is our initial component that loads all the rest of the components. But unlike Angular the default razor app component defines the router setup, that can define the layout page (or the master page) and the content to show when either a URL is not found or if you need to define content for an unauthorized error.
Each routable components in Blazor is defined with the @page attribute at the beginning followed by the URL to which the router should render the component. The default or the index component is defined by the URL "/" (or based on the default route you configure for your web app). There are option to define route attributes and route attribute constraints and navigating from a code block, HeroDetails component shows an introduction to route attribute and navigation.

Read the [documentation here](https://docs.microsoft.com/en-us/aspnet/core/blazor/routing?view=aspnetcore-3.1) for more on routing

Each component has the HTML mark-up along with the code section defined under @code{} block. You have an option to have the code in a separate file by creating a partial class of the same name as the component or a base class for the component, if that is your thing , but I prefer keeping all my markup and code in one place but as an example i have added Dashboard component with a base class approach.

Parameters that a component want to accept from the calling page is defines as C# properties with a attribute decorator called [Parameter]. And the calling page can pass the parameter as an attribute.

Read the [documentation here](https://docs.microsoft.com/en-us/aspnet/core/blazor/components?view=aspnetcore-3.1) for more on components

**dependency Injection**  
Blazor uses the same dependency inject as in ASPNet core. You add the interface and the implementation in the startup class configure services method into the service collection, also define the lifetime of the instance. To inject the dependency into the components, you use the @inject attribute followed by the type and the variable name.

**javaScript interop**  
Blazor lets you call JavaScript functions using JavaScript interop. This by injecting a predefined service called IJSRuntime and using its Invoke methods to call any JavaScript methods in scope. As an example, I have implemented the "GoBack" function in the HeroDetails component using the JavaScript interop.

**data binding**  
Data binding in Blazor is as simple as using the @bind atrribute for the HTML element. For example @bind in an input type text field maps to the value attribute and the variable is updated when the textbox looses focus. You can specify the attribute name by adding the attribute name to end of bind attribute separated by a hyphen, @bind-value. To change the even at which the data binding happens, you can specify the event in another bind attribute like this @bind-value:event="<event in which the change should happen>". You can use the same syntax to bind value to child components.

To show a value in template elements like Div or Label, it is as simple are using the @Variable name. E.g. <div>@Name<div> to show name variable in a div element.

Read the [documentation here](https://docs.microsoft.com/en-us/aspnet/core/blazor/data-binding?view=aspnetcore-3.1) for more on data binding

## conclusion  
This is just an introduction and there is much more in Blazor like State Management, Security, Error Handling , Calling a server API and the exciting Blazor server, which will let you execute the app at the server and update the UI via SignalR.

So there is more coming, watch this space. All my code is in here: https://github.com/gopkumr/BlazorTourOfHeroes