# Infrastructure Repository and Pipeline

## Introduction

You should now have Completed the Following things:
1. Setup your GitHub Account
2. Setup the Example Code in your Account
3. Added the Repository Secrets to the Example Code

Next you will get an Overview over the Example Code to Setup your first own Website on Azure. This will only be the most basic Infrastructure Setup. In a later Step we will then alter the Websites Content.

If you want to learn more about the concept of a pipeline you can do it here:

[https://docs.github.com/en/actions/quickstart](https://docs.github.com/en/actions/quickstart)


# 1. Setting up the Infrastructure Pipeline

The first Step in Creating a Pipeline with GitHub Actions would normally be to Select a Template and Start from there.

To do so you would go to Actions in your Code Repository and select an appropriate Template or start from Scratch with an empty one.

In our Example we already Created the necessary Files. So to say our own Template.

Your first Task is to go into your Repository and look at the Following file.

>_Warning: The formatting of YAML (yml) files is based on spaces and tabs and therefore the following lines should be copied with care.
> It is advised to use Visual Studio Code to validate the copied file._
> <br> [How to work with Git Locally](/01.5_SetupGit.md)


`#File: .github/workflows/azure_infra.yml`
```
on: 
  workflow_dispatch:

name: infra

jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
    
    - name: Azure Login
    uses: azure/login@v1
    with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Azure CLI script
    uses: azure/CLI@v1
    with:
        inlineScript: |
        az appservice plan create -g ${{ secrets.rg }} -n ${{ secrets.asp }} --is-linux --number-of-workers 1 --sku B1
        az webapp create -g ${{ secrets.rg }} -p ${{ secrets.asp }} -n ${{ secrets.webapp }}  --runtime "node|10.14"
```

## Azure Cli

The Pipeline we are Proposing here is using the Azure CLI to create the App Service Plan and the WebApp on Azure.

Azure CLI Docs: 
<br> https://docs.microsoft.com/de-de/cli/azure/what-is-azure-cli

Azure AppServicePlan and WebApp: 
<br> https://docs.microsoft.com/en-us/azure/app-service/overview

## Triggers

The Code Starts by using "`on: workflow_dispatch`" which means one of the Triggers to Start this Pipeline is to do it Manually.

There are many Automatic triggers you can use, to learn more about Triggers check this:
https://docs.github.com/en/actions/reference/events-that-trigger-workflows#workflow_dispatch

## Pipeline Name

Next we Specify the Name of our GitHub Action in our Example "`name: infra`".

## Code runner

After we define the Triggers and the Name of the workflow we need to Specify its "`jobs`".
In our Example we only need one Job "`deploy`".

Next we Specify what image we Expect our job to run on:
"`runs-on: ubuntu-latest`"

```
jobs:

  deploy:
    runs-on: ubuntu-latest
    
```

To learn more about the Workflow Syntax and Jobs visit:
https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobs


## Authentication

The first thing the Workflow will do is to Authenticate via the Build in Feature "`uses: azure/login@v1`" and the use of the Connection String in Stored in "`secrets.AZURE_CREDENTIALS`"

```
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
```

## Deployment of AppService and WebApp

Next the Script creates with the Build in Azure CLI "`uses: azure/CLI@v1`" the AppService Plan and the corresponding WebApp. 

```
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az appservice plan create -g ${{ secrets.rg }} -n ${{ secrets.asp }} --is-linux --number-of-workers 1 --sku B1
          az webapp create -g ${{ secrets.rg }} -p ${{ secrets.asp }} -n ${{ secrets.webapp }}  --runtime "node|10.14"
```

AppService with Azure CLI
<br> https://docs.microsoft.com/de-de/cli/azure/appservice/plan?view=azure-cli-latest

WebApp with Azure CLI
<br> https://docs.microsoft.com/de-de/cli/azure/webapp?view=azure-cli-latest#az_webapp_create


You will see there is a tiny difference in your File to this Code Snipped.
Just Copy over the Missing Part.

# 2. Run your Pipeline

After you Set up your Secrets and fixed the Code in your Repository.
You can try to run your Workflow.
To do so go to Actions and select the Infra workflow on the Left site.

Now Select Run workflow on the Right side.

<br><img src="./images/runWorkflow.PNG" width="800"/><br>

## Workflow Progress

Wait for your Workflow to finish.
If the Task does not run through you may ask one of us to Help you out.
## Check your WebApp is online after approx. 5 minutes

https://`[yourWebAppName]`.azurewebsites.net/

You should see a &quot;Hey, Node developers&quot; welcome screen.

Congratulations, you have deployed your first WebApp infrastructure.
 Now, you can go ahead and deploy some code to your WebApp.