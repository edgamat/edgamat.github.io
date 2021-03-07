---
layout: post
title: 'IIS Config Locations'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When modifying IIS sometimes i forget where the config files are located. Let's write them down so I don't have to hunt for them in the future.

<!--more-->

### Server Level Config

The root of the IIS server has it's own config file. This is the root of all settings under IIS. If you add websites or FTP sites to your server, they all will inherit from these settings.

On Windows 10 (and Windows Server 2016 and higher) these settings are found here:

```
C:\Windows\System32\inetsrv\config\applicationHost.config
```

From a security perspective, it is not a good practice to broadcast too much information about your server. One such example that IIS includes with each response is a header called `X-Powered-By`. This file is where this is defined:

```
<httpProtocol>
    <customHeaders>
        <clear />
        <add name="X-Powered-By" value="ASP.NET" />
    </customHeaders>
    <redirectHeaders>
        <clear />
    </redirectHeaders>
</httpProtocol>
```

You can simply remove it from this file, save the changes, and it won't be broadcast any further.

Starting with IIS 10, you can remove the `Server` header as well. In the `applicationHost.config` file, find the following section:

```
system.WebServer/security/requestFiltering
```

Add an attribute to the element called `removeServerHeader`, and save the changes:

```
<requestFiltering removeServerHeader="true">
    ...
```

### IIS Express

**Running IIS Express from the Command Line**

When running IIS Express from the command line, the same configuration for IIS Express can be found here:

```
C:\Users\{User}\Documents\IISExpress\config\applicationhost.config
```

**Running IIS Under Microsoft Visual Studio**

When using Visual Studio to run IIS Express, a copy of the `applicationHost.config` file is placed here:

```
[SolutionFolder]\.vs\WebApplication6\config\applicationhost.config
```

**Running IIS Under JetBrains Rider**

When using JetBrains Rider to run IIS Express, a copy of the `applicationHost.config` file is placed here:

```
[SolutionFolder]\.idea\config\applicationhost.config
```

### Editing The File with AppCmd.exe

There are 3 ways to edit the `applicationHost.config`:

1. Use a test editor (like notepad)
2. Use the Configuration Editor Feature within the IIS Manager
3. Use the `AppCmd.exe` command-line tool

To edit the file with AppCmd.exe, run the following command:

```
appcmd set config /section:system.webServer/security/requestFiltering /removeServerHeader:true /apphostconfig:[full path to config file]
```

The full path to AppCmd.exe is here: 

```
C:\Windows\system32\inetsrv\appcmd.exe
```

A version is also included with IIS Express:

```
C:\Program Files\IIS Express\appcmd.exe
```

Here is a great resource for using the commands:

[https://blogs.iis.net/eokim/understanding-appcmd-exe-list-set-config-configurationpath-section-name-parameter-name-value](https://blogs.iis.net/eokim/understanding-appcmd-exe-list-set-config-configurationpath-section-name-parameter-name-value)

For example to remove the 'X-Powered-By' header, run this command:

```
appcmd set config /section:system.webServer/httpProtocol /-"customHeaders.[name='X-Powered-By']" /apphostconfig:[full path to config file]]
```
