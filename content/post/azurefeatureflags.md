---
title: "Implementing Feature flags using azure "
date: 2021-05-16T10:00:48+11:00
draft: false
tags: ["developer", ".Net", "visual studio", "feature flags", "azure"]
---

Feature flag is a very popular practice in modern application development, which is used to specifically hide features implemented that are not yet ready to be used by wider audience, and when ready can be enabled by a flip of a switch. The flags can also be used as a kill switch for application feature when it not working as expected.

With feature flags implemented, it would be effective to have the features enabled or disabled from a location outside of the application infrastructure or configuration, this way we can have features spanning across applications be controlled via a centralized flag. Azure has feature management as part of the Azure App configuration service which can manage feature flags and maintain it separate from your hosting model and will act as a centralized repository for feature flags. Microsoft also provides libraries for different programing languages to consume Azure App Configuration service. More about it can be [found here](https://docs.microsoft.com/en-us/azure/azure-app-configuration/manage-feature-flags)

## Creating Azure App Configuration ##
To start with, create the App Configuration service in Azure. For trying out the service, I will use the free tier.
![Create service](/blogimages/appconfig/createappconfigazure.png)

Once the service is created, In the left panel, under operations section, we can navigate to the Feature Manager screen and create a flag. The flag creation is as simple as giving a name to it. The screen also gives an option to create the flag as a FeatureFilter, which is used when the flag is not just a boolean value and you want to control the flag based on other criteria like a subset of users or switching a flag on/off for a time frame. We will keep the details on FeatureFilter for another post.
![Create flag](/blogimages/appconfig/featueflagcreationazure.png)

Once the flag is created, it can be toggled from the feature manager screen.
![Modify flag](/blogimages/appconfig/featuremanagerazure.png)

Note the connection string to the Azure AppConfiguration service, which we will use in our app to connect.
![Connection string](/blogimages/appconfig/connectionstring.png)

## Consuming Azure App Configuration in a WebApp ##
I am implementing a simple web application using ASP.Net Core MVC default app to consume the app configuration service. Adding a view and a controller for the new feature called Feature1. The feature for now will only have a single index page which displays the feature name. Add the feature link to the navigation menu too. While the Microsoft FeatureManagement libraries in aspnetcore can work with flags stored in application's local config files or a database, here I am trying to have it work with the azure app configuration service.

To consume azure app configuration service we need add the below nuget package 
```
Microsoft.Azure.AppConfiguration.AspNetCore
```
aspnetcore feature management requires the below nuget pacakge
```
Microsoft.FeatureManagement.AspNetCore
```

Add the Azure App Configuration connection string to the ```appsettings.json``` connection strings section with a key, i will use  ```AppConfiguration``` .    
To add the App Configuration from azure into the application, update the program.cs file ``` CreateWebHostBuilder``` configuration. Within the configurewebhostdefaults method, add code to configure custom configuration source using ```ConfigureAppConfiguration``` method.   
Within the method, retrieve the azure app configuration connection string from the ```appsettings.json``` and add the azure app configuration as a configuration source using ```AddAzureAppConfiguration``` method, also enable Feature flags.
![Program.cs changes](/blogimages/appconfig/programcs.png)

Add the feature management to the dependency container using ```AddFeatureManagement``` in the configureservices method in startup.cs.
![Startup.cs changes](/blogimages/appconfig/startupcs.png)

Once the setup is done there are few ways the feature flags can be used in the application code.

### Action filter ###
An action filter ```FeatureGate("<feature name>")``` can be used to prevent the action from getting triggered if the feature name passed to the filter is not enabled in the Azure App Configuration.

![Action Filter usage](/blogimages/appconfig/featuregate.png)

### Feature Manager Service ###
If the requirement is to stop execution of a logic based on the feature flag, then a service of type ```IFeatureManager``` can be injected in the controller and methods like ```IFeatureManager.IsEnabledAsync("<feature name>")``` can be used to check if the feature is enabled.

![Feature manager service usage](/blogimages/appconfig/featureservice.png)

### Navigation Links ###
By adding tag helpers from ```@addTagHelper *, Microsoft.FeatureManagement.AspNetCore```  gives the tag ```<feature name="<feature name>">``` to be used around links or buttons or sections, so that the content of the feature tag is not rendered when the feature specified in the name attribute is not enabled. 

![Html feature tag usage](/blogimages/appconfig/featuretag.png)

Doing the above will let you switch on/off the feature in azure and the application will respond to it by removing the link from navigation as well as responding with a HTTP 404 if user tried to access the link directly. With the above implementation,the app needs to be restarted if the flag changes needs to get reflected, as the azure config is loaded only once when the app is launched. To have the config refreshed when it is changed in azure, we need to register a refresh for the azure configuration. Below is how we will do it.

First, the change to the code
![Refresh key usage](/blogimages/appconfig/refreshkey.png)

Here the application monitors the ```RefreshKey``` for any change every 5 seconds (as specified in the cache expiry timespan), when a change in ```RefreshKey``` is noted all azure configurations are refreshed from the service for all keys (specified by refresh all flag).
The refresh key is created as a key-value in the app configuration explorer in azure app configuration.
![Refresh key usage](/blogimages/appconfig/createrefreshkey.png)

For the refresh to work, we need to add the AzureAppconfiguration middleware by making the below changes in startup.cs
![Add configuration middleware](/blogimages/appconfig/AddAzureAppConfig.png)
![Refresh configuration middleware](/blogimages/appconfig/useazureappconfig.png)

After this is implemented, you can update your feature flag and modify the feature key, which will cause the app to get the updated feature flags. Advantage of using a refresh key is that, one single key change can trigger refresh of all the feature flags configured. Alternatively, you can register refresh for individual keys which can be set to refresh only those keys that have been updated.