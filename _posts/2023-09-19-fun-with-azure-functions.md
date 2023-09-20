---
layout: post
title: Fun with Azure Functions
date: 2023-09-19 17:51 -0700
categories: [Azure, Functions]
tags: [azure,python,function,keyvault,cicd]
---
I've been working on some automation for payroll recently. We use Square for our employees to log their work hours. This allows them to clock-in and -out from the point of sale Square stand.

**The Problem** On every payroll run, I would manually record the clock-in and -out times in Excel. With the help of some formulas, Excel would inform me of all employees' normal and overtime hours. This information is then referenced when running payroll in QuickBooks. Adding to the pain was the lack of a report in Square to summarize these numbers.

**The Idea** Develop a quick Python script to call into Square APIs, retrieve all employee's work hours for the pay period. The script would then calculate the totals and email me the numbers. This saves me from the manual data entry, provides for an audit check, and keeps a historical record in my Gmail.

Thus, [timesheet-calc](https://github.com/zPugMan/timesheet-calc) was created.

## Azure
### Initial Setup
With some initial code setup, I was ready to create a CI/CD pipeline. In order to do this, I had to create some Azure resources first. Using the Azure CLI:
```
az login
az group create --name <resource_group> --location "westus2"
az storage account create --name <storage_account>
az functionapp create --name <func_name> --resource-group <resource_group> --storage-account <storage_account> --consumption-plan-location "westus2" --runtime "python" --runtime-version "3.9" --os-type Linux
```
Using Azure Core tools, I then created an initial shell for the function using a predfined template. The templates include the workflow action for automatic deployment.

### Deployment Secret
The workflow action expects a Github secret `AZURE_FUNCTIONAPP_PUBLISH_PROFILE`. To set this secret:
* Login to Azure portal
* Navigate to the Azure Function App
* Click on *Get publish profile*

![Azure function app](/assets/posts/2023/09/2023-09-19%20AzureFunctionApp.png)

* The result of clicking will trigger a download.
* Using a text editor, copy and paste the entire contents as a new *Action* secret within your GitHub repository

![Github Action Secret](/assets/posts/2023/09/2023-09-19%20GithubSecrets.png)

Your repo is now setup for automatic deployments via the `Azure/functions-action@v1` module.