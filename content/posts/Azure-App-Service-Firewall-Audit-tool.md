---
title: "Azure App Service Firewall Audit Tool"
date: 2023-11-25T12:46:17Z
draft: false
---

# Intro

Earlier this year (Jan 2023), Ermetic (now Tenable Cloud Security) released a writeup on a vulnerability in Azure regarding the Kudu/SCM service on Azure App Services that allows for management of the application from Azure-side. It has been fixed from time of posting, but it resulted in the next question right off of the bat:

"Which of our App Services *were* vulnerable to this?"

Technically, any of them that didn't have restrictions. I highly recommend reading through Tenable Cloud Security's post on this at the following link:

[Tenable Cloud Security: EmojiDeploy: Smile! Your Azure web service just got RCE’d ._.](https://ermetic.com/blog/azure/emojideploy-smile-your-azure-web-service-just-got-rced/)

I originally wrote a script for this in PowerShell with a coworker's assistance, but wanted to revisit this and write it in Python.

# What is SCM or Kudu

Source Content Management (SCM) runs the .NET Kudu service that allows for reading details and making changes to an Azure App Service. With access to this site, you would be able to read potentially sensitive information such as Environment Variables or local files that may not be web-viewable. You could also SSH into the App Service to make changes. These may differ based upon how the App Service was deployed (Application code, via GitHub, container, etc.)

To get an idea of what the SCM service looks like, access your App service with an initial subdomain of "scm.azurewebsites.net". Place your app service's name in front of it like:

myappservice.scm.azurewebsites.net

This is what the SCM site looks like

![Resize](https://blog.cloud-ham.com/posts/appservfw-audit/kuduservice.png?width=766px)

# SCM Firewalls in Azure

For all Azure App Services (Excluding consumption Logic Apps), the firewall rules are in the same location: 

* Browse to the App Service in the Azure Portal
* Go to "Networking" on the left-hand menu
* Click on "Access Restriction"

There are two tabs: Main Site, and Advanced Tool Site (SCM site). From the "Advanced tool site", you have three options on how to configure it:

* Design it like a normal firewall, create IP rules and restrictions on access.
* Inherit the Main Site's rules. This works well if the entire App Service is supposed to be private and the existing firewall rules on the Main Site are sufficient to restrict access outside of the organization's network.
* Disable Public Access for the entire application, affects both Main Site and Advanced tool site.

__IMPORTANT__: If the Advanced tool site (SCM site) is inheriting the Main Site's rules, you will NOT see the Main Site's rules from within the Advanced Tool Site's rules.

The following is an example of the SCM firewall where Main Site rules are being inherited. Although it shows an "Allow All", the rules actually taking effect are in the 2nd screenshot.

![Resize](https://blog.cloud-ham.com/posts/appservfw-audit/kudufirewall.png?width=766px)

_Advanced tool site (SCM site) rules are inheriting main but not showing it_.

![Resize](https://blog.cloud-ham.com/posts/appservfw-audit/mainsite-rules.png?width=766px)

_The actual Main Site rules behind inherited_.

# What this script does

[Click here to go to the script in Github](https://github.com/Cloud-Ham/AzAppServiceFirewallAudit_python3)

Simply put, this script will loop through all subscriptions, resource groups, and then each individual App Service to output the firewall rules to a CSV file. Screenshot of example output below, and will likely change a few times soon.

![Resize](https://blog.cloud-ham.com/posts/appservfw-audit/csvoutput.png?width=766px)