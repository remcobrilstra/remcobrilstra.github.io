---
layout: post
title:  "Securing Azure webapps with Lets Encrypt using ARM"
summary: "Using ARM templates to add Lets Encrypt SSL certificates to your website"
author: remcobrilstra
date: '2019-02-18 1:35:23 +0530'
category: Azure Cloud
thumbnail: /assets/img/posts/code.jpg
keywords: ARM, Azure WebApps, Let's Encrypt
usemathjax: false
permalink: /blog/secure-azure-webapp-with-letsencrypt-using-arm/
---


Let's Encrypt offers a free and easy way to get SSL certificates for your website, in this blog post we'll look at settings this up for an Azure App Service using ARM.

Just a note before we start: This is by no means something new, but it has been very useful for me over time so i thought i might be nice to share :)

## About Let's Encrypt
You may have heard of Let's Encrypt. It's a certificate authority that provides SSL certificates completely free! This is done with the goal of securing the internet and is backed by some major players in the tech industry.
For more information about Let's Encrypt have a look at their <a href="https://letsencrypt.org/" target="_blank">website</a>

In a time when providing your website over SSL an effectively a defacto standard, this is very welcome.

There are some things to take notice of when using certificates from Let's Encrypt. The most important thing is that the certificates they provide are only valid for a short period (90 days) [<a href="https://letsencrypt.org/2015/11/09/why-90-days.html" target="_blank">I leave it to them to explain why</a>].

As it is not very workable to manually refresh your certificates every time they expire, Let's Encrypt promotes an automated way of refreshing them. 
The does require some logic to get working which some hosting providers actually handle for you. 

## Let's Encrypt on Azure

In order to user Let's Encrypt certificates on an Azure App Service, there is a very nice site extension that implements all the refreshing logic for you (do note that this extension is not provided by Let's encrypt themselves). However this does require some manual work to set up. Therefor we will look at how to set this all up using ARM-templates, so we never have to think about this stuff again.

> If you are interested in setting Let's Encrypt up 'by hand', Scott Hanselmann wrote a blog post about this a while ago that you might find interesting. Check it out <a href="https://www.hanselman.com/blog/SecuringAnAzureAppServiceWebsiteUnderSSLInMinutesWithLetsEncrypt.aspx" target="_blank">here</a>

## Getting started with ARM
Now we will be using the before mentioned Site extension, however we will initialize it using ARM. This allows us all the advantages of ARM and an easy way to get our site up and running securely.

I do assume some knowledge of ARM templates, if however something is unclear please hit me up in the comments and I'll be happy to elaborate.

## Gathering some data
To get the site extension to work we need to fill out some parameters during creation of the resource.
As the Site Extension needs to be able to replace the certificate used by the website as well as keep track of some data, it needs a Service Principle to gain access to the Azure resources.

To get started go to <a href="https://portal.azure.com">portal.azure.com</a>, find your favorite directory and find *'Azure Active Directory'* in the Left-hand menu.
Then click *'App registrations'* as shown below.
![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-AD-App-Registrations.jpg)

In the following View click *'+ New application registration'*

In the following screen fill in the data requested, make the *name* something descriptive so it will make sense that this user has certain permissions.
The *'Sign-on URL'* is required but not used in this use case, so any valid web address will do.
![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-AD-create-app-reg.jpg)

Click *Create* when you are done.

On the next screen grab the GUID that is displayed under *'Application ID'* this is our **ClientID**
![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-AD-create-app-reg-2.jpg)

Now click settings and then Keys.

In the following screen Fill in a descriptive name for the key and click save
![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-AD-create-app-reg-3.jpg)

Make sure you get the key that is displayed after clicking save, as it is only displayed once.
This key will be our **ClientSecret**

Now go back to the *Azure Active Directory* blade and click *properties*
![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-AD-tenantid.jpg)

Get the GUID that is displayed under *Directory ID* this is our **TenantID**

We have now retrieved all the information we need, to finish up we need to set some permissions for our freshly created service principle.
To do this go to the resource group that will contain your website (or create one if you don't have it yet).

Click *Access control (IAM)* then *Add* and *Add role assignment*
![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-RG-assign-role.jpg)

In the following screen Select the *Contributor* role and find the service priniciple we just created.

![](/assets/img/posts/secure-azure-webapp-with-letsencrypt-using-arm/Azure-RG-assign-role-2.jpg)
Select the Service Principle and hit *Save*.

We're now done for the azure part, just one more piece of preparation.

## Pointing your dns the right way
To assign a custom hostname to a webapp within Azure, you need to make sure that the chosen url is pointing (CNAME) to the default azure url of the new website. Azure validates this and will only assign the hostname if the dns entry is found (this te prevent abuse). There are two ways to pass the Azure dns validation which are the following :

Direct DNS entry, you create a CNAME entry for your new hostname and point it to directly to the new resource url.

*test  CNAME  {sitename}.azurewebsites.net.*

In this case i would be connecting ie. test.remcobrilstra.com to the new web app.


A DNS Validation entry, this option creates a separate DNS entry that CNAME's from a derived domain to a derived domain.
This entry will solely be used to validate your ownership of the domain, but will not route any traffic over url you will be using, this is especially useful when your are migrating a website that is already receiving live traffic.
The entry would look something like this:

*awverify.test CNAME awverify.{sitename}.azurewebsites.net.*

The above entry would also validate the test.remcobrilstra.com connection as far as azure is concerned. Once you go live with this site, you will still have to add/update first entry to make sure people will actually end up on the new site.

## The resources

So above we prepared some resources to get our ARM on, now you might be wondering what are we doing ARM?

We will be provisioning a few resources that are required to properly run the Let's Encrypt extension, these are:

- **App service plan**<br/>
  As you may know a web app is hosted in a App service plan, you are free to add multiple Web App to it if you want (these can also use the Let's encrypt functionality as long as they have the required parameters set). An important note about the App service plan is that we'll be hosting it in the 'S1' size, anything bigger than this will also do, but we need to SSL and custom domain features provided from this size and up.

- **Storage Account**<br/>
  We as creating a storage account for the Let's Encrypt extension, this storage account is used to store the certificate as well as some information the scheduled cert update task uses to function properly.

- **Web App**<br/>
  The web app we'll be binding the custom domain and SSL certificate to need to be created as well.
  This resource comes with some additional (child) resources to make sure the site extension works properly

  - **App settings**<br/>
    This resource contains the app/environment settings that will be passed to the web app, these settings contain all the configuration the extension requires.
  - **Web**<br/>
    This resource allows us to set some generic properties on the web app, in this case we're using it to set the 'always on' setting to true.
    By doing so we'll make sure the website doesn't go to sleep in times of inactivity, if this where to happen at the moment the certificate needs refreshing the job will fail and your site will be left with an invalid SSL certificate.
    Although the damage is limited, once it expires the first request to the site will wake it up and get a security error, due to the fact the site woke up the SSL certificate will be updated pretty much immediately. Buy you will still have had a visitor that ran into that error, which isn't nice. So we'll just add it to be safe.
  - **Site extension**<br/>
    The actual site extension is denoted as resource here as well, although important the configuration is all passed through the app settings so it doesn't really contain anything noteworthy.
- **HostBinding**<br/>
  This resource contains the custom hostname that will be bound to the web app.

## The parameters
The ARM template that I use can be found below, but before you start using the template it might be useful to shortly run through the parameters.
I'll only focus on the ones that you really need to provide yourself, feel free to dive in a bit more if you want to start using this.

**customHostname**<br/>
This one is pretty self explanatory, it contains the hostname that you want your web app to start using.
This should be set to the same value as you used to set up the DNS entry.


**LetsEncrypt_tenant**<br/>
This parameter contains the **TendantId** that we retrieved earlier

**LetsEncrypt_clientId**<br/>
This parameter contains the **clientID** that we retrieved earlier

**LetsEncrypt_clientSecret**<br/>
This parameter contains the **clientSecret** that we retrieved earlier

**LetsEncrypt_email**<br/>
Lets Encrypt requires an e-mail address when requesting a certificate, this e-mail address will be used to inform you if your certificates are expiring.
This will only occur if the refresh logic of the extension somehow doesn't work correctly. If however you are not interested in these messages, Let's Encrypt also allows you to unsubscribe from these messages.
Your Unsubscribe will however only last for a year after that you will have to re-do it.
I'd advise against it though as these messages only pop-up when something goes wrong with the refresh and you probably want to know that.

## Some actual ARM

Below you'll find a functioning ARM template that will allow you to deploy a website wit the Let's Encrypt integration installed.

I tried to keep the template as minimal as possible to avoid confusion, for production use I'd probably set a bit more properties on the web app.


```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "skuName": {
      "type": "string",
      "defaultValue": "S1",
      "allowedValues": [
        "S1",
        "S2",
        "S3",
        "P1",
        "P2",
        "P3",
        "P4"
      ],
      "metadata": {
        "description": "Describes plan's pricing tier and instance size. Check details at https://azure.microsoft.com/en-us/pricing/details/app-service/"
      }
    },
    "skuCapacity": {
      "type": "int",
      "defaultValue": 1,
      "minValue": 1,
      "metadata": {
        "description": "Describes plan's instance count"
      }
    },
    "customHostname": {
      "type": "string",
      "defaultValue": ""
    },
    "letsecryptstorageType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_ZRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS"
      ]
    },
    "LetsEncrypt_tenant": {
      "type": "string"
    },
    "LetsEncrypt_clientId": {
      "type": "string"
    },
    "LetsEncrypt_clientSecret": { "type": "string" },
    "LetsEncrypt_email": { "type": "string" },
    "LetsEncrypt_authenticationEndpoint": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "The authenticaion endpoint/azure active directory authority used in the selected azure region, when unset it defaults to https://login.windows.net/"
      }
    },
    "LetsEncrypt_tokenAudience": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "The token audience/resourceId used in the selected azure region, when unset it defaults to https://management.core.windows.net/"
      }
    },
    "LetsEncrypt_managementEndpoint": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "The Azure ARM management endpoint used in the selected azure region, when unset it defaults to https://management.azure.com"
      }
    },
    "LetsEncrypt_webSitesDefaultDomainName": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "The default azure web app domain name used in the selected azure region, when unset it defaults to azurewebsites.net"
      }
    }
  },
  "variables": {
    "appServicePlan": "[concat('my-appserv-plan-', uniqueString(resourceGroup().id))]",
    "webSiteName": "[concat('my-website-', uniqueString(resourceGroup().id))]",
    "letsecryptstorageName": "[concat('myencrstore', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "apiVersion": "2015-08-01",
      "name": "[variables('appServicePlan')]",
      "type": "Microsoft.Web/serverfarms",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayName": "HostingPlan"
      },
      "sku": {
        "name": "[parameters('skuName')]",
        "capacity": "[parameters('skuCapacity')]"
      },
      "properties": {
        "name": "[variables('appServicePlan')]"
      }
    },
    {
      "apiVersion": "2015-08-01",
      "name": "[variables('webSiteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverFarms/', variables('appServicePlan'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', variables('appServicePlan'))]": "empty",
        "displayName": "Website"
      },
      "properties": {
        "name": "[variables('webSiteName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlan'))]",
        "phpVersion": "",
        "pythonVersion": "",
        "nodeVersion": "",
        "use32BitWorkerProcess": false,
        "webSocketsEnabled": false,
        "alwaysOn": true,
        "http20Enabled": true,
        "minTlsVersion": "1.2"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "type": "config",
          "name": "connectionstrings",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites/', variables('webSiteName'))]"
          ],
          "properties": {
            "AzureWebJobsStorage": {
              "value": "[Concat('DefaultEndpointsProtocol=https;AccountName=',variables('letsecryptstorageName'),';AccountKey=',listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('letsecryptstorageName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value)]",
              "type": "Custom"
            },
            "AzureWebJobsDashboard": {
              "value": "[Concat('DefaultEndpointsProtocol=https;AccountName=',variables('letsecryptstorageName'),';AccountKey=',listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('letsecryptstorageName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value)]",
              "type": "Custom"
            }

          }
        },
        {
          "name": "web",
          "type": "config",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites', variables('webSiteName'))]"
          ],
          "tags": {
            "displayName": "web"
          },
          "properties": {
            "alwaysOn": true
          }
        },
        {
          "name": "appsettings",
          "type": "config",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites', variables('webSiteName'))]",
            "[resourceId('Microsoft.Storage/storageAccounts', variables('letsecryptstorageName'))]"
          ],
          "tags": {
            "displayName": "appsettings"
          },
          "properties": {
            "letsencrypt:Hostnames": "[parameters('customHostname')]",
            "letsencrypt:Email": "[parameters('LetsEncrypt_email')]",
            "letsencrypt:AcmeBaseUri": "https://acme-v01.api.letsencrypt.org/",
            "letsencrypt:SubscriptionId": "[subscription().subscriptionId]",
            "letsencrypt:Tenant": "[parameters('LetsEncrypt_tenant')]",
            "letsencrypt:ClientId": "[parameters('LetsEncrypt_clientId')]",
            "letsencrypt:ClientSecret": "[parameters('LetsEncrypt_clientSecret')]",
            "letsencrypt:ResourceGroupName": "[resourceGroup().name]",
            "letsencrypt:UseIPBasedSSL": "false",
            "letsencrypt:AzureAuthenticationEndpoint": "[parameters('LetsEncrypt_authenticationEndpoint')]",
            "letsencrypt:AzureTokenAudience": "[parameters('LetsEncrypt_tokenAudience')]",
            "letsencrypt:AzureManagementEndpoint": "[parameters('LetsEncrypt_managementEndpoint')]",
            "letsencrypt:AzureDefaultWebSiteDomainName": "[parameters('LetsEncrypt_webSitesDefaultDomainName')]"
          }
        },
        {
          "apiVersion": "2015-08-01",
          "name": "letsencrypt",
          "type": "siteextensions",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites', variables('webSiteName'))]",
            "[resourceId('Microsoft.Web/Sites/config', variables('webSiteName'), 'appsettings')]",
            "[resourceId('Microsoft.Web/Sites/config', variables('webSiteName'), 'connectionstrings')]"
          ],
        }
      ]
    },
    {
      "type": "Microsoft.Web/sites/hostnameBindings",
      "name": "[concat(variables('webSiteName'), '/', parameters('customHostname'))]",
      "apiVersion": "2016-03-01",
      "location": "[resourceGroup().location]",
      "properties": {
      },
      "dependsOn": [
        "[concat('Microsoft.Web/sites/',variables('webSiteName'))]"
      ]
    },
    {
      "name": "[variables('letsecryptstorageName')]",
      "type": "Microsoft.Storage/storageAccounts",
      "location": "[resourceGroup().location]",
      "apiVersion": "2016-01-01",
      "sku": {
        "name": "[parameters('letsecryptstorageType')]"
      },
      "dependsOn": [],
      "tags": {
        "displayName": "letsecryptstorage"
      },
      "kind": "Storage"
    }
  ]
}
```

This should get you started with the ARM template, i've been using a template based on this for everything i deploy.
It's just a breeze and really nice to no longer have to worry about certificates on any of my websites.


If you have any comments let me know!
