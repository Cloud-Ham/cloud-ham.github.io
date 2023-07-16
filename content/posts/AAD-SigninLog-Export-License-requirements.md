---
title: "AAD SigninLog Export License Requirements"
date: 2023-07-10T00:46:17Z
draft: true
---

# Quick Intro

To get Sign-in Logs exported to another tool requires Azure AD Licensing. This blog post is going to get into more detail on two questions I had when I first started:

* If there are no licenses, can the sign-in logs still be exported to an Azure tool (Log analytics)?
* What does tenant licensing mean when only users can be licensed?

# Can you export Azure AD SigninLogs to a native location without licensing?

Yes! The log exporting capabilities function for the following resources without Azure AD Licensing:

* Azure Storage Account
* Log Analytics Workspace
* Sentinel-enabled Log Analytics Workspace

The following did not receive Sign-in logs during testing

* Event Hub

Below are screenshots of each step of the process: Establishing connections in Diagnostic settings of Azure AD, and log activity from each resource.

[IMAGE1-DIAG]
*Image of diagnostic settings configuration*

[IMAGE2-StorageAcct]
*Storage Account JSON output*

[IMAGE3-SentinelLAW]
*Sentinel-enabled Log Analytics search result*

[IMAGE4-RegularLAW]
*Regular Log Analytics search result*

[IMAGE5-EventHub]
*Event Hub metrics*

