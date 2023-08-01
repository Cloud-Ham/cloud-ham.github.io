---
title: "Threat Modelling - Azure Logic Apps"
date: 2023-08-15T17:13:05Z
draft: true
---

# Introduction

Azure Logic Apps are a way to configure automation workflows that can perform a ton of actions such as; manage Sentinel incidnets (Comment, Close, Update, etc.), perform Azure Resource Manager actions (Create Resource, Stop Resource, Delete Resource, etc.), interact with other APIs, and so much more. They also don't require knowledge of code to get started as the main designer is a visual engine. Easy to get started, hard to master, but also... what's the threat landscape look like on these?

I am by no means the first person to do this, but I wanted to draw it out for myself as something to investigate in any Azure environments I find myself in later. What threats are present? How easy are adversarial goals accomplished with them? What are some of the worst case scenario actions that can occur?

For an initial hypothesis;

* Permissions in the Azure environment are the most critical. The people who maintain logic apps should be trusted. There needs to be a separation of people who can edit logic apps, versus those who can run logic apps. Furthermore, whatever the means is of authentication (User, Service Principal, or Managed Identity) must also have permissions tightly limited to exactly what is needed.
* Redirecting a Logic App action that requires authentication details to a malicious endpoint will be the easiest or most efficient way of compromising an account's access.

# Logic App Permissions

## Logic App Contributor

These are the current permissions allowed in Logic App Contributor as of 7/30/2023. (Click here for current)

```
"Microsoft.Authorization/*/read",
"Microsoft.ClassicStorage/storageAccounts/listKeys/action",
"Microsoft.ClassicStorage/storageAccounts/read",
"Microsoft.Insights/alertRules/*",
"Microsoft.Insights/metricAlerts/*",
"Microsoft.Insights/diagnosticSettings/*",
"Microsoft.Insights/logdefinitions/*",
"Microsoft.Insights/metricDefinitions/*",
"Microsoft.Logic/*",
"Microsoft.Resources/deployments/*",
"Microsoft.Resources/subscriptions/operationresults/read",
"Microsoft.Resources/subscriptions/resourceGroups/read",
"Microsoft.Storage/storageAccounts/listkeys/action",
"Microsoft.Storage/storageAccounts/read",
"Microsoft.Support/*",
"Microsoft.Web/connectionGateways/*",
"Microsoft.Web/connections/*",
"Microsoft.Web/customApis/*",
"Microsoft.Web/serverFarms/join/action",
"Microsoft.Web/serverFarms/read",
"Microsoft.Web/sites/functions/listSecrets/action"
```

Verified actions when configured to a user at Resource Group scope:<br>
* User is able to access Storage Account key and inherent the key's permission. Capability exists to create/update/delete existing storage account containers. See section titled, "Adversary case - Storage Account manipulation". (If AAD auth is required to the Storage Account, it blocks this access)
* User cannot create Standard Logic Apps (Requires additional permission)
* User can update or delete Standard Logic Apps
* User can create/update/delete Consumption Logic Apps
* User cannot see the Diagnostic Settings options (I'm going to consider this a bug. Click here to view the page from testing.)

An interesting note here, this role is very close to being able to create Standard Logic Apps. There's no permission to create the required App Service Plan, so that would have to be made by another user. But if an App Service Plan is available, the only missing permission to create a new Standard Logic App is the "Microsoft.Web/Sites/write" permission.

[LogicAppThreatModel-Aug2023-1]

## Logic App Operator

These are the current permissions allowed in Logic App Operator as of 7/30/2023. (Click here for current)

```
"Microsoft.Authorization/*/read",
"Microsoft.Insights/alertRules/*/read",
"Microsoft.Insights/metricAlerts/*/read",
"Microsoft.Insights/diagnosticSettings/*/read",
"Microsoft.Insights/metricDefinitions/*/read",
"Microsoft.Logic/*/read",
"Microsoft.Logic/workflows/disable/action",
"Microsoft.Logic/workflows/enable/action",
"Microsoft.Logic/workflows/validate/action",
"Microsoft.Resources/deployments/operations/read",
"Microsoft.Resources/subscriptions/operationresults/read",
"Microsoft.Resources/subscriptions/resourceGroups/read",
"Microsoft.Support/*",
"Microsoft.Web/connectionGateways/*/read",
"Microsoft.Web/connections/*/read",
"Microsoft.Web/customApis/*/read",
"Microsoft.Web/serverFarms/read"
```

These permissions are way easier to read as is. The user with this role can read most configurations surrounding the Logic App, but can disable, enable, and validate the status of the Workflow. This user will still have the capability to read the workflow, and we'll discuss more about dangers here in "Adversary case - Extract running secrets"

## A better custom role

Let's review these roles and find how we can build a better, more targeted role for the user of interest. 

__Logic App Contributor at Resource Group or higher scope__:<br>
* Storage account container manipulation (Can access Storage Account keys)
* Can create new logic apps
* Can edit existing logic apps
* Can disable logic apps
* Can disable logging at the configured scope
* Can access details of the running workflow (Secret exposure)
* Can exfiltrate secrets via workflow modification

__Logic App Contributor at Resource scope__: <br>
* Can edit existing logic app
* Can disable logic app
* Can disable logging on the logic app
* Can access details of the running workflow (Secret exposure)

# Adversary Case - Exfiltrate Key Vault secrets

A common case in Azure is that an application team working within a Resource Group will need an identity that is able to work with a variety of resources and possible tasks to automate their workload. For example, the Logic App may make changes to Azure resources within the resource group (Scale, Stop, Start) or access data (Upload, Download, Query). What *should* happen is that these permissions are separated into different managed identities and scoped down, but due to the overwhelming amount of detail and review that this would require, they are provided one managed identity that have excessive permissions.

An adversary with access to and permissions for the Logic App and Managed Identity can work to exfiltrate secrets from an Azure Key Vault. And most of all, it may blend in with operations if not monitored closely.