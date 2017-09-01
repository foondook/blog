---
layout: post
title: "Add Authentication and Authorization to a Razor Pages web"
teaser: "I also got asked about Razor Pages Authentication, when I published my first post about Razor Pages. In this post I'm going to write down how Authentication and Authorization works in the Razor Pages. This is a bit unintuitive for developers who are not familiar with ASP.NET Core. "
author: "Jürgen Gutsch"
comments: true
image: /img/cardlogo-dark.png
tags: 
- .NET Core
- Unit Test
- XUnit
- MSTest
---

I also got asked about Razor Pages authentication, when I published my first post about Razor Pages. In this post I'm going to write down how authentication and authorization works in the Razor Pages. This is a bit unintuitive for developers who are not familiar with ASP.NET Core. 

## Setup a Razor Page project

To try this out, you need to have the latest preview of Visual Studio 2017 installed. (I use 15.3.0 preview 3) And you need .NET Core 2.0 preview installed (2.0.0-preview2-006497 in my case)

In Visual Studio 2017, use "File... New Project" to create a new project. Navigate to ".NET Core", chose the "ASP.NET Core Web Application (.NET Core)" project  and choose a name and a location for that new project.

![]({{ site.baseurl }}/img/razor-pages/new-project.PNG)

In the next dialogue, you probably need to switch to ASP.NET Core 2.0 to see all the new available project types. (I will write about the other ones in the next posts.) Select the "Web Application (Razor Pages)" and pressed "OK".

![]({{ site.baseurl }}/img/razor-pages/new-razor-pages.PNG)

Important: As you can see, the button "Change Authentication" is grey out, that means you are not able to add or change authentication using tha dialog. This is why I wrote it is not really intuitive for an developer, who is not familiar with ASP.NET Core. I hope Microsoft will change it for that project template, because there is no reason why it's disabled.

After pressing "OK" the new ASP.NET Core razor pages project is created without any authentication.

## Adding Authentication

As mentioned in the last post about the Razor Pages, this project type is part of the MVC framework, even though it is not using Controllers or the MVC pattern. But this means it should be possible to configure Authentication in the same way as in usual MVC projects.

Even Razor Pages projects are referencing the Microsoft.AspNetCore.All package, which includes all the needed stuff to your project, including Authentication and Authorization.

Let's try it with the Authentication via individual user accounts. Feel free to try it with any other auth type.

The easiest way to configure auth  is by copying the needed stuff from a regular MVC project. FIrst I'm going to copy the Data and the Models folder, which contain the EF DataContext and the Models used in this contexts. I'm also copying the Services folder, which contains services to send emails and text messages to the users

After this is done, we need to configure this stuff in the startup.cs. We need to add a few lines to the ConfigureServices method to register the DbContext, to register the Identity stuff and to register our copied services:

~~~ csharp
public void ConfigureServices(IServiceCollection services)
{
  services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

  services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

  services.AddTransient<IEmailSender, AuthMessageSender>();
  services.AddTransient<ISmsSender, AuthMessageSender>();

  services.AddMvc();
}
~~~

Inside the Configure method, we only need to add two lines of code. Because we are now handling with a database we should handle and render database errors in developer mode:

~~~csharp
if (env.IsDevelopment())
{
  app.UseDeveloperExceptionPage();
  app.UseBrowserLink();
  app.UseDatabaseErrorPage();
}
else
{
  app.UseExceptionHandler("/Home/Error");
}
~~~

And we need to add the AuthenticationMiddleWare to the HTTP request pipeline:

~~~ csharp
app.UseStaticFiles();

app.UseAuthentication();

app.UseMvc(routes =>
{
  routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
});
~~~

There is one more thing to do: We need to add a connections string to the appsettings.json:

~~~ json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=aspnet-WebApplication2-035BF05A-157A-4711-ABA4-CE52D5EEF5CB;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  // ...
}
~~~

I added this lines at the beginning of that file. With this in place, auth is basically configured to your app.

## Configure Authorization

Unfortunately using the Authorization attribute is not yet supported in the .NET Core 2.0 preview 2, but it will definitely come to the public release. This is why filters are not yet supported in razor pages.

That means authorization needs to be done manually.