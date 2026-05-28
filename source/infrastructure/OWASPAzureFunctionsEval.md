# OWASP Azure Functions Evaluation

### BillingManagement
- Used in the currently redirected manage-membership page on owasp.org
- Enabled: Yes
- Invocations (30 days): 0
- Evaluation: Can be removed

### BillingSlackBot
- Slack commands: /stripe-details, /contact-details, /contact-lookup
- Enabled: Yes
- Invocations (30 days): 2
- Evaluation: Can be removed; not used for GlueUp memberships

### BuildRepositoriesEntry
- Originally built site JSON; replaced by OWASPAutomation
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### BuildSiteFiles
- Originally built site JSON; replaced by OWASPAutomation
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### BuildSiteFilesOrchestrator
- Originally built site JSON; replaced by OWASPAutomation
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### BuildSiteFilesStart
- Originally built site JSON; replaced by OWASPAutomation
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### BuildStaticWebsiteFiles
- Originally built site JSON; replaced by OWASPAutomation
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### BuildStaticWebsiteFilesTwo
- Originally built site JSON; replaced by OWASPAutomation
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### CancelSubscription
- Used to cancel subscriptions in manage-membership
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed; not used in GlueUp

### chapter-create
- Slack command to create a chapter
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed with new website

### chapter-process
- Processes chapter creation dialog
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed with new website

### chapter-lookup
- Slack command to give chapter details from Salesforce
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed with new website

### chapter-report
- Slack command to launch chapter report
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed with new website

### committee-create
- Slack command to create a committee
- Enabled: Yes
- Invocations: 0
- Evaluation: Keep while committees remain in GitHub

### committee-process
- Processes committee creation dialog
- Enabled: Yes
- Invocations: 0
- Evaluation: Keep; update to use Monday.com instead of Copper CRM

### contact-lookup
- Slack command to look up a contact from Salesforce
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### contact-lookup-process
- Processes contact lookup
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### CreateCheckoutSession
- Used in donate.md and previously membership pages
- Enabled: Yes
- Invocations: 101
- Evaluation: Processes donations; errors likely misuse

### CreateForceMajeureMembership
- Previously used for Force Majeure memberships
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### CreateLeaderMembership
- Previously used for complimentary leader memberships
- Enabled: Yes
- Invocations: 11
- Evaluation: Can be removed; successes not valid

### disable-owasp-emails
- Ran nightly to disable unused emails; now does nothing
- Enabled: Yes
- Invocations: 30
- Evaluation: Can be removed

### DisableEmail15DayNotice
- Sent notices 15 days before email disable
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### DisableEmail1DayNotice
- Sent notices 1 day before email disable
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### DisableEmail7DayNotice
- Sent notices 7 days before email disable
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### DisableEmailVerify
- Added users to notification queues
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### DisableOWASPEmail
- Disables OWASP email
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### event-create
- Slack command to create regional event info
- Enabled: Yes
- Invocations: 0
- Evaluation: Keep while regional events remain in GitHub

### event-process
- Processes regional event creation
- Enabled: Yes
- Invocations: 0
- Evaluation: Keep; update to use Monday.com

### EventBotQueueWorker
- Processes Slack event creation queue
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### EventsCheckout
- Handled ordering tickets for Slack-created events
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### EventsSlackbot
- Slack commands for creating events/sites
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### get-member-info
- Used in www-members site
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### get-repo-file
- Pulls a file from GitHub
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### GetMeetupEvents
- Displays chapter Meetup events
- Enabled: Yes
- Invocations: 47,504
- Errors: 815
- Evaluation: Can be removed when moved to owasp.community

### HandleAddMembers
- Queues add-members requests
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### IsLeaderByEmail
- Looks up leader email
- Enabled: Yes
- Invocations: 1
- Evaluation: Can be removed

### IsMember
- Looks up membership in Copper CRM
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### leader-report
- Requests a leader report
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### member-report
- Requests a member report
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### owasp-slack-add-user
- Adds a user to the OWASP Slack instance
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### process-handle-add-members
- Processes the HandleAddMembers command
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### project-create
- Slack command to create a www-project- page
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### project-process
- Processes the project-create command
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### project-report
- Slack command to launch a project report
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### project_create_jira
- Slack command to create a project page from JIRA
- Enabled: Yes
- Invocations: 0
- Evaluation: Useful until projects move

### project_create_jira_process
- Processes the JIRA-based project creation
- Enabled: Yes
- Invocations: 0
- Evaluation: Useful; needs update

### provision-zoom-email
- Adds leaders to a shared Zoom account
- Enabled: Yes
- Invocations: 1
- Evaluation: In use; Zoom accounts need updates

### provision-zoom-process
- Processes Zoom provisioning
- Enabled: Yes
- Invocations: 1
- Evaluation: Same as above

### ProvisionEmail
- Self-provisioning of OWASP email
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### rebuild-site
- Rebuilds all GitHub pages sites
- Enabled: Yes
- Invocations: 0
- Evaluation: Use sparingly

### rebuild_milestones
- Rebuilds staff milestones
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### report-process
- Processes report commands
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### RunCurrentTests
- Used to run and test functionality
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### SlackActionTrigger
- Pre-processes Slack commands
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### stripe-customer-cleanup
- Cleans unused Stripe customers
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### StripeQueueWorker
- Processes Stripe payments, invoices, donations
- Enabled: Yes
- Invocations: 22
- Evaluation: Keep while Stripe donations remain

### StripeWebhookProcessor
- Adds queue messages from Stripe webhooks
- Enabled: Yes
- Invocations: 23
- Evaluation: Keep while Stripe donations remain

### StudentMemberQueueWatcher
- Created student memberships in Salesforce
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### StudentMemberWebhook
- Added queue messages for student memberships
- Enabled: No
- Invocations: 0
- Evaluation: Can be removed

### update-member-info
- Allows self-updates to member information
- Enabled: Yes
- Invocations: 0
- Evaluation: Can be removed

### validate-otp-user
- Cloudflare validator for old members site
- Enabled: Yes
- Invocations: 4
- Evaluation: Can be removed
