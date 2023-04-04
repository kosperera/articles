---
title: Selecting what .NET version is to use
subtitle: It works on my machine but the pipeline is failing
slug: select-the-dotnet-ver-to-use
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: azure, devops, continuous-integration, net, vscode, csharp
cover: https://user-images.githubusercontent.com/958227/229709054-87d8a9cc-acfc-4b0b-a1b1-59e5a87051f3.jpeg?auto=compress
domain: keepontruckin.hashnode.dev
---

I'm a little procrastinated, not gonna lie. The Alertbox Inc. (@[alertboxinc](@alertboxinc)) pipelines page is open on my work PC and one of the CI pipelines is broken. What a SHAM!

It's a little weird since the pipeline logs show the target .NET version isn't available, but the changeset seems to work locally. Not only are there no tests failing, but there are no errors at all. For the record, Alertbox has several hundred git repos and [pipelines they run quite often on hosted agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#use-a-microsoft-hosted-agent). Unfortunately, not all code repos are on [the latest LTS version of .NET](https://versionsof.net/core/), so they require several versions of .NET installed locally, and you can only boil so many oceans at the same time.

<img class="img-responsive img-comic" alt="2017-06-10_budget_money_spending_projects_upgrades_technology_software_engineering" src="https://user-images.githubusercontent.com/958227/229605458-5b7203ba-af2a-4bbc-bbf6-6c03cbf2c257.gif" width="100%">

Maybe they don't have [end-of-life .NET versions](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core#lifecycle) on pipeline agents? Yup! [The Azure Pipeline team is upgrading agents](https://devblogs.microsoft.com/devops/upgrade-of-net-agent-for-azure-pipelines/). I alsmost want to migrate the code repo to the latest LTS version of .NET, say, replacing `TargetFramework` from `net6.0` and run `dotnet list package --outdated` to list nuget packages to upgrade, like they do for the reliable services code repos. You'd be amazed how tricky it is to have several versions of .NET running locally since [the `dotnet` driver selects the latest SDK installed locally](https://learn.microsoft.com/en-us/dotnet/core/versions/selection#the-sdk-uses-the-latest-installed-version) commands, and not from the entry-project / solution in the code repos that are in, so it's not such a great idea and probably not even pragmatic since it would be impossible to know what code repos are to migrate without going thru all of them one at a time completely. And I'm way too lazy to go thru all of that *Jazz* and migrate each one, so it's definitely necessary to break out that pipeline and modify the YAML script to install .NET.

```yaml
#.azuredevops/steps/build.yaml
steps:
# Install .NET from global.json
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    useGlobalJson: true
```

All I need to do is add a [UseDotNet task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/use-dotnet-v2?view=azure-pipelines) as a prerequisite for build step. But there are more problems. They require a *global.json* file to figure out the SDK version to install.

```json
// global.json
{
  "sdk": {
    "version": "5.0.17",
    "allowPrerelease": false,
    "rollForward": "latestMinor"
  }
}
```

I can run `dotnet new globaljson` to create the file to mention the minimal SDK version. Then I can [run the pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/manage-pipelines-with-azure-cli?view=azure-devops#example) and the `UseDotNet@2` task will install .NET on pipeline agents for me. Excellent! Moving right along.

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
# Check for status
az pipelines runs list \
   --top 3
   --status all
   --branch kp/fix-23-dotnet-not-found \
   --output table
```

It's taking a few tries to [get the pipelines green](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/runs?view=azure-devops#example) since [some of the dependencies and C# features](https://www.tiktok.com/@shanselman/video/7177520596736183598) don't seem to work anymore. Time to get myself a matching dev container for `net5.0` and I'm done!



/KP