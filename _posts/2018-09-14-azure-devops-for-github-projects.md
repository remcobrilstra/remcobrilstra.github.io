---
layout: post
title:  "Azure Devops for Github Projects"
summary: ""
author: remcobrilstra
date: '2018-09-14 1:35:23 +0530'
category: Azure DevOps
thumbnail: /assets/img/posts/code.jpg
keywords: 
usemathjax: false
permalink: /blog/azure-devops-for-github-projects/
---

Microsoft announced the rebranding of VSTS to Azure DevOps and with it an offering for OSS projects and better GitHub integration, making Azure DevOps a very interesting option for OSS projects. In this post I’ll go detail the process of getting started with Azure DevOps CI for a GitHub project.

## VSTS is dead, long life Azure DevOps
Earlier this week Microsoft announced the rebrand of VSTS (the application formerly known as Visual Studio Online) to Azure DevOps. I won’t go into details of this rebrand at this point, but I do want to focus on a particular part of the announcement, which is the extended integration into GitHub. This is especially interesting a Microsoft also announced that Azure DevOps will proved free Pipelines (CI/CD) for open source project from now on.

## The new Free
VSTS also had a free offering (which still exists). However this wasn’t focused on OSS and limited the amount of minutes a project can utilize on agents hosted by Microsoft. As for the OSS variant now being offered this limit has been lifted allowing unlimited builds/deployments to be executed. Only limiting the amount of parallel jobs being executed on their clusters which shouldn’t be to much of a limitation in general.

## So what’s the plan?
To get started we need a project, for this post I’ll be using a default DotNet core web application, however in the words of Donovan Brown, Azure DevOps can be used for _“Any Language, Any Platform”_. Whatever you got cooking, chances are you are able to build and deploy it with out of the box functionality in Azure DevOps.


For the purposes of this blog post I’ll go through the process of setting up a GitHub project/repo as well as a DotNet core application and pushing to latter to the former. If this is clear cut for you just skip to the _‘Doing the Azure DevOps thing’_ section.


**NOTE:** I ran into some things that are currently still ‘sub-optimal’ which I elaborate on in the section _‘It ain’t all sunshine and unicorns’_ if you indent to run through this guide for more then just fun and games I’d advice to read that before you continue te prevent you running into any issues.

## Setting up a Simple project on GitHub
As I  stated I’ll be starting from scratch today, although you should be able to get start with any existing project you have laying around just as well. So to get started lets get a project up and running.


## Creating a Simple DotNet core project
For this exercise I’ll be creating an ‘ASP.NET Core Web Application’ using Visual Studio.

![](/assets/img/posts/azure-devops-for-github-projects/VisualStudio_Project_Create.png)

For the further process it doesn’t really matter what options you choose in the next screen so just pick whatever you like.


After your project is created you will have a very basic web application that won’t do much but should suffice for our purposes.


Now we’ll move over to GitHub to get started.


## Creating your GitHub project
At this point I assume you have a GitHub account if that is not the case simply Create a new one to continue.


Once you login you’ll  see a ‘Create a Repository’ button, once you click that you’ll be prompted to the following screen.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_Repo_Create.png)

This screen allows you name your new repository as well as populate it with some default ignore file and a license, for now we will keep it empty and continue to create the repository.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_Repo_Empty.png)

CONGRATULATIONS!  You have just created your very own Git repository!


## Pushing your code GitHub
In order to get our code into the newly created repo, copy the *.git url displayed on this screen and find your way back to Visual Studio.

![](/assets/img/posts/azure-devops-for-github-projects/VisualStudio_SourceControl_Add.png)

On the bottom right of your screen you will find a ‘Add to Source Control’ button, expanding that will show a ‘Git’ button. Clicking this will create a local git repo in the folder where you created your project, this will place the .SLN in the root of your repository. If you prefer this to be different this would be the time to make some organizational changes


After this is done the following the screen will be shown (click the ‘Publish Git Repo’ to see the textbox). In this text box copy the url of your GitHub repo an click publish.

![](/assets/img/posts/azure-devops-for-github-projects/VisualStudio_Git_Publish.png)

You will be prompted to login to your GitHub account. Once you do so a access token will be generated on your github account and the code will be pushed to the repository (**note:** as this creates an access token, you will receive an e-mail from github about this activity on your account. Fear not it was you who did that )


As I already stated your code will now be pushed to GitHub, once this is completed go back to your browser and refresh to page. At this point you will see a nicely populated repository with your code in it.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_Repo_Filled.png)

Congrats you are now officially on the GitHubs, now let’s start doing what we came here for.


## Doing the Azure DevOps thing

Now although the entire suite of tools is Called Azure DevOps, all the separate tools have been rebranded to reflect what they do (and possibly to be able to market them better) therefor what used to be Build&Release in VSTS is now called Azure Pipelines. Which therefor is what this integration will refer to from now on.


In the Top menu click ‘Marketplace’ to enter the GitHub integration marketplace, once there search for ‘Azure Pipelines’.


Now click the ‘Azure pipelines’ entry, to get to the details page for this Marketplace offering. On the bottom of the page you will be able to install this integration into your repository.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_AzurePipeline_Install.png)

Click install and confirm your order to get to the next stage of this process.


After doing this you will be prompted to provide Azure Pipelines with access to your repository. At this point you may opt to provide Azure Pipelines with access to all your repositories or just to the one you just created.


**Note:** providing access to all repositories will also include any repositories you may create in the future.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_AzurePipeline_Permissions.png)

During this install process your are allowed to select from a list of Azure DevOps accounts you may already have, to create a new one. If however you don’t have any existing Azure DevOps accounts you may run into the same snack as I did which I describe in the last section. It might be wise to read that **before you continue**.


If all is well you can click install run through the selection and you’ll be send to a wizard for Azure DevOps creation, at this point you will need to login with a Live account which I’ll presume you have.


After either selecting or creating a Azure DevOps account the installer will move into Azure DevOps/Pipelines.


## Configuring your Azure Pipeline

This will throw you into the new YAML based pipeline wizard, for those of you already familiar with Azure DevOps you might prefer the visual editor which you will have to go to by clicking build in the side-menu an creating a new definition, You are able to work around that but for now we’ll continue with the YAML experience.

![](/assets/img/posts/azure-devops-for-github-projects/AzureDevOps_Build_Create.png)

Once you select your repository you will see Azure DevOps doing some analysis on your repo, this analysis attempts to identify the type of project you have checked-in. This allows Azure Pipelines to give a few suggestions for standard templates that could build your code. You are however always able to select an empty template, or by selecting  ‘All templates’ browse all the other templates available.


As I created a straight forward DotNet core application the ‘ASP.NET Core’ template will fit my needs best so I’ll select that. If you are doing this with any other type of project select what applies to you.

![](/assets/img/posts/azure-devops-for-github-projects/AzureDevOps_Build_ViewYAML.png)

After selecting the template you’ll be given a preview of the YAML file that will be used which you can change on the spot if required.


For now we’ll keep it as is an click ‘Save and run’

![](/assets/img/posts/azure-devops-for-github-projects/AzureDevOps_Build_SaveAndRun.png)
This will show a dialog allowing you to either check the YAML file directly into the branch you are working in or create a separate branch and adding the file there. I’d advice using the option to create a new branch for reasons I’ll get into below.


Click ‘save and run’ to continue your process

![](/assets/img/posts/azure-devops-for-github-projects/AzureDevOps_Build_Starting.png)

When you continue you will see your build process in action. Initially this may show the screen above while Azure Pipelines is looking for a build agent to process your job. This generally doesn’t take more than a few seconds.


While we are watching our build run, Azure DevOps did some additional work in the background. It created the branch that we instructed it to, but it also created a Pull request from the branch to your master. This pull requests has your newly created CI pipeline configured as a validation check.


When you switch back to GitHub you’ll notice that a pull request has been opened, for the addition of the YAML file.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_PullRequest.png)

When you open the pull request you’ll see that it is actually awaiting a validation build in Azure Pipelines to validate your request. This is especially useful when you are building something a bit more complex than my sample as a failed build will hold this pull request until you have fixed the YAML file to successfully build your application and only then will this new addition be added to your master branch.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_Build_Pending.png)

**NOTE:** This will still be saying its checks haven’t completed after the build you kicked off is completed. This is due to the fact that you kicked that one off manually and the validation build for the PR will be scheduled right after that one.


After a short wait, you’ll see everything light up in a pleasing green.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_Build_Success.png)

And you are free to Merge this PR, and start using your Azure Pipeline in the wild
Clicking the detail of the check will show a detail view with a link to Azure Pipelines for more details on the build (**Note:** there seems to be an issue with public access to that ATM please see the end of this post for details).


Once you merge your PR into master you will see another Azure Pipelines build popping up to validate the changes to Master.


At this point you are done! Yay.


You can now start working on your code base in github and accepting PRs without the fear of breaking the build.


## It ain’t all sunshine and unicorns
While writing this post there where a few things I notices which require some improvement in my opinion. Although that is to be expected with a product that only launched this week it’s still good to take this into account when you start using Azure Pipelines with your GitHub repository.

### Naming of a new Azure DevOps project
When I started this I made it a point to start from scratch (so new live-account new GitHub-account etc.) just to make sure I didn’t run into anything. In general all was oké but there was one weird thing when I went to the ‘installation wizard’. For accounts at I have used in the past (aka already have Azure DevOps accounts on their name). I was prompted to use either an existing account or provide a name for a new one, which I though was a very nice experience.


However when I loggedin with a brand new live account I wasn’t prompted to fill in a name, and I simply created a Azure DevOps account named {firstname}{lastname} and a project with the same name. As this wasn’t the name I wanted to use I changed both account and project name in Azure DevOps afterwards. However this caused the link in the pull request to no longer work as that still pointed to the old name of the account which no longer existed.


So if you are starting anew in Azure DevOps I’d advice to create a new account beforehand to make sure you don’t run into the same issue I did.


Although it seems like something that is pretty obviously not the intended behavior so it might get fixed soon (pretty please Microsoft).

### PR link to your build results
As stated in the post above the PR details show a link to Azure Pipelines that allows you to view the build results/logs in detail. As this is an OSS project everyone is able to see this, which is really useful if you are creating a PR for an project an you want to know how you managed to break the build.

![](/assets/img/posts/azure-devops-for-github-projects/GitHub_PullRequest_Details.png)

This link that get generated however isn’t currently publicly accessible, even though the project itself is.


The link underneath the PR data on GitHub is as follows:


https://dev.azure.com/JustCodeStuff/web/build.aspx?pcguid=18250f89-1973-4218-9539-38b8b2457975&builduri=vstfs:%2f%2f%2fBuild%2fBuild%2f5


Which hits a login prompt ,while the project itself under the following link is available


https://dev.azure.com/JustCodeStuff/JustCodeStuff.DotNetCore.Sample


Even the normal URL to the build details works just fine


[https://dev.azure.com/JustCodeStuff/JustCodeStuff.DotNetCore.Sample/
JustCodeStuff.DotNetCore.Sample%20Team/_build/resultsbuildId=5](https://dev.azure.com/JustCodeStuff/JustCodeStuff.DotNetCore.Sample/JustCodeStuff.DotNetCore.Sample%20Team/_build/resultsbuildId=5)

This is quite annoying but i’d expect this to be fixed soon.


### No visual Editor
As an experienced used of VSTS/Azure DevOps I’m used to working with the Visual Editor for builds and releases. The downside of the visual editor is that it can’t be used when working with YAML files. Although I understand the added value of having the definition of your build in source-control and tend to use that a lot as well I was somewhat disappointed that there was no option from the wizard select the visual editor as an option to get started with your pipeline (@microsoft being able to use the Visual editor for YAML files wouldn’t hurt either).

![](/assets/img/posts/azure-devops-for-github-projects/newpipeline.png)

If you really want to use the visual editor it is still available by clicking Builds in the left menu. Once there click “new +”, this will default to the YAML editor but there is a link to ‘use the visual designer’ in the bottom.

## In conclusion
I really like the ease of integration the guys put together and I am definitely going to use this going forward. Just a few bugfixes/improvements would make it perfect!


If you have any questions of remarks please let me know in the comments!