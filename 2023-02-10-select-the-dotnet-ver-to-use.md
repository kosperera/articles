---
title: Selecting what .NET version is to use
subtitle: They removed .NET 5 from pipeline agents but I'm way too lazy to migrate to .NET 6
slug: select-the-dotnet-ver-to-use
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: azure, devops, continuous-integration, net, vscode, csharp
cover: https://user-images.githubusercontent.com/958227/233758740-54d5f724-44e8-4982-b095-cd40a3c5e1ae.jpeg?auto=compress
domain: keepontruckin.hashnode.dev
---

I'm a little procrastinated, not gonna lie. The Alertbox Inc. (@[alertboxinc](@alertboxinc)) pipelines page is open on my work PC and one of the CI pipelines is broken. What a SHAM!

It's a little weird since the pipeline logs show the target .NET version `net5.0` isn't available, but the changeset seems to work locally. Not only are there no tests failing, but there are no errors at all. For the record, Alertbox has several hundred git repos and [pipelines that they run quite often on hosted agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#use-a-microsoft-hosted-agent). The truth is they have not upgraded the .NET used by the code repos to the [latest version](https://versionsof.net/core/), so they require several .NET versions installed locally.

<img class="img-responsive img-comic" alt="2017-06-10_budget_money_spending_projects_upgrades_technology_software_engineering" src="https://user-images.githubusercontent.com/958227/229605458-5b7203ba-af2a-4bbc-bbf6-6c03cbf2c257.gif" width="100%">

Maybe they don't have [.NET end-of-life support versions](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core#lifecycle) on hosted agents anymore? Yup! [The Azure Pipeline team is upgrading the .NET used by pipeline agents](https://devblogs.microsoft.com/devops/upgrade-of-net-agent-for-azure-pipelines/). I almost want to upgrade the .NET used by the code repo to the latest LTS version, say, replacing `TargetFramework` to `net6.0` from `net5.0` and running `dotnet list package --outdated` to see what packages are to upgrade. You'd be amazed how problematic is it to have several .NET versions running locally since [the `dotnet` driver selects the latest SDK installed locally](https://learn.microsoft.com/en-us/dotnet/core/versions/selection#the-sdk-uses-the-latest-installed-version) for commands and not from the entry project / solution that's in, so it would be difficult to figure out what packages are to upgrade without going thru all of them one at a time completely. And .NET Core 3.1 will be end-of-life support soon, so that means more code repos to upgrade.

It's not such a great idea and probably not even pragmatic to go thru all of that *Jazz* and upgrade each one since you can only boil so many oceans at the same time, so it's definitely necessary to break out that pipeline and modify the YAML script to install any end-of-life support .NET versions.

```yaml
#.azuredevops/steps/build.yaml
steps:
# Install .NET from global.json
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    useGlobalJson: true
```

All I need to do is add a [UseDotNet task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/use-dotnet-v2?view=azure-pipelines) as a prerequisite for the build step and it will install .NET on pipelines for me. But there are more problems. They require a [*global.json* file](https://learn.microsoft.com/en-us/dotnet/core/tools/global-json#globaljson-schema) to figure out what SDK version is to install.

```json
// global.json
{
  "sdk": {
    "version": "5.0.408",
    "allowPrerelease": false,
    "rollForward": "latestMinor"
  }
}
```

I can run `dotnet --list-sdks` to get the SDK version that is running on my work PC, and then run `dotnet new globaljson` to create the *global.json* file to mention that SDK version. Then I can [run the pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/manage-pipelines-with-azure-cli?view=azure-devops#example) and the `UseDotNet@2` task will take care the rest for me. Excellent! Moving right along.

```bash
# Trigger the pipeline
az pipelines run \
   --branch kp/fix-23-dotnet-not-found \
   --folder-path ci \
   --name ca-cocktails \
   --output table \
   --open
```

```bash
# Check for build status
az pipelines runs list \
   --top 3 \
   --status all \
   --branch kp/fix-23-dotnet-not-found \
   --output table
```

It's taking a few tries to [get the pipeline green](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runs?view=azure-devops#example) since [some of the dependencies and C# features](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/breaking-changes) don't seem to recognize anymore since [.NET 5 only supports C# 9](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-5#c-updates) and not latest version. This may be difficult. I'll come back to this later!



/KP