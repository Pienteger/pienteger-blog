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

## Install NGINX

Nginx is a web server that can also be used as a reverse proxy, load balancer, mail proxy and HTTP cache. Most importantly, it is free and open source. To install NGINX on your server, run the following command:

```bash
sudo apt update
sudo apt install nginx
```

If a W: GPG error: <https://nginx.org/packages/ubuntu> focal InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY $key is encountered during the NGINX repository update, execute the following:

```bash
## Replace $key with the corresponding $key from your GPG error.
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $key
sudo apt update
sudo apt install nginx
```

Now, since Nginx was installed for the first time, explicitly start it by running:

```bash
sudo systemctl start nginx
sudo systemctl status nginx # To check if the service is running
```

## Configure NGINX

To configure Nginx as a reverse proxy to forward HTTP requests to your ASP.NET Core app, modify `/etc/nginx/sites-available/default`. Open it in a text editor, and replace the contents with the following snippet:

```text
server {
    listen        80;
    server_name   example.com *.example.com;
    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

> If your application is leveraging SignalR, please read the [Official Documentation](https://learn.microsoft.com/en-us/aspnet/core/signalr/scale?view=aspnetcore-6.0#linux-with-nginx) for more additional configuration.

Once the Nginx configuration is established, run `sudo nginx -t` to verify the syntax of the configuration files. If the configuration file test is successful, force Nginx to pick up the changes by running sudo `nginx -s reload`.

Now check if the app can be run the server:

- Navigate to the app's directory.
- Run the app: dotnet <app_assembly.dll>, where app_assembly.dll is the assembly file name of the app.
- Run `curl http://localhost:5000` to verify the app works locally.
- Go to the browser and navigate to `http://<serveraddress>:<port>` to verify the app works across the internet.

> If the app runs on the server but fails to respond over the internet, check the server's firewall and confirm port 80 is open.

When done testing the app, shut down the app with <kbd>Ctrl+C</kbd> (Windows) or <kbd>⌘+C</kbd> (macOS) at the command prompt.

## Create a systemd service

Create a new file called `nishadolapp.service` under the `/etc/systemd/system/` directory and add the following content:

```bash
[Unit]
Description= Sample ASP.NET Core app running on Ubuntu 20.04

[Service]
WorkingDirectory=<app_directory>
ExecStart=/usr/bin/dotnet <app_directory>/<assembly>.dll --urls=http://localhost:port/
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=nishadolapp
User=root
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

Now, enable and start the service by running the following command:

```bash
sudo systemctl enable nishadolapp.service
sudo systemctl start nishadolapp.service
```

If you redeploy the app, you need to restart the service by running the following command:

```bash
sudo systemctl restart nishadolapp.service
```

If you want to stop the service, run the following command:

```bash
sudo systemctl stop nishadolapp.service
```

To check if the service is running, run the following command:

```bash
sudo systemctl status nishadolapp.service
```

If you see any error, you can check the logs by running the following command:

```bash
sudo journalctl -fu nishadolapp.service
## sudo journalctl -fu kestrel-helloapp.service --since "2016-10-18" --until "2016-10-18 04:00" 
```

## Conclusion

In this article, you learned how to deploy an ASP.NET Core app to a Linux server. You also learned how to configure NGINX as a reverse proxy to forward HTTP requests to your ASP.NET Core app. Finally, you learned how to create a systemd service to run the app as a background process.
