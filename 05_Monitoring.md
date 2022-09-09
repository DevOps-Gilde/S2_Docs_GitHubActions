# Monitoring

## General

This is an optional extra task you can work on. Focus of the task is monitoring your demo web app. App Service is deeply integrated with App Insights. App Insights is a feature within Azure Monitor. To monitor your Web App you have to create a new App Insights resource that is linked to use the **[Kusto query language](https://docs.microsoft.com/de-de/azure/data-explorer/kusto/concepts/)** to retrieve the **telemetry** data. The picture below illustrates this:
<br><img src="./images/MonitoringOverview.png" width="400"/>

To watch your **telemetry** in App Insight we will use the portal. We created special users for portal access. Contact one of us to get the credentials for it. The next two big steps are:
1. Extend your infra pipeline to create an App Insights resource and link it to your Web App
2. Grab a user and monitor your Web App via the Azure portal 

The next chapters will detail these two major steps.

## Extend your infra pipeline

To create the additional components we will extend the "Azure CLI script" step in our existing infra pipeline. The comment in the file below marks the place where code needs to be added.

`#File: .github/workflows/azure_infra.yml`
```
on: 
  workflow_dispatch:

name: infra

jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
    ...
    - name: Azure CLI script
    uses: azure/CLI@v1
    with:
        inlineScript: |
        ...        
        # That is the code you have to add
```

To run Azure CLI commands we first have to create first the application insights extension.
```
az extension add -n application-insights
```

With the extension in place we can now create the application insights resource. In contrast to the command for the Web App we have to specify the location. We will use the same location as the location of the resource group. To achieve we will store the location in a shell variable `locName`. The assigned value will be the result of an azure cli command. Therefore, the command needs to embedded in backticks.
```
locName=$(az group show --name ${{ secrets.RG }} --query location --output tsv)
```
An interesting part is the `--query` argument. It is required because  `az group  show` returns the complete JSON representation of the resource group. But we need only the value of the property location. `--query` argument allows you to apply a JMESPath filter expression to the returned JSON. In our case the filter expression is just the name of the property. The `--output tsv` specifies the output format which does not contain any formatting. Both options together yield the location we are looking for.

As with the Web App the name of the App Insights resource needs to be unique since you are all working in the same resource group. Therefore we will use the same name just with an additional prefix.
```
az monitor app-insights component create --app apimon${{ secrets.WEBAPP }} --location $locName --kind web -g ${{ secrets.RG }} --application-type web --retention-time 30
```

Establishing a link requires a reference in the Web App that represents our App Insights resource. App insights provides a so called *instrumentation key* that can be used as reference. The following command stores this key in a variable instrumentationKey. 
```
instrumentationKey=(az monitor app-insights component show --app apimon${{ secrets.WEBAPP }} --resource-group ${{ secrets.RG }} --query  "instrumentationKey" --output tsv)
```
The command follows the same pattern as above where we stored the location of the resource group in a variable. The only difference here is the different attribut we are interested in.

Now let's use our variable to link the Web App to App Insights. This requires updating a few **application** settings inside the Web App. One is of course our **instrumentation key**. The command below shows how that is done. 
```
az webapp config appsettings set --name ${{ secrets.WEBAPP }} --resource-group ${{ secrets.RG }} --settings APPINSIGHTS_INSTRUMENTATIONKEY=$instrumentationKey APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=$instrumentationKey ApplicationInsightsAgent_EXTENSION_VERSION=~2
```

## Monitor your Web App in App Insights

First contact one of us to grab a user along with the credentials.

To login to the Azure Portal enter "portal.azure.com" in the browser. Use the credentials given to you to login.

Navigate in the portal to the resource group "ws-devops".
<br><img src="./images/MonitorApInsightsRG.png" width="400"/>

Open your App Insights instance and by clicking on the entry representing your instance. Click in the main menu on the left hand side on "Logs". Click away the welcome and query screen.
<br><img src="./images/MonitorApInsightsLogs.png" width="400"/>

You will see detail screen where you can enter your Kusto query. The screenshot below explains a bit the major controls.
<br><img src="./images/MonitorApInsightsKusto.png" width="400"/>

The simplest Kusto query is just to state the name of the log table you are interested in as shown below. If you execute this query by pressing the previously outlined button you get the result. Note that there is a small delay until the data is visible in App Insights (app. 1 minute).
```
requests
```

Let's run now a bit more sophisticated query that counts all failed requests (In our case all requests where the HTTP status code does not equal 200). We will extend our query by two keywords:
1. where - as in SQL this allows us to filter the records by a certain criteria
2. summarize - Allows you to run an aggregating operation such as count
The snippet below shows the full query. As you can see both keywords are chained by the pipe operator:
```
requests
| where resultCode != 200
| summarize count()
```
The result of the query should output the number of failed requests now. With these two queries we just touched the basics of monitoring in App Insights. Kusto provides much more capabilities and you can also use it to trigger automatic actions based on the query result.
