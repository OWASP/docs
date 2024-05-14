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
