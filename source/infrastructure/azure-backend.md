# Azure Infrastructure
OWASP leverages a few Azure technologies in order to automate and support business processes.  This page will serve to document the backend jobs that exist today.

```{admonition} Azure Subscription Details
  Subscription Name: Microsoft Azure Sponsorship
  Subcription ID: d368e2aa-f5b7-4b60-9bfb-4ca652df453b
```
## Automation Account

The automation account hosts runbooks written in Python which are responsible for integrating and updating data amoungst the various systems supporting OWASP. These are all scheduled backend jobs that run nightly.

```{admonition} Automation Account & Runbook Details
 Azure Resource Group: Staff-Administration
 Automation Account Name: OWASPAutomation
 Github Repo: [OWASP-Foundation/Automations](https://github.com/OWASP-Foundation/Automations)
```
## Automation Account

### Summary of Runbooks
| Runbook Name | Summary of Functionality |
|:---|:---|
| ChapterNightly |Updates the _data/chapters.json file for the website based on the info obtained from all the GitHub chapter repositories|
| CommunityEventsNightly |Updates the _data/communityevents.json file for the website based on the info obtained from GitHub chapter repositories and Meetup|
| EventsAndCommitteeNightly |Updates the _data/committees.json and _data\revents.json file for the website based on the info obtained from GitHub revent and committee repositories|
| EventsAndCorpMembersNightly |Copies _data/corp_members.yml to assets/sitedata/corp_members.yml and _data/events.yml to assets/sitedata/events.yml|
| LeadersNightly |Updates the _data/leaders.json file for the website based on the info obtained from all the GitHub repositories|
| MemberEmailCleanupNightly |Suspends Google accounts based on info obtained from various systems|
| ProjectsNightly |Updates the _data/projects.json file for the website based on the info obtained from GitHub project repositories|
| RepositoryNightlyBuild |Gathers data for all Github repositories and updates the RepoEntries table in the owaspadmin941c Azure Storage Account  |
| UpdateGoogleLeaderGroups |Adds all the leaders into the proper Google leader groups|

![dataflow](owasp-runbooks-data-flow.jpg)

## OWASP API

The OWASP API is implemented using Azure Functions written in Python.  This API supports a lot of the functionality on the website and in Slack.

```{admonition} Azure Function Details
 Azure Resource Group: Staff-Administration
 Function Name: owaspadmin
 Github Repo: [OWASP-Foundation/owaspadmin-azurefunctions]([https://github.com/OWASP-Foundation/Automations](https://github.com/OWASP-Foundation/owaspadmin-azurefunctions))
```


### Summary of API Endpoints
| Functional Area | Function | Type | URLs/Slack command | Notes |
|:---|:---|:---|:---|:---|
| **Membership** |||||
|| BillingManagement | HTTP | https://owasp.org/manage-membership/ | Returns basic info indicating that an email with link will be sent to the email address on file |
|| CreateCheckoutSession | HTTP | https://owasp.org/membership/ https://owasp.org/manage-membership/ https://owasp.org/membership/force_majeure/ (not really used here) <br>https://owasp.org/donate/ | Handles membership -> Stripe -> Back to Azure function StripeWebhookProcessor <br><br>Shows member information for managing subs and provisioning email (etc) <br><br>Handles donations -> Stripe -> Back to Azure function StripeWebhookProcessor |
|| CreateLeaderMembership | HTTP | https://owasp.org/membership/ https://owasp.org/membership/force_majeure/ (not really used here) | Handles creating the free leader memberships |
|| CancelSubscription | HTTP | https://owasp.org/manage-membership/ | Cancels membership subscription |
|| get-member-info | HTTP | https://members.owasp.org/ | Gets the membership info displayed on index.md in the Member Portal |
|| HandleAddMembers | HTTP | https://admin.owasp.org | Used when adding members from a CSV file from conferences |
|| process-handle-add-members | Queue || Processes the queue item created from HandleAddMembers |
