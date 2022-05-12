---
layout: post
title:  "Configuring App Configuration in Azure with ARM"
summary: ""
author: remcobrilstra
date: '16-05-2022 1:35:23 +0530'
category: Me
thumbnail: /assets/img/posts/code.jpg
keywords: ARM, KeyVault, AppConfiguration, AzureSQL
usemathjax: false
permalink: /blog/configuring-app-configuration-in-azure-with-arm/
---

## the goal
Today we're setting up [Azure App Configuration](https://azure.microsoft.com/en-us/services/app-configuration/), this Azure component provides an easy way to store/maintain configuration in a central place. this way you can access shared configuration from multiple applications in your eco-system. 

What i want to do today is setup an easy and secure way to get started with this resources from an ARM template. our setup will look as follows.

![](/assets/img/posts/2022-05-16-settingup-AzureAppConfig/setup-overview.jpg)
- **Keyvault** here we'll store our sensitive configuration values to passthrough to the App Configuration. this is actually a quit interesting feature as is allows encrypted storage of these values more in this later.
- **Azure SQL** we'll have a database in this setup of which we want to make the connectionstring available
- **App configuration** and the star of the show, we'll add the app configuration to 
- **App Service** the app service would connect to the App configuration and user the related settings.

**note:** we won't setup the App service in this post as i want to focus on setting up the infrastructure and not on coding the application.

## creating the resources
In order to get started we'll add all the required resources to the arm template

---
include basic arm
---


## App configuration and KeyVaults
As stated before we won't be storing our secrets directly in the app configuration, we'll be utilizing the encryption of the Key vault to story them in a secure manner.

App configuration allows you te reference secrets from a Keyvault and expose them as a configuration settings to any apps using it.
In our example we'll be using the connectionstring to a database but secrets can essentially be anything piece of sensitive data you need access to.

An important thing to note is that the permissions on the keyvault propegate through the App configuration.
As a result of this the identity that you use to connect to the App configuration will also require permission on the key vault to access to the underlying values. I guess you could use this this to only provide certain apps access to the secrets in your app configuration, but personally i feel that would just make thing unclear.


## adding the connectionstring
To get started adding the connectionstring, we'll first have to extract it using a line of ARM you probably already encountered
__**CONNECTIONSTRING ARM**__

Now we'll start by writing that connectionstring to the key vault
__**WRITE TO KEYVAULT**__

With the connectionstring safely stored in the key vault we can continue with referencing it from the App configuration
__**WRITE TO APP CONFIG**__

__note:__ do not forget to properly set the dependancies of these step otherwise this wil not execute in the proper order.



## adding some more variables

- Add some more values


## looping in ARM

- cleaning up the above with a loop


