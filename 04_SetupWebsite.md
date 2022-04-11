# Infrastructure Repository and Pipeline

## Introduction

You should now have Completed the Following things:
1. Setup your GitHub Account
2. Setup the Example Code in your Account
3. Added the Repository Secrets to the Example Code
4. Setup the initial WebApp on Azure

In this task we will Push our own Website Content to our freshly created Azure Website.

If you get Stuck here you may ask us for Help. We can also provide a Complete solution.
# 1. Setting up the Website Pipeline

>_Warning: The formatting of YAML (yml) files is based on spaces and tabs and therefore the following lines should be copied with care.
> It is advised to use Visual Studio Code to validate the copied file._
> <br>[How to work with Git Locally](/01.5_SetupGit.md)

`# File: .github/workflows/azure_webapp.yml`
```
on: 
  workflow_dispatch:

name: app

jobs:
  build-and-deploy:
    runs-on: **
    steps:
    # checkout the repo
    - name: 'Checkout Github Action' 
      uses: actions/checkout@master
      
    - name: Setup Node 10.x
      uses: actions/setup-node@v1
      with:
        **

    - name: 'npm install, build, and test'
      run: |
        **

    - name: Azure Login
      uses: azure/login@v1
      with:
        **

    - name: 'Azure webapp deploy'
      uses: azure/webapps-deploy@v2
      with:
        **
        package: ${{ github.workspace }}/app

```

Our Code seems to be corrupted maybe these notes of our Developer can help you fix it.

```
Where did i do this Login before?
```

```
To build the npm dependencies i need to Change Directory into the  app folder and then run
npm install
npm run build --if-present
npm run test --if-present
```

```
Run on ubuntu-latest
Then check out the Repository
The `node-version` should be "10.x"
```

```
azure/webapps-deploy@v2

Needs the `app-name` and a `package` location. Where did i store the webapps Name again?
```
# 2. Run your Pipeline

After you Set up your Secrets and fixed the Code in your Repository.
You can try to run your Workflow.
To do so go to Actions and select the app workflow on the Left site.

Now Select Run workflow on the Right side.

<br><img src="./images/runWorkflow.PNG" width="800"/><br>

## Workflow Progress

Wait for your Workflow to finish.
If the Task does not run through you may ask one of us to Help you out.
## Check your WebApp is online after approx. 2 minutes

https://`[yourWebAppName]`.azurewebsites.net/

You should see a &quot;Hey, Node developers&quot; welcome screen.

Congratulations, you have deployed your first WebApp infrastructure.
 Now, you can go ahead and deploy some code to your WebApp.