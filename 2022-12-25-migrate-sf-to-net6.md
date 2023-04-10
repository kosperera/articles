---
title: Migrating Service Fabric apps to .NET 6
subtitle: There's more to do than just flipping the target framework when migrating Service Fabric apps to the .NET 6
slug: upgrade-sf-to-net6.md
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: azure, servicefabric, net, migrations
cover: https://user-images.githubusercontent.com/958227/230860274-a8d02fb7-0a51-4c44-ae7a-ef8ed3e030cc.jpeg?auto=compress
domain: keepontruckin.hashnode.dev
---

Migrating Service Fabric apps to the .NET 6 suck! Not only is there no opensource community engagement, but there's [no same-day support for new .NET versions][qna-sf-community-meetup-no-same-day-support] at all. What a SHAM! There's nothing I can do about that.

[qna-sf-community-meetup-no-same-day-support]: https://youtu.be/LjClUEu9duk?t=2207

<img class="img-responsive img-comic" alt="2013-06-13_frustration_obliviousness_sales personnel_software_third party library_new version_windows_engineering" src="https://user-images.githubusercontent.com/958227/230864432-7ee6a01a-e105-41a2-abd7-6779e6068b65.gif?auto=compress" width="100%">

The Alertbox Inc. (@[alertboxinc](@alertboxinc)) pipelines page is open on my work PC and one of the CI pipelines is broken. For the record, Alertbox has ~60 microservices. They require Service Fabric / .NET Core combo to build and release. I'm going to go ahead and say that they use `windows-latest` on hosted agents. And the [Azure Pipelines team is upgrading the .NET used by agents][announcements-az-pipelines-dotnet-upgrade], so they have no way of running pipelines without [adding the UseDotNet task to select what .NET version is to use][hdi-select-dotnet] in the pipeline agents. Maybe that's a plausible fix, but they recommend keeping the source code up-to-date with the current LTS version of .NET. So what does each microservice have that must be upgraded for .NET 6?

[announcements-az-pipelines-dotnet-upgrade]: https://devblogs.microsoft.com/devops/upgrade-of-net-agent-for-azure-pipelines
[hdi-select-dotnet]: /select-the-dotnet-ver-to-use



### Migrating the Service Fabric app

The first thing we're going to need is the [latest update for Service Fabric][hdi-sf-getting-started]. And that means runtime, SDK, and the tools to [create, build, and run reliable service apps on a local cluster][hdi-sf-create-reliable-services]. Unfortunately, [the IIS team recently sunset the Web PI][announcements-iis-web-pi-sunset], so the Service Fabric team bundled the latest releases with Visual Studio, and I'm going to have to update Visual Studio to download them for me.

The other thing we're going to need is a *global.json* file since there are several .NET versions installed already in my work PC, so it's definitely necessary to break out that [`dotnet` driver and create a *global.json* file to select `net6.0` SDK version][hdi-select-dotnet] that is in.

[hdi-sf-getting-started]: https://learn.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started
[hdi-sf-create-reliable-services]: https://learn.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-services-quick-start#create-a-stateless-service
[announcements-iis-web-pi-sunset]: https://blogs.iis.net/iisteam/web-platform-installer-end-of-support-feed

```bash
#!/bin/zsh
# Show installed SDKs.
dotnet --list-sdks
# Add a global.json file
dotnet new globaljson --sdk-version 6.0.403
```

I'm a bit of a control freak, not gonna lie. I like the idea of knowing what's happening underneath. It's slightly obnoxious that [Service Fabric project templates are not opensourced][hdi-find-sf-project-templates] yet. I can [create a new reliable service app on .NET 6][hdi-sf-create-reliable-services] to figure out what's new in the scaffolding project.

[hdi-find-sf-project-templates]: https://github.com/microsoft/service-fabric/issues/1290#issuecomment-1489868105

The first on the list is the entry project *.sfproj*. They use a [non-SDK-style project template][proposal-sf-sdk-style] with a bunch of *.xml* files for configuration but with no C# code. They require [*Fabric.MSBuild* nuget][nuget-sf-msbuild-package] to build and package Service Fabric apps. Unfortunately, the `dotnet add package` command won't update dependencies since they only support `<PackageReference />` and non-SDK-style project template uses the *package.config* file to manage dependencies.

[proposal-sf-sdk-style]: https://github.com/microsoft/service-fabric/issues/885
[nuget-sf-msbuild-package]: https://www.nuget.org/packages/Microsoft.VisualStudio.Azure.Fabric.MSBuild

```xml
<!-- packages.config -->
<package id="Microsoft.VisualStudio.Azure.Fabric.MSBuild"
         version="1.7.7"
         targetFramework="net48" />
```

I can update the *package.config* to use the latest versions but there's more to do. The *.sfproj* still points to the older build tools. And you do want to use the latest version to build and package. 

Updating the *.sfproj* is extraordinarily uncomplicated. All we need to do is figure out the references to the *Fabric.MSBuild* package and replace the version numbers, say, find all and replace the version number to `1.7.7` from the version that is already been used. Then I can replace the `<TargetFrameworkVersion />` version with the .NET Framework 4.8 and we're done.

The *.xml* configurations need no changes, so we can ignore them for now.



### Migrating reliable services and dependencies

Next on the list are reliable services. This covers a lot of ground since Alertbox uses multiple stateless services bundled into a single Service Fabric app, say, Web APIs and Console apps, and they target `netcoreapp3.1` to build and run, so we're going to have to upgrade them to `net6.0`. I can replace `<DefaultNetCoreTragetFramework />` in the *Directory.Build.props* file like they do on [@dotnet repos on GitHub][gh-search-build-props-xml-files], but there's more to it. They need to upgrade dependencies.

[gh-search-build-props-xml-files]: https://github.com/search?q=org%3Adotnet+%3CTargetFrameworks%3E+language%3AXML&type=code&l=XML

Figuring out dependencies to upgrade for all the *.csproj* projects that are in the code repo is basically the same as listing installed SDKs.

```bash
#!/bin/zsh
# First restore packages
dotnet restore && \
# Then scan for outdated packages
dotnet list package --outdated
```

I can check and review each package to decide whether or not to upgrade. 

```bash
dotnet add package Azure.Messaging.ServiceBus
```

It's slightly obnoxious that they only let you check for the status, so I'm going to have to [upgrade outdated packages][hdi-update-packages] one at a time, say, migrating Web APIs and the Automapper configurations to the latest versions by going thru what's new and migration docs. It's taking a few tries to get the unit tests green since it <mark>demands highly intense concentration on breaking changes</mark> of each package upgrade.

[hdi-update-packages]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-add-package#examples



### Troubleshooting

Visual Studio can build from source but it keeps failing to deploy the Service Fabric app onto the local cluster. I'm going to go ahead and say that they use *Deploy-Application.ps1* powershell script in the *.sfproj* to deploy, so replacing it from the script in that new boilerplate code to take care of it seems like the right answer.

Then I can build and run the app from source and Visual Studio will do the rest for me. But there are more problems.

[The Visual Studio debugger won't attach.][sf-issue-vs-debugger-wont-attach] This may be difficult. I'll come back to this later.

[sf-issue-vs-debugger-wont-attach]: https://github.com/microsoft/service-fabric/issues/1356

This may not be an issue with Service Fabric since it works on my colleague's PC, so it's definitely necessary to diff both development environments to figure out what SDK version to use.

[Service Fabric creates dynamic proxy classes for service remoting,][explain-why-vs-debugger-wont-attach] but the .NET runtime team has [turned off metadata generation for dynamic classes][explain-why-metadata-wont-generate] in .NET 6, so it would be impossible to tell Visual Studio debugger how to load metadata for dynamic proxy classes that Service Fabric creates. The .NET team has a [release that always generates metadata][dotnet-runtime-fix-for-metadata-gen], so a little `winget install` magic is all that's necessary to update the .NET 6 installed locally.

[explain-why-vs-debugger-wont-attach]: https://github.com/microsoft/service-fabric/issues/1356#issuecomment-1362622324
[explain-why-metadata-wont-generate]: https://github.com/dotnet/runtime/issues/62977#issuecomment-1186149824
[dotnet-runtime-fix-for-metadata-gen]: https://github.com/dotnet/runtime/pull/77322

```bash
#!/bin/zsh
# Display the status of the installed SDK.
dotnet sdk check
# Install/upgrade the SDK via Windows Package Manager.
winget install Microsoft.DotNet.SDK.6
# Show installed SDKs.
dotnet --list-sdks
```

But there's more to do. We need to update the *global.json* with the new SDK version.

```json
// global.json
{
  "sdk": {
    "version": "6.0.407",
    "allowPrerelease": false,
    "rollForward": "latestMinor"
  }
}
```

Then I can build and run from source and Visual Studio debugger will do the rest for me. And if version `>=6.0.407` is not installed locally, nothing gets compiled.

### Updating pipelines to select only .NET 6

I'm not exactly sure when the Azure Pipelines team is going to remove .NET 6 once it reaches end-of-life support, so I'm going to have to update the build steps to [always select the .NET version used in the code repo][hdi-select-dotnet]. All I need to do is push all changes to a feature branch to raise a pull request and the branch policies will trigger the CI pipeline for me. Excellent!



/KP