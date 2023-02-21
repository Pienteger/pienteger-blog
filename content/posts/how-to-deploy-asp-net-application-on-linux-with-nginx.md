---
title: How to deploy ASP.NET application on Linux with NGINX
date: 2023-02-20
tags:
  - Linux
  - ASP.NET
  - CSharp
social_image: /media/rocket.jpg
description: Deploying ASP.NET on Linux and configuring NGINX as reverse proxy is easy. This post will show you how to do it.
---

## Introduction

From the beginning, ASP.NET has been a Windows-only framework. However, with the release of .NET Core, ASP.NET is now available on Linux and can be deployed on Linux servers that uses Apache or Nginx as a web server instead of IIS. In this post, we will guide you how to deploy ASP.NET on Linux. We will not go through the details, instead, we will focus on the steps to deploy ASP.NET on Linux.

## Prerequisites

- Access to a Linux vps server with any of the following distro with a standard user account with sudo privilege.
  - Ubuntu 20.04 or later
  - Red Hat Enterprise (RHEL) 8.0 or later
  - SLES 12 or 15
- The latest stable [.NET runtime installed](#installing-net-runtime) on the server.
- An existing ASP.NET Core app.

### Installing .NET Runtime

The first step is to install the .NET runtime on the server. You can follow the instructions on the [official documentation](https://learn.microsoft.com/en-us/dotnet/core/install/linux-scripted-manual#manual-install) to install the .NET runtime on your server.

## Publish and copy the app

It is not ideal, but first let's disable the HTTPS Redirection middleware. To do that, open the `Program.cs` file and comment out the following line:

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}
```

Also, remove `https://localhost:5001` (if present) from the `applicationUrl` property in the `Properties/launchSettings.json` file.

Now, publish the app by running the following command and copy the ASP.NET Core app to the server using a tool that integrates into the organization's workflow (for example, `SCP`, `SFTP`). It's common to locate web apps under the var directory (for example, `var/www/nishadolapp`).

```bash
dotnet publish --configuration Release
```



### Test the app

- From the command line, run the app: `dotnet <app_assembly>.dll`.
- In a browser, navigate to `http://<serveraddress>:<port>` to verify the app works on Linux locally.

> Under a production deployment scenario, a [continuous integration workflow](/ease-your-asp-net-deployment-with-github-action) does the work of publishing the app and copying the assets to the server.

## Configure a reverse proxy server

First let's add **Forwarded Headers Middleware** to the app. This middleware will help the app to get the correct client IP address and the scheme (HTTP or HTTPS) from the `X-Forwarded-For` and `X-Forwarded-Proto` headers and help other security policies to work correctly. To do that, open the `Program.cs` file and add the following line before the `UseAuthentication()` middleware:

```csharp
using Microsoft.AspNetCore.HttpOverrides;
...
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});
app.UseAuthentication();
```
