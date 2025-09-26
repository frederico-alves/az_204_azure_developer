# Configure web app settings

[Configure web app settings](https://learn.microsoft.com/en-us/training/modules/configure-web-app-settings/)

## App settings

App settings are variables passed as environment variables to the application code. App settings are always encrypted when stored (encrypted-at-rest). App settings names can only contain letters, numbers (0-9), periods ("."), and underscores ("\_") Special characters in the value of an App Setting.

Application settings can be accessed by navigating to your app's management page and selecting `Environment variables > Application settings`.

### Edit application settings in bulk

To add or edit app settings in bulk, select the Advanced edit button. When finished, select OK. Don't forget to select Apply back in the Environment variables page. App settings have the following JSON formatting:

```json
[
  {
    "name": "<key-1>",
    "value": "<value-1>",
    "slotSetting": false
  },
  {
    "name": "<key-2>",
    "value": "<value-2>",
    "slotSetting": false
  },
  ...
]
```

### Configure environment variables for custom containers

Your custom container might use environment variables that need to be supplied externally. You can pass them in via the Cloud Shell. In Bash:

```Azure CLI
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings key1=value1 key2=value2
```

In PowerShell:

```Azure PowerShell
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
```

When your app runs, the App Service app settings are injected into the process as environment variables automatically. You can verify container environment variables with the URL `https://<app-name>.scm.azurewebsites.net/Env`.

## Configure general settings

In the `Configuration > General settings` section you can configure some common settings for your app. Some settings require you to scale up to higher pricing tiers.

A list of the currently available settings:

- Stack settings: The software stack to run the app, including the language and SDK versions. For Linux apps and custom container apps, you can also set an optional start-up command or file.
- Platform settings: Lets you configure settings for the hosting platform, including:
  - Platform bitness: 32-bit or 64-bit. For Windows apps only.
  - FTP state: Allow only FTPS or disable FTP altogether.
  - HTTP version: Set to 2.0 to enable support for HTTPS/2 protocol.
  - Web sockets: For ASP.NET SignalR or socket.io, for example.
  - Always On: Keeps the app loaded even when there's no traffic. When Always On isn't turned on (default), the app is unloaded after 20 minutes without any incoming requests. The unloaded app can cause high latency for new requests because of its warm-up time. When Always On is turned on, the front-end load balancer sends a GET request to the application root every five minutes. The continuous ping prevents the app from being unloaded. Always On is required for continuous WebJobs or for WebJobs that are triggered using a CRON expression.
  - ARR affinity: In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to Off for stateless applications.
  - HTTPS Only: When enabled, all HTTP traffic is redirected to HTTPS.
  - Minimum TLS version: Select the minimum TLS encryption version required by your app.
- Debugging: Enable remote debugging for ASP.NET, ASP.NET Core, or Node.js apps. This option turns off automatically after 48 hours.
- Incoming client certificates: Require client certificates in mutual authentication. TLS mutual authentication is used to restrict access to your app by enabling different types of authentication for it.

## Configure path mappings

In the Configuration > Path mappings section you can configure handler mappings, and virtual application and directory mappings. The Path mappings page displays different options based on the OS type.

### Windows apps (uncontainerized)

Each app has the default root path (`/`) mapped to `D:\home\site\wwwroot`, where your code is deployed by default. If your app root is in a different folder, or if your repository has more than one application, you can edit or add virtual applications and directories.

### Linux and containerized apps

You can add custom storage for your containerized app. Containerized apps include all Linux apps and also the Windows and Linux custom containers running on App Service. Select New Azure Storage Mount and configure your custom storage as follows:

- Configuration options: Basic or Advanced. Select Basic if the storage account isn't using service endpoints, private endpoints, or Azure Key Vault. Otherwise, select Advanced.
- Storage accounts: The storage account with the container you want.
- Storage type: Azure Blobs or Azure Files. Windows container apps only support Azure Files. Azure Blobs only supports read-only access.
- Storage container: For basic configuration, the container you want.
- Share name: For advanced configuration, the file share name.
- Access key: For advanced configuration, the access key.
- Mount path: The absolute path in your container to mount the custom storage.
- Deployment slot setting: When checked, the storage mount settings also apply to deployment slots.

## Enable diagnostic logging

There are built-in diagnostics to assist with debugging an App Service app.

- Application logging; Windows, Linux;
- Deployment logging; Windows, Linux;

### Enable application logging (Linux/Container)

1. In App Service logs set the Application logging option to File System.
2. In Quota (MB), specify the disk quota for the application logs. In Retention Period (Days), set the number of days the logs should be retained.
3. When finished, select Save.

### Enable web server logging

1. For Web server logging, select Storage to store logs on blob storage, or File System to store logs on the App Service file system.
2. In Retention Period (Days), set the number of days the logs should be retained.
3. When finished, select Save.

### Add log messages in code

In your application code, you use the usual logging facilities to send log messages to the application logs. For example:

- ASP.NET applications can use the `System.Diagnostics.Trace` class to log information to the application diagnostics log. For example:

```C#
System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");
```

Python applications can use `azure-monitor-opentelemetry` to send logs to the application diagnostics logs.

### Stream logs

Before you stream logs in real time, enable the log type that you want.

- Azure portal - To stream logs in the Azure portal, navigate to your app and select Log stream.
- Azure CLI - To stream logs live in Cloud Shell, use the following command: `az webapp log tail --name appname --resource-group myResourceGroup`
- Local console - To stream logs in the local console, install Azure CLI and sign in to your account. Once signed in, follow the instructions shown for Azure CLI.

### Access log files

If you configure the Azure Storage blobs option for a log type, you need a client tool that works with Azure Storage.

For logs stored in the App Service file system, the easiest way is to download the ZIP file in the browser at:

- Linux/container apps: `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`

## Configure security certificates

Azure App Service has tools that let you create, upload, or import a private certificate or a public certificate into App Service.

A certificate uploaded into an app is stored in a deployment unit that is bound to the app service plan's resource group and region combination (internally called a webspace). The certificate is accessible to other apps in the same resource group and region combination.

- Create a free App Service managed certificate; A private certificate that's free of charge and easy to use if you just need to secure your custom domain in App Service.
- Purchase an App Service certificate; A private certificate managed by Azure. It combines the simplicity of automated certificate management and the flexibility of renewal and export options.
- Import a certificate from Key Vault; Useful if you use Azure Key Vault to manage your certificates.
- Upload a public certificate; Public certificates aren't used to secure custom domains, but you can load them into your code if you need them to access remote resources.

### Private certificate requirements

The free App Service managed certificate and the App Service certificate already satisfy the requirements of App Service. If you want to use a private certificate in App Service, your certificate must meet the following requirements:

- Exported as a password-protected PFX file, encrypted using triple DES.
- Contains private key at least 2,048 bits long.
- Contains all intermediate certificates and the root certificate in the certificate chain.

To secure a custom domain in a TLS binding, the certificate has other requirements:

- Contains an Extended Key Usage for server authentication (OID = 1.3.6.1.5.5.7.3.1)
- Signed by a trusted certificate authority

### Creating a free managed certificate

To create custom TLS/SSL bindings or enable client certificates for your App Service app, your App Service plan must be in the Basic, Standard, Premium, or Isolated tier.

The free App Service managed certificate is a turn-key solution for securing your custom DNS name in App Service. It's a TLS/SSL server certificate fully managed by App Service and renewed continuously and automatically in six-month increments, 45 days before expiration. You create the certificate and bind it to a custom domain, and let App Service do the rest.

The free certificate comes with the following limitations:

- Doesn't support wildcard certificates.
- Doesn't support usage as a client certificate by using certificate thumbprint, which is planned for deprecation and removal.
- Doesn't support private DNS.
- Isn't exportable.
- Isn't supported in an App Service Environment (ASE).
- Only supports alphanumeric characters, dashes (-), and periods (.).
- Only custom domains of length up to 64 characters are supported.

### Import an App Service Certificate

If you purchase an App Service Certificate from Azure, Azure manages the following tasks:

- Takes care of the purchase process from certificate provider.
- Performs domain verification of the certificate.
- Maintains the certificate in Azure Key Vault.
- Manages certificate renewal.
- Synchronize the certificate automatically with the imported copies in App Service apps.

If you already have a working App Service certificate, you can:

- Import the certificate into App Service.
- Manage the certificate, such as renew, rekey, and export it.
