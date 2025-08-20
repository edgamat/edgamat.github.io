---
layout: post
title: 'Securing Your Supply Chain - Using a Private Nuget Feed'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

We have a private Nuget Feed for some internal packages we publish. Setting it up is easy, but is it secure?

<!--more-->

### Dangers Are Everywhere

There are many ways someone could attempt to insert their own code into our application using Nuget. One way is to confuse Nuget with a separate package that uses the same name as the one you publish. Another is to create one that contains a typo, hoping you will make the mistake and restore their package instead of the real one you intended.

If you use a public and a private feed, you are open to such dangers. But if you use just a single private feed, and you configure it correctly, these risk are greatly reduced.

We host our code in Azure DevOps and have it host a private feed for us. This ability to host a private feed is not unique to Azure DevOps. Many hosting services exist for package repositories. But this is what we are using, so I will use it as my example.

### Adding the Private Feed

One adds a new source using the .NET CLI:

```powershell
dotnet nuget add source -n <NameOfSource> <PackageFeedUrl>
```

The `<NameOfSource>` is a friendly name for the source, like "PrivateFeed".

For Azure Artifacts, the `PackageFeedUrl` is like this:

`https://pkgs.dev.azure.com/<MyOrg>/<MyFeedID>/_packaging/<MyFeedName>>/nuget/v3/index.json`

If your feed requires authentication (if it doesn't then you've done it wrong!) you can use the credentials manager to authenticate the feed by installing the Azure Artifacts Credentials Provider using the instructions here:

<https://github.com/microsoft/artifacts-credprovider#azure-artifacts-credential-provider>

Then, use the .NET CLI to restore the packages:

```powershell
dotnet restore --interactive 
```

If you don't use the `--interactive` option, the `restore` command will return errors when attempting to lookup packages.

All of this is taken care for you if you restore the packages within Visual Studio. It will prompt you to setup credentials.

### Using Credentials

Instead of using the Azure Artifacts Credentials provider, you can store the credentials in the `nuget.config` file directly. This is what occurs when you provide the credentials when using the `dotnet nuget add source` command. For Azure, the credentials are in the form of your username (an email address usually) and a personal access token you generate on the DevOps website. Using them will store the credentials in the `nuget.config` file:

```powershell
dotnet nuget add source -u <UserName> -p <PersonalAccessToken> -n <NameOfSource> <PrivateFeedUrl>
```

### Package Source Mappings

You can setup your own set of package source mappings in the Nuget.config file to ensure you restore your private packages from the private feed only. This removes the possibility of pulling down a copy of the same package on the public feed.

To find your Nuget.config file, run this command:

```powershell
dotnet nuget config paths
```

One my laptop it lists one path, but you may have more than one. Mine is located here:

`%APPDATA%\Nuget\nuget.config`

It contains the 2 sources:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="PrivateFeed" value="<PackageFeedUrl>" />
  </packageSources>
</configuration>
```

Here is a package source mapping that ensures all packages that start with `MyPrefix.` will only be restored from the private feed:

```xml
  <packageSourceMapping>
    <source key="PrivateFeed">
      <package pattern="MyPrefix.*" />
    </source>
  </packageSourceMapping>
```

### Adding a Nuget.config File to Your Repo

In many scenarios, it could be wiser to add the Nuget.config file directly into you source code repository. It allows everyone to use the same settings. And it also makes things easier when working on multiple projects that don't share the same package sources. Placing it in the same folder as the solution file will have it be used automatically. If not, then you must provide the config file as an option, each time you run a command:

```powershell
dotnet restore --configfile "<PathToNugetConfigFile>"
```

If you decide to do include a `nuget.config` file in your repo, you need to ensure that you don't store the credentials in the file. There are a couple of ways around this. One is to use an environment variable for the credentials, where the key/name of the variable is:

`NuGetPackageSourceCredentials_{NameOfSource}`

And the value is `Username={username};Password={password}`

The other way is to use an environment variable for the password:

```xml
<packageSourceCredentials>
    <PrivateFeed>
        <add key="Username" value="user@contoso.com" />
        <add key="ClearTextPassword" value="%PersonalAccessToken%" />
    </PrivateFeed>
</packageSourceCredentials>
```

Either approach is fine. In your CI/CD pipeline, it is possible to inject an environment variable, as well as on each developer's workstation/laptop. Which approach you chose is likely going to depend on how you authenticate with your private feed.

### Should I Only Use a Private Feed?

Now that you have set up a private feed, you might ask if you should remove the public feed? Using the private feed only means you have more control over the packages you are using. You also remove the possibility of someone tricking you into pulling down a package with the same name as the one you add to your private feed. But everyone's scenario is different and there maybe situations where having both a public and private feed makes a lot of sense. Just make sure you use package source mappings for all your private packages.

### Summary

Using a private package feed allows you to host your own packages without having to make them public. But doing so comes with some security risks you need to be aware of. You must configure Nuget correctly using package source mappings to ensure you restore your packages only from your private feed. It can make a lot of sense to keep the `nuget.config` file in your repository to ensure everyone uses the same settings. Just make sure you don't store any credentials in this file!


