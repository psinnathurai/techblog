---
title: "How to Create Your Own Blog Using Hugo and Azure Static Web Apps"
date: 2024-05-30T01:50:32+02:00
draft: false
ShowToc: true
author: 'Pratheep Sinnathurai'
comments: true
---

# Overview
Since May 2024, I have been a Microsoft MVP in Azure Hybrid and Migration. Many people have asked me, "Hey Pratheep, how did you start your blog?" To avoid spending too much time on questions like "How should my page look?" and "Which font should I use?" I initially published my first blogs on Medium.com. However, I always wanted to create my own blog site.

Recently, I noticed that some Microsoft-provided websites were using Hugo, a tool I hadn't heard of before. Intrigued, I decided to invest some time in learning and experimenting with Hugo. In this blog post, I will show you how to create your own blog using Hugo and Azure Static Web Apps. The best part? It's all free.

![Meme of the Days](../meme.jpg#center)

# What is Hugo
Hugo is a popular open-source static site generator written in Go. It allows you to create fast and flexible websites effortlessly. With Hugo, you can author content in Markdown or HTML and generate a static website that can be hosted on any web server.

One issue that bothered me on Medium.com was the incorrect formatting of PowerShell command syntax. With Hugo, I can use Markdown, which provides better support for coding syntax.

I discovered Hugo when I learned that Microsoft's [Azure Verified Module Website](https://azure.github.io/Azure-Verified-Modules/) was built using it. Intrigued by its capabilities, I decided to give Hugo a try myself.

# Installation
## Hugo 
To get started with Hugo, you'll need to install it on your machine. I used a Windows 11 device. On Windows 11, you can use WinGet to install Hugo:

```bash
winget install -e --id Hugo.Hugo.Extended
```

## Create a Hugo App
Now that we have Hugo installed, we can use the Hugo CLI to create a new app:

```bash
hugo new site techblog
```
Navigate to the newly created Hugo app directory:

```bash
cd .\techblog\
```

Initialize a new local Git repository:

```bash
git init
```

Rename the current branch to "main":

```bash
git branch -M main
```

Now, let's select a theme for your new website. You can browse a complete list of available themes here: [Hugo Themes](https://themes.gohugo.io/).

If you want to customize a theme, you can fork it and make your modifications. Alternatively, you can update the theme whenever the repository is updated.

Personally, I prefer the GitHub Style theme, so I chose that one. To add the GitHub Style theme as a submodule to your current Git repository and place it in the `themes/github-style` directory, use the following command:

```bash
git submodule add https://github.com/MeiK2333/github-style themes/github-style
```

In the `hugo.toml` file, change the theme to the new theme:

```toml
theme = "github-style"
```

Commit the changes:

```bash
git add -A
git commit -m "first commit"
```

## Deploy Your Hugo App to GitHub
To save and later deploy our Hugo app, we'll create a GitHub repository. If you don't have a GitHub account yet, you can create one [here](https://github.com/signup).

Create a new repository using this [link](https://github.com/new). You can choose to make it public or private.

Add your GitHub repository as a remote to your local repository. Replace the URL with your GitHub repository's URL:

```bash
git remote add origin https://github.com/psinnathurai/techblog
```

Push your local repository to your remote repository:

```bash
git push --set-upstream origin main
```

## Deploy a New Static Web App
### Troubleshooting
We could easily create a Static Web App in Azure using the portal, but I prefer to use Bicep to deploy our Azure resources. After some testing, I was **NOT** able to create the Static Web App using Bicep, but I will show you where I encountered issues.

**Please do not follow the next steps until I mention it's safe again.**

First, use the following Bicep file and parameter file to create a Web App:

`main.bicep`:
```bicep
param name string
param location string
param sku string
param repositoryUrl string
param branch string
param appLocation string
param apiLocation string
param appArtifactLocation string
param areStaticSitesDistributedBackendsEnabled bool

module staticSite 'br/public:avm/res/web/static-site:0.3.0' = {
  name: name
  params: {
    name: name
    location: location
    sku: sku
    repositoryUrl: repositoryUrl
    branch: branch
    appSettings: {
      APP_LOCATION: appLocation
      API_LOCATION: apiLocation
      APP_ARTIFACT_LOCATION: appArtifactLocation
      ARE_STATIC_SITES_DISTRIBUTED_BACKENDS_ENABLED: areStaticSitesDistributedBackendsEnabled
    }
  }
}
```

`params.bicep`:
```bicep
using './main.bicep'

@description('The name of the static site')
param name  = 'sinnathuraitechblog'

@description('The location of the static site')
param location  = 'westeurope'

@description('The SKU of the static site')
param sku  = 'Free'

@description('The repository URL of the static site')
param repositoryUrl  = 'https://github.com/psinnathurai/techblog'

@description('The branch of the repository for the static site')
param branch  = 'main'

@description('The app location for the static site')
param appLocation  = '/'

@description('The API location for the static site')
param apiLocation  = ''

@description('The app artifact location for the static site')
param appArtifactLocation  = 'public'

@description('Whether distributed backends are enabled for the static site')
param areStaticSitesDistributedBackendsEnabled  = false
```

Once our Static Web App in the Azure portal is ready, we need to copy the deployment token for authentication in GitHub.

Copy the deployment token:

![First Step](../01.png#center)

1. Go to your previously created GitHub repository.
2. Click on Settings.
3. Under Secrets and variables, click on Actions.

![Second Step](../02.png#center)

Add a repository secret named `AZURE_STATIC_WEB_APPS_API_TOKEN` and use the copied deployment token as the value.

![Third Step](../03.png#center)

For automation, add a GitHub Action that will run when there is a new change in the repository and publish the latest update to our website.

1. Click on Actions.
2. Create a new workflow.
3. Paste the YAML code below.

![Fourth Step](../04.png#center)

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0
          lfs: false
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for Github integrations (i.e. PR comments)
          action: "upload"
          ###### Repository/Build Configurations - These values can be configured to match your app requirements. ######
          # For more information regarding Static Web App workflow configurations, please visit: https://aka.ms/swaworkflowconfig
          app_location: "/" # App source code path
          api_location: "" # Api source code path - optional
          output_location: "public" # Built app content directory - optional
          ###### End of Repository/Build Configurations ######

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
      - name: Close Pull Request
        id: closepullrequest
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          action: "close"
```

After the workflow runs, I received the following error:

![Fifth Step](../05.png#center)

I couldn't find a solution for this issue. I suspect it might be because, in the Azure portal, we can specify that we're using Hugo, which isn't possible in the Bicep deployment for a Static Web App.

### Using the Azure Portal
**It is now safe to follow these steps.**

Now, we'll create the static website in the Azure portal.

1. Go to the Azure Portal.
2. Search for "Static Web Apps".
3. Click on "Create".

![Sixth Step](../06.png#center)

1. Select the subscription and the resource group.
2. Enter the name.
3. Select "Free" as the plan type.
4. Choose GitHub as the source and select your GitHub account (the deployment token will be added to the repository in the background).
5. Select the organization, repository, and branch.

![Seventh Step](../07.png#center)

Under "Build Presets," select Hugo. This step is missing in the Bicep deployment for a Static Web App. Click "Review and Create" and then "Create".

![Eighth Step](../08.png#center)

Wait until the GitHub Action runs are finished. You can check the status of the actions in the GitHub repository under Actions.

# Conclusion

Creating your own blog using Hugo and Azure Static Web Apps is both straightforward and rewarding. 
By leveraging Hugo's  static site generation and Azure's hosting solutions, you can build and deploy a professional blog with ease. 
This method gives you greater control over your site's appearance and functionality, ensuring fast performance and flexibility.

In this guide, we covered the steps to install Hugo, create a new site, select and customize a theme, and deploy your site to GitHub and Azure Static Web Apps. 
We've also discussed the benefits of using Markdown for better coding syntax support and the simplicity of automating deployments with GitHub Actions.

Although we faced some challenges with the Bicep deployment, using the Azure portal proved to be an effective alternative. 
Following these steps, you can now focus on creating high-quality content for your blog, knowing that your site is built on a solid foundation.

I hope this guide has been helpful and inspires you to start your own blog. If you have any questions or run into any issues, feel free to reach out. Happy blogging!