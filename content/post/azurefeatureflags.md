---
title: "Implementing Feature flags in azure "
date: 2021-05-16T10:00:48+11:00
draft: false
tags: ["developer", ".Net", "visual studio", "feature flags", "azure"]
---

Feature flag is a very popular practice in modern application development, where features in the application are hidden using a flag (as the name suggests) and when required the feature can be enabled for the end users with a flip of a switch. The flags can also be used as a kill switch for application feature when it not working as expected.

With features wrapped within feature flags in the application code, it would be ineffective  to have the application deployed each time a feature need to be enabled or disabled. Azure provides a feature management service as part of the Azure App configuration to manage feature flags in a centralised repository external to the application. And use the App Configuration libraries to consume this service in any programing language.


To start with, create the App Configuration service in Azure. For trying out the service, I will use the free tier.
![Create service](/blogimages/appconfig/createappconfigazure.png)

Once the service is created, In the left panel, under operations section, we can navigate to the Feature Manager screen and create a flag. The flag creation is as simple as giving a name to it. The screen also gives an option to create the flag as a FeatureFilter, which is used when the flag is not just a boolean value and you want to control the flag based on other criteria like a subset of users or switching a flag on/off for a time frame. We will keep the details on FeatureFilter for another post.
![Create flag](/blogimages/appconfig/featueflagcreationazure.png)

Once the flag is created, it can be toggled from the feature manager screen.
![Modify flag](/blogimages/appconfig/featuremanagerazure.png)

Note the connection string to the Azure AppConfiguration service, which we will use in our app to connect.
![Connection string](/blogimages/appconfig/connectionstring.png)

I am implementing a simple application using ASP.Net Core to give Azure App Configuration a try and using the default aspnetcore MVC app. Adding a view and a controller for the new feature called Feature1. The feature for now will only have a single index page which displays the feature name. Add the feature link to the navigation menu too. While the FeatureManagement libraries in aspnetcore can work with flags stored in application's local config files, but here I am trying to have it work with the azure app configuration service.

To implement feature management with azure there are two nuget packages to be installed
```
Microsoft.Azure.AppConfiguration.AspNetCore
Microsoft.FeatureManagement.AspNetCore
```

Add the Azure App Configuration connection string to the ```appsettings.json``` connection strings section with key ```AppConfiguration``` file.  
To use App Configuration from azure in the application, update the program.cs file ``` CreateWebHostBuilder``` configuration and add azure app configuration. Within the configure webhost defaults method add code to configure app configuration using ```ConfigureAppConfiguration``` method. 
Within the method Retrieve the app configuration connection string from the app settings and add the azure app configuration  using ```AddAzureAppConfiguration``` method, also enable Feature flags.
![Program.cs changes](/blogimages/appconfig/programcs.png)

Add the feature management to the dependency container using ```AddFeatureManagement``` in the configureservices method in startup.cs.
![Startup.cs changes](/blogimages/appconfig/startupcs.png)

Once the setup is done there are few ways the feature flags can be used in the application.

### Action filter ###
An action filter ```FeatureGate("<feature name>")``` can be used as action filter for preventing the action from getting triggered if the feature name passed to the FeatureGate filter is not enabled in the Azure App Configuration.

![Action Filter usage](/blogimages/appconfig/featuregate.png)

### Feature Manager Service ###
If the requirement is to stop execution of a logic based on the feature flag, then a service of type ```IFeatureManager``` can be injected in the controller and methods like IFeatureManager.IsEnabledAsync("<feature name>") can be used to check if the feature is enabled.

![Feature manager service usage](/blogimages/appconfig/featureservice.png)

### Navigation Links ###
By adding the ```@addTagHelper *, Microsoft.FeatureManagement.AspNetCore```, tag helper ```<feature name="<feature name>">``` can be used around links or buttons or sections, so that the content of the feature tag is not rendered when the feature specified in the name attribute is not enabled. 

![Html feature tag usage](/blogimages/appconfig/featuretag.png)

This should let you switch on/off the feature in azure and the application will respond to it by removing the link from navigation as well as responding with a HTTP 404 if user tried to access the link directly. With the above implementation, we need to restart the application for the flag to get reflected as the azure config is loaded only once when the app is launched, but with some changes and middleware this can be changed. 