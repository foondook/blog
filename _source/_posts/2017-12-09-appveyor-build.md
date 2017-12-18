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

Once the account is created you are able to create the first build within the AppVeyor portal. Logon to ci.appveyor.com and create a new project. Select the SCM provider (GitHub, BitBucket, VSTS, etc.) and the repository you want to build. In my case it is the https://github.com/JuergenGutsch/unittesting-aspnetcore.

![](../img/appveyor-build/new-project.png)

Once the project is created you need edit the settings or use the even more simpler way by creating a `appveyor.yml` file to configure the build. Clicking threw all the settings is nothing more than creating the `appveyor.yml` using a UI. At the end you can either use the settings stored in the UI or the generated config file. 

It makes sense to have a look into the settings to see what's possible in AppVeyor and to learn the configuration. But the amount of options and possibilities is huge. That's what a meant when I wrote about complex configuration in the last post.

At the end the file I created looks like this:

~~~ yaml
version: 1.0.0-build-{build}
pull_requests:
  do_not_increment_build_number: true
skip_tags: true
image: Visual Studio 2017
dotnet_csproj:
  patch: true
  file: '**\*.csproj'
  version: '{version}'
  package_version: '{version}'
  assembly_version: '{version}'
  file_version: '{version}'
  informational_version: '{version}'

install:
- cmd: dotnet restore WebApiDemo.sln
build_script:
- cmd: dotnet build WebApiDemo
test_script:
- cmd: dotnet test WebApiDemo.Tests
~~~

This is pretty much all I need to build a ASP.NET Core application.

* I use a special version pattern here. Because I prefer semantic versioning, the actual build version shouldn't be used in the version number.
* I don't want to increment the build number on pull request builds
* I don't build  on new tags
* I use the VS2017 image, so I have the new SDKs, .NET Core 2.0, etc.
* The dotnet_csproj section (including patch: true) sets the version number to the *.csproj files. This numbers are than used in the DLLs and in NuGet
* The next three sections are the actual build steps
  * On install I restore the packages of the solution.
  * On `build_script` I build the web project. It is also possible to to build the entire solution. To reduce the overall build time, you should than disable the build of the tests section, because `dotnet test` runs the build again.
  * On `test_script` I implicit build and run the tests project. It makes sense to run the test projects explicit or to move all the test projects in a subfolder and call that subfolder. If you call the solution, `dotnet test` will also try to run the regular projects and will write out warnings because it tries to run projects that doesn't contain tests.

That's it. You don't really need more to just build and test your application.

## Let's try it

Save that file in the root of your project folder, commit and push it to GitHub. once the `appveyor.yml` was pushed to the server, the build will start immediately. And it was successfully:

[!TODO IMAGE first successful build]

This was easy to setup. I would recommend to click threw the settings in the AppVeyor UI to learn more about the possibilities.

## Let's try the Pull Request build

To create a PR, I need to create a feature branch first. I created it locally using the name "feature/build-test" and pushed that branch to the origin. 

> Because I use git-flow to work with git, my main branch I work on is `develop`, which contains the next version. `Master` contains the current version and only get's updated if I create a new release. This is why the feature branches are created based on `develop` and will merged back to `develop` than.

You now can see that this branch got also built by AppVeyor:

[!TODO IMAGE built branch]

Now let's create the PR using the GitHub web UI. It automatically assigns my latest feature branch and the main branch, which is develop in my case:

[!TODO IMAGE create a PR]

Here we see that both branches are successfully built and tested previously. After pressing save we see the build state in the PRs overview:

[!TODO IMAGE built PR]

We now see that the PR was build separately. This is the difference to BitBucket Pipelines, where the PR is not built explicitly. This is pretty nice and helps a lot with open source projects, where you get a lot PRs from outside your projects.

After the PR was approved and merged AppVeyor starts again to build the develop branch.

## The Continuous Deployment pipeline

With this I'm save to trigger a direct deployment on every push to the main branches. As you maybe know, it is super simple to deploy a web application to an Azure web app, by connecting it directly to any Git repository. Usually this is pretty dangerous, if you don't have any builds and tests before you deploy the code. But in this case, we are sure the PRs and the branches are building and testing successfully.

We just need to ensure that the deployment is only be triggered, if the build is successfully done.

To do that, I created a new Web App on Azure and connect this app to the Git repository on GitHub. I'll now add a failing test and commit it to the Git repository. What now should happen is, that the build starts before the code gets pushed to Azure and the failing build should disable the push to Azure.

I'm skeptical whether this is working or not. Let's see.

The Azure Web App is created and running on [http://build-with-appveyor.azurewebsites.net/.](http://build-with-appveyor.azurewebsites.net/.) The Deployment settings are configured to listen on changes on the develop branch. That means, every time we push changes to that branch, the deployment to Azure will start.

I'll now create a new feature branch called "feature/failing-test" and push it to GitHub. I don't follow the same steps as described in the previous section about the PRs, to keep this test simple. I merge the feature branch directly and without an PR to develop and push all the changes to GitHub. Yes, I'm still a rebel... ;-)

The build starts immediately and fails, but what about the deployment? Let's have a look at the deployments on Azure. We should only see the initial successful deployment. Unfortunately we have the same issue as on BitBucket Pipelines. The failing build doesn't prevent GitHub to trigger the configured Azure Webhook. This needs us to do the same hack as on BitBucket: To trigger the Azure Webhook manually within the build. Fortunately AppVeyor also has a deployment section, where we can trigger the web hook.

> There is also an option to deploy to Azure without the hook, but I miss the simple web deployment settings, to just push the application to Azure. So the different Azure integrations and the web deployment on AppVeyor are more complex than needed. I like to keep the configuration as simple as possible. Triggering the mentioned web hook and letting Azure do the whole stuff is much more easier in this case.
>
> It is also possible to call the webhook URL using a webhook provider in AppVeyor, but this didn't work at the first try with azure. I think Azure needs some more values to start the deployment.

Let's add some needed environment variables to the appveyor.yml:

~~~ yaml
environment:
  SITE_NAME: build-with-appveyor
  FTP_PASSWORD:
    secure: *******
  GITHUB_USERNAME: JuergenGutsch
  GITHUB_PASSWORD:
    secure: *******
  REPOSITORY_NAME: unittesting-aspnetcore
~~~

Please note the `secure` strings: this strings contains encrypted passwords, encrypted by AppVeyor. Without the secure option, you need to write the unencrypted passwords, which is not a good idea. **Don't store any secret information in the source code repositories!** A much better option is to set the environment variables on the build server only and to just use them in the scripts.

[!TODO IMAGE secure strings]

I prepared a PowerShell script that does the same job as the bash script in the previous article:

~~~ powershell
$uri = "https://`$$Env:SITE_NAME:$Env:FTP_PASSWORD@$Env:SITE_NAME.scm.azurewebsites.net/deploy?scmType=GitHub"
$headers = @{ }
$headers.Add("Content-Type", "application/json")
$headers.Add("Accept", "application/json")
$headers.Add("X-SITE-DEPLOYMENT-ID", $Env:SITE_NAME)
$headers.Add("Transfer-encoding", "chunked")
$body = @{ }
$body.Add("format", "basic")
$body.Add("url", "https://$Env:GITHUB_USERNAME:$Env:GITHUB_PASSWORD@GITHUB.org/$Env:GITHUB_USERNAME/$Env:REPOSITORY_NAME.git")

Invoke-RestRequest -Uri $uri -Method "POST" -Headers $headers -Body $body
~~~

We now need to call that script in the `deploy_script` section

~~~ yaml
deploy_script:
- ps: .\deploy-to-azure.ps1
~~~

This script uses the environment variables defined previously.

Now the deployment is triggered after a successful build. 

## Conclusion

AppVeyor has a lot more options an possibilities to configure a build. This is nice, but makes the build more complex and the settings to configure the builds are huge. As mentioned, I like simplicity, this is why I prefer the `appveyor.yml` over the web UI. I use AppVeyor a lot since I'm working on open source projects, it fits perfectly with Github and runs as expected.

Anyway, for more complex and more real-live projects I use a little different approach to configure the build. I don't configure all the build scripts separately in AppVeyor. For those projects I configure the build in CAKE-Script, which is C# MAKE using C#-script. This simplifies the AppVeyor configuration a lot more.

In the next post, I'l write about how to setup a build, test and deployment pipeline to azure using CAKE and any possible build server.