# Azure Infrastructure
OWASP leverages a few Azure technologies in order to automate and support business processes.  This page will serve to document the backend jobs that exist today.

```{admonition} Azure Subscription Details
  Subscription Name: Microsoft Azure Sponsorship
  Subcription ID: d368e2aa-f5b7-4b60-9bfb-4ca652df453b
```
## Automation Account

The automation account hosts runbooks written in Python which are responsible for integrating and updating data amoungst the various systems supporting OWASP.

```{admonition} Automation Account & Runbook Details
 Resource Group: Staff-Administration
 Automation Account Name: OWASPAutomation
```
## Automation Account

### Summary of Runbooks
| Runbook Name | Summary of Functionality |
|:---|---:|
| ChapterNightly | |
| CommunityEventsNightly ||
| EventsAndCommitteeNightly ||
| EventsAndCorpMembersNightly ||
| LeadersNightly ||
| MemberEmailCleanupNightly ||
| ProjectsNightly ||
| RepositoryNightlyBuild ||
| UpdateGoogleLeaderGroups ||

![dataflow](owasp-runbooks-data-flow.jpg)
