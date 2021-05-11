---
title: "Radio Player using  Blazor 5"
date: 2021-05-11T16:35:28+10:00
draft: false 
tags: ["Microsoft.Net", "Blazor", "Web Assembly", "C#", "HTML", "Javascript"]
---
I have been reading the Blazor 5 documentation and decided to create a simple project to give its features a try. As always, there were a ton of ideas in my mind but while scanning through dev.to i came across a post by [Aleks Popovic, where he made a Radio player using react](https://dev.to/alekswritescode/radio-player-app-in-react-84k), so i decided to create one using Blazor 5. I used the same service as Aleks to get the radio stations, called the [Radio-Browser](https://www.radio-browser.info/).

First step was a to choose a suitable UI which is simple and easy to use as a radio. I borrowed the style of the player from a [codepen.io sample for music player](https://codepen.io/webandapp/full/reVmyN). With the UI design out of the way, it was time to create a component and wire up the code to fetch and play radio stations.

To keep it simple the project currently list a set of predefined genre and fetch stations for a selected genre and display it as a list. The user can choose the station and they listen to it.

## State persistence
The list of genre is contained within its own component called LeftNavMenu. This component is included within the main layout page which renders the player component. The selected genre is maintained by an in-memory state container. The state container is used by both the LeftNavMenu component and the Player component to share the selected genre. When user selects a different genre from the LeftNavMenu the value in the state container is updated and action is triggered to notify the player component of the change. This approach can be used to share state between nested components or independent components.

The state container is configured as singleton instance in the service collection dependency container which is injected in all the Blazor components and used.

## Cascade Values and parameters
The index component is the first component that is loaded and it contains the Radio player component. During initialization of the index component the radio server API is triggered to fetch all the radio stations for the selected genre. The fetched radio stations list is passed onto the Radio player component as a parameter, the first station of the list is passed into the  radio player as a cascade value. The difference between the two is that cascading values can be passed onto to all the components within the CascadeValues section, where as for parameters the values would need to be passed to individual components.

## CSS Isolation
One issue with CSS is bleeding of style, where style applied in one of the component affecting other components rendered in the same page. This was the issue with the genre LeftNavMenu component. As a was to get around this problem, blazor has introduced CSS isolation where you create a css file along with the component file and name the css file as <componentname>.razor.css. The component styles are rewritten during compile time by appending a unique identifier to the css properties as well as to the HTML elements in the component UI.

```
HTML
<li b-3xxtam6d07>

CSS
li[b-3xxtam6d07]{
    color:red;
}
``` 
All the component styles are then bundled and included inside the www\index.html head tag as <ProjectAssemblyName>.styles.css.

These were the 3 of the new features that are used in this project and there are more, there are also other features like JS Interop, event handling used within the project which were part of the initial Blazor.

The source fot the project is available in [github project] (https://github.com/vinca-creative/BlazoRadio.git), feel free to take a look and give suggestion.



 

