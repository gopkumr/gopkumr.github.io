---
title: "Clean your code using Code cleanup in Visual Studio 2019"
date: 2016-09-16T14:22:48+11:00
draft: false
tags: ["developer", ".Net", "visual studio", "features"]
---

All we developers would have spent time in cleaning up the code after we are done with a long day of code and coffee! for .Net developers it is to do with removing all the using clause added automatically by visual studio/nuget that you no longer need, removing variables that was not put to use, adding read only to eligible private variables, adding or removing braces from single statement blocks etc etc.

There are a list of tools/extensions that helped developers to automate most of these tasks. Popular of these tools were Re-Sharper, StyleCop, CodeMaid...

Microsoft with its visual studio versions starting 2012 incorporated many of these feature into the IDE. Up until Visual studio 2017, developers were able to remove unused using, sort using clauses and other cleanups by right-clicking the file and selecting its option or choosing refactoring quick action bulb.

**Executing Code cleanup from visual studio**

Visual Studio 2019 has taken this to the next level by introducing an option to run a predefined set of code clean up actions on an individual file or on all files in a project/solution and this option is available in visual studio status bar, Analyse Menu or on your right-click context menu of file/project/solution.

![Code cleanup option](/blogimages/cleanup1.png)

**Code cleanup profiles**

As you can see from the above screenshot visual studio 2019 allows you to create a list of code clean up activities (which they call fixers) and save them as what they call a profile (no, you cannot define a name for the profile and there is currently option for only 2 profiles). You can choose from a list of predefined fixers (currently 14 of them are present). To configure profile, you can choose the option from any of the menu option in the above screenshot or you can choose Visual Studio menu *Analyse > Code cleanup > Configure code cleanup*
and you get a window like below to choose the fixers into profiles.

![Code cleanup configuration](/blogimages/cleanup2.png)

It is just the features first release and it something every developer will appreciate and it has a potential to do much more. Visual studio developer community has already started pouring in suggestions and feedback on the feature to make it better.