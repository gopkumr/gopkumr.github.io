---
title: "How can I turn my Blazor WebAssembly to PWA?"
date: 2020-06-04T18:52:19+11:00
draft: false
---

![PWAmeetsBlazor](https://dev-to-uploads.s3.amazonaws.com/i/g9e27g1fckgxdipathy2.png)

## Lets get started with an existing Blazor WebAssembly project
I already have a Blazor WebAssembly project created implementing Angular Tour of heros application. You can find the project in my GitHub repository here
Repo: https://github.com/gopkumr/BlazorTourOfHeroes.git
Branch: Release

## Next step is making this into PWA
As with any web application, adding PWA capabilities to Blazor follows the web standard process of adding a manifest json file and the service workers js file.
- Manifest.json file specifies the PWA metadata like name, icon author etc
- Service-Worker.js provides the offline capabilities.
both these files were added to the wwwroot folder and included in the index.html page like below
```
<link href="manifest.json" rel="manifest" />
 <script>navigator.serviceWorker.register('service-worker.js');</script>
```

I have added the files to the project and it can be found in the same GitHub Repo but in a branch called PWA
Repo: https://github.com/gopkumr/BlazorTourOfHeroes.git
Branch: pwa

## A little about the manifest.json file that i have include in the project
Now that we came to the topic of manifest.json below are the properties I chose to add to my manifest file.

- **short_name and name:**
  short_name is used on the user's home screen, app launcher, or desktop. name is used when the app is installed.
- **icons:**
  Set the icons for the browser to use on the home screen, app launcher, desktop etc.
- **start_url:**
The start_url tells the browser where your application should start when it is launched.
- **background_color:**
The background_color property is used on the splash screen when the application is first launched.
- **display:**
Used to customize what browser UI is shown when your app is launched. Below are the possible values 
+ *fullscreen:*	Opens the web application without any browser UI and takes up the entirety of the available display area.
+ *standalone:*	Opens the web app to look and feel like a standalone native app. The app runs in its own window, separate from the browser, and hides standard browser UI elements like the URL bar.
+ *minimal-ui:*	This mode is similar to standalone, but provides the user a minimal set of UI elements for controlling navigation (such as back and reload).
+ *browser:*	A standard browser experience.
- **theme_color:**
The theme_color sets the color of the tool bar.

## Offline capabilities
Adding the service-worker.js gives the app the offline capability that it requires to work like a native app.
When the app is launched the service worker will look for the page/component to be served from the cache, if it was not available in cache then it tries to reach the server for the request to be served. This is the strategy followed irrespective of connectivity being present or not.

Since this logic is baked into service worker js, if any URL in the app is to be always served from the server (like a server rendered page) then edits need to be done into service worker js onFetch method.

For any data that is served from the page will be directed to the server, so an offline capability has to be handled by the app developer either by using a localstorage or an indexDB for storing data at client side or presenting user with no connectivity message.

## Reference links
All PWA concepts like manifest.json, service worker js, local storage, push notification are web standards independent of Blazor. Blazor has brought these webconcepts, webassembly and our beloved C# & Razor together.
- Service Worker: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- Manifest.json: https://developer.mozilla.org/en-US/docs/Web/Manifest
- Blazor: https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor

