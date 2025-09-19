# Azure App Service

[Implement Azure App Service web apps](https://learn.microsoft.com/en-us/training/paths/create-azure-app-service-web-apps/)

## Overview

Azure App Service is a fully managed platform designed to simplify the deployment and scaling of web apps, mobile back ends, and RESTful APIs. It abstracts away infrastructure management, letting you focus on writing code and shipping features faster.

You can build using your preferred stack—whether it’s .NET, Java (Java SE, Tomcat, JBoss), Node.js, Python, or PHP—and deploy to either Windows or Linux environments. If you're working with containers, App Service also supports custom container deployments, giving you full control over your runtime.

### Built-in auto scale support

Azure App Service auto-scales.

### Container support

With Azure App Service, you can deploy and run containerized web apps on Windows and Linux. You can pull container images from a private Azure Container Registry or Docker Hub. Azure App Service also supports multi-container apps, Windows containers, and Docker Compose for orchestrating container instances.

### Continuous integration/deployment support (CI/CD)

The Azure portal provides out-of-the-box continuous integration and deployment with Azure DevOps Services, GitHub, Bitbucket, FTP, or a local Git repository on your development machine. Connect your web app with any of the above sources and App Service can automatically sync your code and apply changes as they’re pushed to the connected repository. Continuous integration and deployment for containerized web apps is also supported using either Azure Container Registry or Docker Hub.

### Deployment slots

When deploying a web app, you can use a separate deployment slot instead of the default production slot when you're running in the Standard App Service pricing tier or better. Deployment slots are live apps with their own host names. App content and configurations elements can be swapped between two deployment slots, including the production slot.

### App Service on Linux

App Service can also host web apps natively on Linux for supported application stacks. It can also run custom Linux containers (also known as Web App for Containers). App Service on Linux supports many language specific built-in images. Just deploy your code. Supported languages and frameworks include: .NET Core, Java (Tomcat, JBoss EAP, or Java SE with an embedded web server), Node.js, Python, and PHP. If the runtime your application requires isn't supported in the built-in images, you can deploy it with a custom container.

The languages, and their supported versions, are updated regularly. You can retrieve the current list by using the following command in the Cloud Shell.

```bash
az webapp list-runtimes --os-type linux
```

### Limitations

App Service on Linux does have some limitations:

- App Service on Linux isn't supported on Shared pricing tier.
- The Azure portal shows only features that currently work for Linux apps. As features are enabled, they're activated on the portal.
- When deployed to built-in images, your code and content are allocated as a storage volume for web content, backed by Azure Storage. The disk latency of this volume is higher and more variable than the latency of the container filesystem. Apps that require heavy read-only access to content files might benefit from the custom container option, which places files in the container filesystem instead of on the content volume.

### App Service Environment

App Service Environment is an Azure App Service feature that provides a fully isolated and dedicated environment for running App Service apps. It offers improved security at high scale.

## Examine Azure App Service plans

An App Service plan defines a set of compute resources for a web app to run. One or more apps can be configured to run on the same computing resources (or in the same App Service plan).

When you create an App Service plan in a certain region (for example, West Europe), a set of compute resources is created for that plan in that region. Whatever apps you put into this App Service plan run on these compute resources as defined by your App Service plan. Each App Service plan defines:

- Operating System (Windows, Linux)
- Region (West US, East US, etc.)
- Number of VM instances
- Size of VM instances (for example, P1v3, P2v3, based on pricing tier)
- Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, IsolatedV2)

The pricing tier of an App Service plan determines what App Service features you get and how much you pay for the plan. There are a few categories of pricing tiers:

- Shared compute: Free and Shared, the two base tiers, runs an app on the same Azure VM as other App Service apps, including apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources can't scale out.
- Dedicated compute: The Basic, Standard, Premium, PremiumV2, and PremiumV3 tiers run apps on dedicated Azure VMs. Only apps in the same App Service plans share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.
- Isolated: The Isolated and IsolatedV2 tiers run dedicated Azure VMs on dedicated Azure Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the maximum scale-out capabilities.

### How does my app run and scale?

In the Free and Shared tiers, an app receives CPU minutes on a shared VM instance and can't scale out.

In other tiers, an app runs and scales as follows:

- An app runs on all the VM instances configured in the App Service plan.
- If multiple apps are in the same App Service plan, they all share the same VM instances.
- If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.
- If you enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.

In this way, the App Service plan is the scale unit of the App Service apps. If the plan is configured to run five VM instances, then all apps in the plan run on all five instances. If the plan is configured for autoscaling, then all apps in the plan are scaled out together based on the autoscale settings.

### What if my app needs more capabilities or features?

Your App Service plan can be scaled up and down at any time. It's as simple as changing the pricing tier of the plan. If your app is in the same App Service plan with other apps, you might want to improve the app's performance by isolating the compute resources. You can do it by moving the app into a separate App Service plan.

You can potentially save money by putting multiple apps into one App Service plan. However, since apps in the same App Service plan all share the same compute resources you need to understand the capacity of the existing App Service plan and the expected load for the new app.

Isolate your app into a new App Service plan when:

- The app is resource-intensive.
- You want to scale the app independently from the other apps in the existing plan.
- The app needs resources in a different geographical region.

This approach gives you a dedicated resource pool and greater control over your app’s performance and scaling.
