---
layout: post
title: "Using AppVeyor build with ASP.NET Core"
teaser: "description"
author: "Jürgen Gutsch"
comments: true
image: /img/cardlogo-dark.png
tags: - ASP.NET Core
- BitBucket
- AppVeyor 
- DevOps
- Continuous Integration
- Build
---

In the last post I wrote about BitBucket Pipelines and did a small comparison to AppVeyor. Not all of your are using BitBucket to host your source code repository. I think the most of you are using GitHub and unfortunately, there is nothing like Pipelines built in into GitHub, but there are many different third party services which are working great with GitHub. One of them is AppVeyor, which I currently use in some of my projects. 

AppVeyor is more complex than Pipelines, especially if you configure the build using the Appveyor UI, but it can definitely do a lot more then Pipelines. At the end the AppVeyor UI just creates an Appvayor configuration file called appveyor.yml, which will also stored in the projects root directory.

Let's see how AppVeyor works using the same ASP.NET Core project as in the last post. As in that post, the goal is to get all branches and the pull requests built with AppVeyor.

## Creating an AppVeyor account

Because AppVeyor is a third party service, you need to create a separate account first. But this is easily done using your GitHub account. Just follow the Login steps and give AppVeyor access to your GitHub account and your repositories. This is needed because, the service will create web hooks, to get triggered after a SCM change, it will commit changes in the appveyor.yml and it is able to create tags and releases within your repository. That means you need to trust that service. Anyway, I think AppVeyor is trustworthy enough for open source projects.

## Set-up the build

Once the account is created you are able to create the first build within the AppVeyor portal.











## Let's try it

After that config was saved, the build will start after the next SCM change







## Let's try the PR build

To create a PR, I need to create a feature branch first. I created it locally using the name "feature/build-test" and pushed that branch to the origin. You now can see that this branch got built by AppVeyor:



[!TODO IMAGE]



Now let's create the PR using the GitHub web UI. It automatically assigns my latest feature branch and the main branch, which is develop in my case:

[!TODO IMAGE]

Here we see that both branches are successfully built and tested previously. After pressing save we see the build state in the PRs overview:

[!TODO IMAGE]









## The Continuous Deployment pipeline

With this I'm save to trigger a direct deployment on every push to the main branches. As you maybe know, it is super simple to deploy a web application to an Azure web app, by connecting it directly to any Git repository. Usually this is pretty dangerous, if you don't have any builds and tests before you deploy the code. But in this case, we are sure the PRs and the branches are building and testing successfully.

We just need to ensure that the deployment is only be triggered, if the build is successfully done.

To do that, I created a new Web App on Azure and connect this app to the Git repository on GitHub. I'll now add a failing test and commit it to the Git repository. What now should happen is, that the build starts before the code gets pushed to Azure and the failing build should disable the push to Azure.

I'm skeptical whether this is working or not. Let's see.

The Azure Web App is created and running on http://build-with-github-appveyor.azurewebsites.net/. The Deployment settings are configured like this:

[!TODO IMAGE]

It is configured to listen on changes on the develop branch. That means, every time we push changes to that branch, the deployment to Azure will start.

I'll now create a new feature branch called "feature/failing-test" and push it to GitHub. I don't follow the same steps as described in the previous section about the PRs, to keep the test simple. I merge the feature branch directly and without an PR to develop and push all the changes to GitHub. Yes, I'm still a rebel... ;-)

The build starts immediately and fails, but what about the deployment? Let's have a look at the deployments on Azure. We should only see the initial successful deployment.

[!TODO TEST]

## Conclusion

