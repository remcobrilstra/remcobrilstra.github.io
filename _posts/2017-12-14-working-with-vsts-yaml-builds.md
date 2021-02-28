---
layout: post
title:  "Working With Vsts Yaml Builds"
summary: "Multi Author Support allows to create articles with different Authors"
author: remcobrilstra
date: '2017-12-14 1:35:23 +0530'
category: Azure DevOps
thumbnail: /assets/img/posts/code.jpg
keywords: 
usemathjax: false
permalink: /blog/working-with-vsts-yaml-builds/
---


At the Connect(); 2017, Microsoft announced the addition of YAML based build (which I shortly covered in my connect post). Now I finally had some time to play around with this new feature. So I felt it was time go a bit more in-depth on this new system.

So I’ll start with an introduction into the subject and walk through getting is all up and running.

## Why you want this
If you have some experience with build systems you will be familiar with the situation where changes in an application require changes to the build process. This is a natural effect of development of any application, but will cause some problems, especially if the changes to the build process are not backwards compatible.

In the past (the non-yaml times) this would mean that you would have to create different versions of your build processes for develop and masters branches. In a landscape where you have to be able to build older software versions also, you’d have to create build definition for versions of the application and somehow administrate which build definitions belong to which version of the application. This gets very complicated very fast, not to mention that all these definitions have to be maintained to some extent.

### YAML builds to the rescue
With the new YAML builds you maintain the build process of your software with the code itself. Meaning it gets checked in with the rest of your code and gets versioned as such. The beauty of this solution is that if you need to build a application version from 2 years ago, you are able to start a build of the related revision and VSTS will fetch the build definition from that point in time and execute that.

In the announcement this was referenced to a Pipeline As Code, referring to the similarity to Infrastructure As Code which takes a similar approach to handling Infrastructure. There are many solution for IAC, including but definitely not being limited to Azure Resource Management (ARM).

An added advantage (this is true for IAC as well as PAC) is that this puts the responsibility with the programmer to make sure that his/her changes are compliant with the configuration in the repository.

A slight disadvantage might be that this does provide yet another way to quite literally break the build, but on the upside it also allows you to fix it.

## Some critical notes
Before we start a short ‘getting started’ guide, there are some things that do (in my humblest of opinions) require some improvement.

### Editing
Currently there is no editor available for the YAML files. YAML is of course a pretty simple format, but it isn’t always very clear what you are allowed to do. The long way around the seeing if your edit has the desired effect is committing & pushing your updated file and seeing if VSTS gives you any error and/or you build performs as expected. This is a pretty cumbersome process especially when setting up a new build.

To make this a bit less straining a ‘View YAML’ button was added to the old build definition which allows you to view the YAML for that build definition. This allows you to setup your build in the normal editing experience then export it to YAML.

There is however no way (as far as I know) to easily go from YAML to a editable situation. So say you want to change an existing YAML build you are left to tinker with the file or recreate the YAML build in the editor manually to then apply your change and export it to yaml again. This (especially when your builds get more complex) can be a very labor intensive and error prone operation, so I hope there will be solution of this in the future.

### Just Phases & Steps
It is important to note that the YAML files (currently) only support Phases and Steps. Now there is something to say for this as this is the essence of your build process, however if your build depends on certain variables (something like ‘BuildConfiguration’ or ‘BuildPlatform’) you will have to create them in the VSTS build editor.

I do see that advantage of this as it allows you to create different build based on the same process using different trigger/branches or variables. On the other hand it can also cause issues if variables change during the lifetime of the software. You will have to make sure to keep changes here backwards compatible, as something like a missing variable will _‘break the build’_.

### YAML
This is not so much something that I except to be changed just a general wondering ‘WHY YAML?!’. An explanation for this would be that YAML is used for exactly this purpose in other popular build systems, however I would hope that there is a better argument for this then ‘but the other boys are also doing it’.

As builds can already be freely exported and imported in JSON it would in my opinion have been a logical choice to reuse this format. This reuse would also have removed the editing issues I stated above as the definition would be easily imported for editing. Which leads me to another possible reason for this choice, as a full export contains way more information then the YAML system currently (and possibly will ever) supports, using a different format does reduce the risk of confusion between the two file-types.

## Getting started
Now that we got that out of the way, lets get this show on the road.

### Enable the preview feature
As this functionality is currently still in preview, it will not be enabled by default.  To enable is click your image in the top right corner and click Preview Features. In the panel that shows up make sure to select ‘for this account’ in the drop down as this features can only be enabled for the entire VSTS instance.

Now find the ‘Build YAML definitions flag in turn it on.

![alt text](/assets/img/posts/working-with-vsts-yaml-builds/YamlBuildFlag.png "Enabling the preview feature")

Congratulations, you are ready to get started.

### Create a standard build
Our starting point will be a standard build, for the purposes of this demo I created a ASP.NET Core build and checked in a simple Core web project. But generally any existing build will do.

_For this example I’ll assume that you have an empty project with a core project checked into Git._

So under **Build and Release / Builds**  click the ‘+ New’ button
![alt text](/assets/img/posts/working-with-vsts-yaml-builds/VSTS_build_selectagentqueu.png "Selecting an agent queue")

Now click the view YAML link in the top right corner of the screen

![alt text](/assets/img/posts/working-with-vsts-yaml-builds/VSTS_View_YAML.png "View YAML")

### Creating the YAML build
Now copy the YAML from the shown popup and create a file named **.vsts-ci.yml** in the root of your repository. In general the name of this file can be anything you want, however VSTS monitors for this name and whenever there is no existing YAML build looking for this file any commit on the file will trigger the automatic creation of a new build definition based on that file.

Once you commit and push this file, you will see a build definition appearing in VSTS, as this build includes a trigger your build will also be running now.

Unfortunately the new build will fail ☹

As mentioned before variables are not included in the YAML files, however the standard builds use variables (namely like ‘BuildConfiguration’ and ‘BuildPlatform’) which are now missing from our newly created build.

To get things working switch to the ‘Variables’ tab and add the missing variables.

![alt text](/assets/img/posts/working-with-vsts-yaml-builds/VSTS_build_set_defaultvariables.png "Add default variables")

Click ‘Save & Queue’ and your build will start, and should succeed

Congrats, you just executed your very first YAML based build.

### Another approach
If you want to use a different name and/or create multiple build definitions based on the same YAML file, you can simply use the ‘+ new’ button and select the YAML template.

![alt text](/assets/img/posts/working-with-vsts-yaml-builds/VSTS_build_YAMLdefinition.png "Add default variables")

Under process you will be able to select the YAML file you’d like you use for this build definition.

![alt text](/assets/img/posts/working-with-vsts-yaml-builds/VSTS_build_selectYAMLfile.png "Add default variables")

I hope this gives a clear way to start working with these new builds. if however you are missing anything leave a comment below and I’ll see what I can do.


## Whats next?
As you may know the tasks system that VSTS uses to facilitate build is also used to setup release, however the VSTS team decided to initially only support this feature on the build system. Presumably to give them the time to get this stable before moving further. As stated above there are still some things that might need some TLC before this gets out of preview.
However it is a great idea that this functionality will cover releases in the future, as this will allow for maintaining the entire cycle (Build/Release/Infrastructure) from the codebase.

### Sources
If you want to get into this subject some more, the links below should get in handy.

https://github.com/Microsoft/vsts-agent/blob/master/docs/preview/yamlgettingstarted-scripts.md

https://docs.microsoft.com/en-us/vsts/build-release/actions/build-yaml
