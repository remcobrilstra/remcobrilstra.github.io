---
layout: post
title:  "Microsoft Connect(); 2017 Devops Announcements"
summary: ""
author: remcobrilstra
date: '2017-11-15 1:35:23 +0530'
category: Microsoft Ignite 2017
thumbnail: /assets/img/posts/code.jpg
keywords: 
usemathjax: false
permalink: /blog/ms-connect-2017-devops-announcements/
---

Some real interesting announcements are coming from Connect(); 2017.  This article will focus on some of the interesting DevOps announcement that just got out.


## Azure DevOps Projects

This allows you to create a full CI/CD pipeline from scratch using various sample applications, or just use your own code.

Currently DevOps project support sample project in the following languages:


- .NET Framework
- .NET Core
- Java
- Node.js
- PHP
- Python
- Static Html


If you don’t feel like using the provided sample projects you can just point to your own Git repo and setup the pipeline using that.

![](/assets/img/posts/ms-connect-2017-devops-announcements/azure_devops_projects.png)

Once you work your way through the wizard, you will be able to create a new VSTS account or use an existing one, once you’ve done this you get the following:


- GIT repo (with the code you requested)
- CI Build
- CI Release process


If that’s not enough you will actually end of with a running application using the CI pipeline you just created.


Off course these are pretty simple examples of a CI pipeline, this will however get you started really fast and also allow for a low impact introduction to Continuous Integration.


Make sure to check it out !


https://portal.azure.com/#create/Microsoft.AzureProject


## Pipeline As Code

You thought you were all DevOps because you are using Infrastructure As Code? Think again, Pipeline As Code is here!


VSTS now allows you to define your build Process in a YAML file and version it together with your code.


When creating a new Build simply select that it will be a YAML build and you can select the file from your repo. No need to configure any build logic in VSTS. Nor will you have to update your build definitions in VSTS when your build requirements change with the application.
This will also allow you to create builds of older versions of your application without having to worry about any changes to the build process.


It’s also good to mention that these YAML files are simply a different representation of the existing process, so no new system and more importantly access to all the existing Tasks (incl. Marketplace extensions).


This feature doesn’t seem to be available of on the ring I’m on but it should show up as a preview feature soon, I’d encourage anyone to check it out!


## Release gates

Now we had validation steps between releases for a while, but the possibilities where limited. The newly released ‘Release Gates’ add new level of validating to the mix.


Release Gates, are automated tasks that can be used to validate either if a release should be deployed to an environment or validate if the deployment has executed correctly. Out-of-the-box you will have the following options available:

![](/assets/img/posts/ms-connect-2017-devops-announcements/VSTS_Release_gates.png)

- **Azure Monitoring** <br/>
    This option allows you to check on an Application Insights alerts being triggered, when they do happen after deployment your release will be stalled and eventually will fail if the issue if not resolved.

- **Invoke Azure Function** <br/>
    This will allows you to extend this functionality by calling your very own Azure functions.

- **Invoke REST API** <br/>
    Call a REST end-point to validate you release.
    Publish To Azure Service Bus
    Send a message into the Azure Service bus.

- **Query Work Items** <br/>
    This type will allow you to check if a VSTS work item query returns any result and will stall the build if it does (i.e.. Don’t release while any Bugs are open)


As you can see above there is a lot of possibility to implement you own custom validation process and remove yet another piece of manual labor from your release pipeline. I’m interested in what people will get up with.

 
In short these are some pretty neat features that improve the way we can shape our DevOps process. I know this is only a short overview, but I hope this’ll get you started.