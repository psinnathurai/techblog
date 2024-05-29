+++
title = 'How to create your own Blog using Hugo and Azure Static Web Site'
date = 2024-05-27T16:35:32+02:00
draft = false
+++

# Overview
Since May 2024, I have been a Microsoft MVP in Azure Hybrid and Migration. Since then, many people have asked me, "Hey Pratheep, how did you start with your blog?" The answer is that to avoid spending too much time on questions like "How should my page look?" and "Which font should I use?" I initially published my first blogs on medium.com. However, I always wanted to create my own blog site.

Recently, I noticed that some Microsoft-provided websites were using Hugo, a tool I hadn't heard of before. I decided to invest some time in learning and experimenting with Hugo. In this blog post, I will show you how to create your own blog using Hugo and Azure Static Web Apps. The best part? It's all free.

![Meme](/images/blog/20240529/meme.jpg#center)

# What is Hugo
Hugo is a widely used open-source static site generator written in Go. It enables the creation of fast and flexible websites effortlessly. With Hugo, you can author content in Markdown or HTML and generate a static website that can be hosted on any web server.

One issue that bothered me on Medium.com was the incorrect formatting of PowerShell command syntax. With Hugo, I can use Markdown, which provides better support for coding syntax.

I discovered Hugo when I learned that Microsoft's Azure Verified Module Website (https://azure.github.io/Azure-Verified-Modules/) was built using it. Intrigued by its capabilities, I decided to give Hugo a try myself.


# Installation
## Hugo 
To get started with Hugo, you'll need to install it on your machine. In my case I have used a Windows 11 Device.
On Windows 11 you can use WinGet to install Hugo.

Once you have Hugo installed, you can create a new Hugo site by running the following command in your terminal:

```bash
winget install -e --id Hugo.Hugo.Extended
```

## Create a Hugo App
Now that we installed Hugo, we can use the Hugo CLI to create a new App.

```bash
hugo new site techblog
```
Change the Directory to the newly created Hugo App.

```bash
cd .\techblog\
```

Create a new local Git Repository

```bash
git init
```

This command renames the current branch to "main" in a Git repository.

```bash
git branch -M main
```

Now, let's select a theme for your new website. You can browse a complete list of available themes here: [Hugo Themes](https://themes.gohugo.io/).

If you want to customize a theme, you can fork it and make your modifications. Alternatively, you can update the theme whenever the repository is updated.

Personally, I prefer the GitHub Style theme, so I chose that one.

To add the GitHub Style theme as a submodule to your current Git repository and place it in the `themes/github-style` directory, use the following command:

```bash
git submodule add https://github.com/MeiK2333/github-style themes/github-style
```

In the hugo.toml file you need to change the Theme to the new Theme.

```bash
theme = "github-style"
```


Now we need to commit the Change.

```bash
git add -A
git commit -m "first commit"
```
## Deploy your Hugo App to GitHub
To save and later deploy our Hugo App we will create a GitHub Repository.
If you don't have a GitHub Account until now than it's time to create a GitHub Account. 

Click on the following Link to create your GitHub Account

[Create a GitHub Account](https://github.com/signup)

After we have our GitHub Account you can create your new Repository using the following Link.

You can create a public or a private Repository but that's up on you.

[Create a GitHub Repository](https://github.com/new)

Add your Github Repository as a remote to our local created Repository.
Change the Link of your GitHub Repository 

```bash
git remote add origin https://github.com/psinnathurai/techblog
```

Push your local repository to your Remote repository.

```bash
git push --set-upstream origin main
```

##Deploy a new Static Web App
### Troubles
We could do the easy way and create a Static Web App in Azure using the Portal.
But I think that's a little bit to easy for us.

As always I like to find out if it is possible to use Bicep to deploy our Azure Resource.
After some testing I was **NOT** able to create the Static Web App unfortunantely but I still would like to show where I failed.

**Please do not follow the next steps until I mention it fine again.**

First of all we need to use the following Bicep File and Bicep Param File to create a Web App.

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

The following Part is the Parameter Bicep File.

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

Once our Static Web App in the azure Portal is ready we need to copy the Deployment Token for the authentcation in GitHub.

Copy the Deployment Token

![First Step](/images/blog/20240529/01.png#center)

1. Go to your previously created GitHub Repository
2. Click on Settings
3. Under Secrets and variables click on Actions

![Second Step](/images/blog/20240529/02.png#center)

Add a Repository Secret with the Name AZURE_STATIC_WEB_APPS_API_TOKEN and as Value the previously copied Deployment Token.

![Third Step](/images/blog/20240529/03.png#center)

For the Automation we will now add a GitHub Action which will run when there is a new change in the Repository and publish the latest update to our Website.

1. Click on Action
2. Create a new Workflow
3. Paste the YAML Code below


![Fourth Step](/images/blog/20240529/04.png#center)


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

After the workflow run through I received the following Error:

![Fifth Step](/images/blog/20240529/05.png#center)

I was not able to find a solution for this Issue. I guess the Issue is related to the fact that in the Azure Portal it is possible to define that we are using Hugo and in Bicep this is not possible .

### Do it in the Azure Portal
**It is now safe again to follow the steps.**

Now we will just create the Static Website in the Azure Portal.

1. Go to the Azure Portal
2. Search for Static Web Apps
3. Click on Create

![Sixth Step](/images/blog/20240529/06.png#center)

1. Select the Subscription and the Resource Group
2. Select the Name
3. Select Free as Plan Type
4. Select as Source GitHub and Select your GitHub Account (In the Background it will add the Deployment Token in the Repository which you will select)
5. Select the Organization, Repository and the Branch

![Seventh Step](/images/blog/20240529/07.png#center)

Under Build Presets select Hugo. This is the step which is missing in my opinion in the Bicep Deployment for a Static Web App.
Click Review and Create and Create

![Eigth Step](/images/blog/20240529/08.png#center)

Wait until the GitHub Action runs are finished.
You can check the of the Actions in the GitHub Repository under Actions.

