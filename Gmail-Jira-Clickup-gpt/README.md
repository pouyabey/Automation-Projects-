# Ridgeline IT Automation
## Gmail + Jira + OpenAI API + Cloudflare Workers + ClickUp

Project Status:
Working Prototype / Pre-Security-Hardening

##### ============================================================

##### SECURITY NOTICE

##### ============================================================

This repository intentionally uses generic placeholders for internal
infrastructure information.

The following values have been removed, redacted, or replaced with
generic examples for security and privacy reasons:

- Production Worker hostname
- School email addresses
- Google Cloud Project ID
- Google Cloud Project Number
- Google OAuth Client information
- Google Pub/Sub resource identifiers
- ClickUp Workspace ID
- ClickUp Space ID
- ClickUp List IDs
- ClickUp Custom Field IDs
- ClickUp Custom Field Option IDs
- Internal account identifiers
- OAuth tokens
- API tokens
- Webhook secrets
- Other environment-specific identifiers

Examples in this document may therefore use placeholders such as:

<WORKER_DOMAIN>
<GOOGLE_PROJECT_ID>
<PUBSUB_TOPIC>
<CLICKUP_LIST_ID>
<CUSTOM_FIELD_ID>
<SCHOOL_IT_EMAIL>

These placeholders do NOT represent the production values.

Production credentials and identifiers are maintained separately through
the appropriate administrative platforms and environment configuration.

No production secret should ever be committed to this repository.


============================================================
1. PROJECT PURPOSE
============================================================

Ridgeline IT Automation is an automation system designed to reduce the
manual administrative work involved in school IT support.

The system connects:

- Gmail
- Jira Service Management
- OpenAI API
- Cloudflare Workers
- Cloudflare Workers KV
- Google Cloud Pub/Sub
- ClickUp

Instead of manually reviewing every email or Jira ticket and manually
re-entering that information into ClickUp, the system can automatically:

1. Detect a new IT request.
2. Retrieve the relevant request information.
3. Analyze the request using OpenAI.
4. Determine whether the request is actionable.
5. Classify the type of IT work.
6. Determine priority.
7. Select the appropriate ClickUp list.
8. Generate a concise task name.
9. Generate a useful task description.
10. Identify the next action.
11. Generate technical recommendations.
12. Generate subtasks when appropriate.
13. Create the parent task in ClickUp.
14. Populate relevant ClickUp custom fields.
15. Create ClickUp subtasks.
16. Link the ClickUp task back to the original request when appropriate.
17. Maintain processing state to prevent duplicate Gmail tasks.

The long-term goal is to create a centralized and largely automated IT
operations workflow while reducing repetitive administrative work.


============================================================
2. CURRENT PROJECT STATUS
============================================================

Currently Working:

[X] Gmail → AI → ClickUp

[X] Jira → AI → ClickUp

[X] Manual AI Task Creator → ClickUp

[X] Gmail OAuth

[X] Gmail API

[X] Gmail Watch

[X] Google Cloud Pub/Sub

[X] Gmail History API

[X] Cloudflare Workers KV

[X] Gmail message deduplication

[X] AI actionable/non-actionable classification

[X] Structured AI task generation

[X] ClickUp task creation

[X] ClickUp custom fields

[X] ClickUp subtask creation

[X] Related Gmail message links

[X] Jira webhook authentication


Not Yet Complete:

[ ] Gmail Pub/Sub webhook authentication hardening

[ ] Additional sensitive-data minimization

[ ] Final school privacy/data-policy review

[ ] Automatic Gmail Watch renewal

[ ] Production logging review

[ ] Credential rotation procedures

[ ] Final production security review


============================================================
3. HIGH-LEVEL ARCHITECTURE
============================================================

GMAIL WORKFLOW:

Gmail Inbox
    |
    v
Gmail API Watch
    |
    v
Google Cloud Pub/Sub Topic
    |
    v
Pub/Sub Push Subscription
    |
    v
Cloudflare Worker
    |
    +-------------------------+
    |                         |
    v                         v
Gmail History API       Cloudflare Workers KV
    |                         |
    |                         +--> History checkpoint
    |                         +--> Processed Message IDs
    |                         +--> Processing state
    |                         +--> Pub/Sub state
    |
    v
New Gmail Message
    |
    v
OpenAI Responses API
    |
    +---- Not Actionable
    |          |
    |          v
    |     Mark Processed
    |
    +---- Actionable
               |
               v
        Structured IT Task
               |
               v
           ClickUp API
               |
               +--> Parent Task
               +--> Custom Fields
               +--> Subtasks


JIRA WORKFLOW:

Jira Service Management
    |
    v
Jira Automation
    |
    v
Authenticated Webhook
    |
    v
Cloudflare Worker
    |
    v
OpenAI Responses API
    |
    v
Structured IT Task
    |
    v
ClickUp API


MANUAL WORKFLOW:

Ridgeline IT Automation UI
    |
    v
Paste IT Request
    |
    v
OpenAI Responses API
    |
    v
Editable Task Preview
    |
    v
Create in ClickUp


============================================================
4. TECHNOLOGIES USED
============================================================

Cloudflare:

- Cloudflare Workers
- Cloudflare Workers KV
- Worker Secrets


Google:

- Google Cloud
- Google Auth Platform
- OAuth 2.0
- Gmail API
- Gmail History API
- Gmail Watch
- Google Cloud Pub/Sub


AI:

- OpenAI API
- OpenAI Responses API
- Structured Outputs / JSON Schema


IT Service Management:

- Jira Service Management
- Jira Automation


Task Management:

- ClickUp API
- ClickUp Lists
- ClickUp Custom Fields
- ClickUp Subtasks


============================================================
5. CLOUDFLARE WORKER
============================================================

The central integration service is implemented as a Cloudflare Worker.

Production hostname:

<WORKER_DOMAIN>


Example:

https://<WORKER_DOMAIN>


The production hostname has intentionally been removed from this public
documentation.


Important Worker routes:

/

Main Ridgeline IT Automation web interface.


/status

Reports whether required configuration is available.


/test-openai

Tests OpenAI API connectivity.


/google-auth

Starts Google OAuth authorization.


/google-oauth-callback

Receives the Google OAuth callback.


/test-gmail

Tests Gmail API connectivity.


/gmail-watch

Activates or renews Gmail push notifications.


/gmail-state

Displays Gmail History processing state.


/gmail-webhook

Receives Google Pub/Sub Gmail notifications.


/analyze

Performs manual AI task analysis.


/create

Creates a ClickUp task from the manual interface.


/jira-webhook

Receives Jira Automation events.


============================================================
6. CLOUDFLARE WORKER SECRETS
============================================================

Production secrets are configured through Cloudflare Worker Secrets.

The application expects environment variables similar to:

OPENAI_API_KEY

CLICKUP_API_TOKEN

GOOGLE_CLIENT_ID

GOOGLE_CLIENT_SECRET

GOOGLE_OAUTH_STATE_SECRET

GMAIL_REFRESH_TOKEN

JIRA_WEBHOOK_SECRET


IMPORTANT:

The actual values are NOT included in this repository.


The application accesses them through the Worker environment.

Conceptual example:

env.OPENAI_API_KEY

env.CLICKUP_API_TOKEN


Secrets must never be hardcoded into source code.


============================================================
7. CLOUDFLARE WORKERS KV
============================================================

A Cloudflare Workers KV namespace is bound to the Worker.

Public documentation name:

<GMAIL_STATE_KV_NAMESPACE>


Worker binding:

GMAIL_STATE


The actual production namespace identifier has been intentionally omitted.


The purpose of KV is to maintain Gmail processing state and significantly
reduce duplicate processing.


Important key patterns:

gmail:last_history_id

gmail:watch_expiration

gmail:processed:<MESSAGE_ID>

gmail:processing:<MESSAGE_ID>

gmail:pubsub:<PUBSUB_MESSAGE_ID>


============================================================
8. GMAIL STATE KEYS
============================================================

gmail:last_history_id

Stores the Gmail History ID representing the mailbox history that has
already been processed.


------------------------------------------------------------

gmail:watch_expiration

Stores the expiration timestamp returned by Gmail Watch.


------------------------------------------------------------

gmail:processed:<MESSAGE_ID>

Indicates that a Gmail message has already been handled.


------------------------------------------------------------

gmail:processing:<MESSAGE_ID>

Represents temporary processing state while a message is being handled.


------------------------------------------------------------

gmail:pubsub:<PUBSUB_MESSAGE_ID>

Tracks Pub/Sub notification processing to reduce repeated handling of the
same Pub/Sub event.


============================================================
9. PROCESSED MESSAGE RETENTION
============================================================

Processed Gmail Message IDs are currently retained in KV for approximately:

90 days


The full Gmail email body is not intentionally stored in KV.


KV primarily contains:

- Message IDs
- History IDs
- Processing state
- Pub/Sub state
- Watch expiration metadata


============================================================
10. ORIGINAL GMAIL DUPLICATION PROBLEM
============================================================

The first Gmail automation implementation used the following approach:

Pub/Sub Notification
    |
    v
Cloudflare Worker
    |
    v
Search Gmail for recent Inbox messages
    |
    v
Process messages inside a recent time window


This implementation created an important reliability problem.

A Gmail Pub/Sub notification represents a mailbox change.

It does NOT necessarily represent a unique email that should be processed
exactly once.


Multiple Gmail notifications could therefore occur while the same email
remained inside the recent-message search window.


Every notification caused the Worker to search recent Inbox messages again.


The same email could therefore be:

- Retrieved repeatedly
- Sent to OpenAI repeatedly
- Classified repeatedly
- Converted into multiple ClickUp tasks


During testing, one email generated several similar ClickUp tasks.


The Pub/Sub subscription was temporarily removed while the Gmail processing
architecture was redesigned.


============================================================
11. DUPLICATE PREVENTION REDESIGN
============================================================

The Gmail implementation was redesigned around:

Gmail History API
+
Cloudflare Workers KV


New architecture:

Gmail receives new message
    |
    v
Gmail creates mailbox history event
    |
    v
Pub/Sub sends notification containing historyId
    |
    v
Worker receives notification
    |
    v
Worker reads previous historyId from KV
    |
    v
Gmail History API
    |
    v
Find messageAdded events
    |
    v
Extract unique Gmail Message IDs
    |
    v
Check KV
    |
    +---- Already processed --> Skip
    |
    +---- New message
              |
              v
        Retrieve message
              |
              v
          Analyze email
              |
              v
       Create ClickUp Task
              |
              v
        Mark as processed


This architecture successfully resolved the observed duplicate-task problem
during testing.


============================================================
12. GMAIL HISTORY API
============================================================

The Gmail History API is now the primary mechanism used to determine which
messages are actually new.


The Worker receives:

historyId


from Gmail through Pub/Sub.


The Worker then retrieves:

gmail:last_history_id


from KV.


It requests Gmail history using conceptually:

users.history.list


with:

startHistoryId=<LAST_PROCESSED_HISTORY_ID>

historyTypes=messageAdded

labelId=INBOX


This means the Worker asks Gmail:

"What messages were added to the Inbox after the history point I already
processed?"


============================================================
13. GMAIL HISTORY PAGINATION
============================================================

Gmail History responses can contain multiple pages.


The Worker supports:

nextPageToken


and continues retrieving history until all available pages have been
processed.


Message IDs are collected into a Set.


This provides an additional in-memory deduplication layer before messages
are processed.


============================================================
14. PER-MESSAGE PROCESSING
============================================================

For every Message ID returned by Gmail History:

1. Check:

gmail:processed:<MESSAGE_ID>


2. If the key exists:

Skip the message.


3. Otherwise create temporary processing state:

gmail:processing:<MESSAGE_ID>


4. Retrieve the Gmail message.


5. Verify that the message currently has the INBOX label.


6. Parse the message.


7. Check whether the message originated from the connected mailbox.


8. Send the relevant message content to OpenAI.


9. Determine whether the email represents actionable IT work.


10. If non-actionable:

Mark the message as processed.


11. If actionable:

Create the ClickUp task.


12. Only after successful ClickUp creation:

Mark the Gmail message as processed.


13. Remove temporary processing state.


============================================================
15. FAILURE HANDLING
============================================================

An important reliability rule is:

An actionable Gmail message is NOT marked successfully processed until
ClickUp confirms that its task was created.


Correct sequence:

Receive Gmail Message
    |
    v
AI Analysis
    |
    v
Create ClickUp Task
    |
    v
ClickUp Success
    |
    v
Mark Gmail Message Processed


If OpenAI or ClickUp fails:

The message is not permanently marked as successfully processed.


This allows the workflow to retry later.


============================================================
16. HISTORY CHECKPOINT ADVANCEMENT
============================================================

The Worker advances:

gmail:last_history_id


only after processing the relevant Gmail History batch successfully.


Example:

History Batch:

Message A
Message B
Message C


Processing:

A = Success
B = Success
C = Failure


The overall history checkpoint is not advanced.


During retry:

Message A
    --> KV says processed
    --> Skip


Message B
    --> KV says processed
    --> Skip


Message C
    --> Not processed
    --> Retry


This reduces the possibility of permanently losing an IT request because
one operation failed.


============================================================
17. INVALID HISTORY ID RECOVERY
============================================================

Gmail History IDs are not permanent.


If the stored History ID becomes too old, Gmail may reject it.


The Worker detects an invalid/expired history state.


Instead of scanning an arbitrary collection of recent emails and risking
large-scale duplicate creation, the Worker can reset the processing
baseline to the current Gmail History ID.


This design prioritizes safe recovery and duplicate prevention.


============================================================
18. GOOGLE CLOUD PROJECT
============================================================

The system uses a dedicated Google Cloud project.


Public identifier:

<GOOGLE_PROJECT_ID>


Project number:

<GOOGLE_PROJECT_NUMBER>


The real production identifiers have intentionally been removed from this
repository.


The project contains/configures:

- Gmail API
- Google OAuth
- Google Cloud Pub/Sub
- Pub/Sub Topic
- Pub/Sub Subscription
- Gmail publishing permissions


============================================================
19. GOOGLE OAUTH
============================================================

A Google OAuth Web Application is configured for the Worker.


OAuth Client:

<GOOGLE_OAUTH_CLIENT>


Application Type:

Web Application


Authorized Redirect URI:

https://<WORKER_DOMAIN>/google-oauth-callback


The production OAuth client information and Worker hostname have
intentionally been omitted.


============================================================
20. GMAIL OAUTH SCOPE
============================================================

The Gmail integration currently uses:

https://www.googleapis.com/auth/gmail.readonly


The readonly scope was intentionally selected.


The automation needs the ability to:

- Read Gmail messages
- Read Gmail metadata
- Read Gmail History
- Support Gmail Watch processing


The automation does NOT require permission to:

- Send email
- Delete email
- Modify email


Least-privilege access is preferred wherever practical.


============================================================
21. GOOGLE OAUTH FLOW
============================================================

Authorization begins at:

/google-auth


The Worker redirects the administrator to Google OAuth.


After authorization, Google redirects to:

/google-oauth-callback


The Worker exchanges the temporary authorization code for OAuth tokens.


The resulting refresh token is stored as the Cloudflare secret:

GMAIL_REFRESH_TOKEN


The refresh token is NOT stored in this repository.


When Gmail API access is required, the Worker exchanges the refresh token
for a temporary Google access token.


============================================================
22. OAUTH STATE PROTECTION
============================================================

The OAuth implementation includes a signed state value.


Secret:

GOOGLE_OAUTH_STATE_SECRET


The Worker signs OAuth state using:

HMAC SHA-256


The state contains:

- Timestamp
- Cryptographic signature


When Google redirects back to the Worker, the Worker verifies:

- State exists
- Signature is valid
- State has not expired


The state lifetime is approximately:

15 minutes


This provides protection against forged OAuth callback requests and
CSRF-style OAuth attacks.


============================================================
23. GMAIL CONNECTION TEST
============================================================

The Gmail connection can be tested through:

/test-gmail


The endpoint calls Gmail:

users.getProfile


A successful response confirms that:

- Google OAuth credentials are functioning
- The refresh token is valid
- Gmail API is enabled
- The Worker can obtain an access token
- The intended mailbox is accessible


The production mailbox address has intentionally been removed.


Public placeholder:

<SCHOOL_IT_EMAIL>


============================================================
24. GOOGLE CLOUD PUB/SUB
============================================================

Google Cloud Pub/Sub provides event delivery between Gmail and the
Cloudflare Worker.


Topic:

<PUBSUB_TOPIC>


Full production resource:

projects/<GOOGLE_PROJECT_ID>/topics/<PUBSUB_TOPIC>


The actual production topic/project identifiers have intentionally been
replaced with placeholders.


============================================================
25. GMAIL PUB/SUB PUBLISHER
============================================================

Gmail requires permission to publish notifications into the configured
Pub/Sub Topic.


The Gmail push notification publishing service is granted:

Pub/Sub Publisher


on the appropriate Topic.


The Google-managed Gmail publishing identity is configured according to
Google's Gmail Push Notification requirements.


============================================================
26. PUB/SUB SUBSCRIPTION
============================================================

A Push Subscription connects the Pub/Sub Topic to the Cloudflare Worker.


Public placeholder:

<PUBSUB_SUBSCRIPTION>


Delivery Type:

Push


Push Endpoint:

https://<WORKER_DOMAIN>/gmail-webhook


Payload Unwrapping:

Disabled


The Worker therefore receives the normal Google Pub/Sub message envelope.


============================================================
27. PUB/SUB MESSAGE FORMAT
============================================================

Conceptually, the Worker receives:

{
    "message": {
        "data": "<BASE64_GMAIL_NOTIFICATION>",
        "messageId": "<PUBSUB_MESSAGE_ID>"
    },
    "subscription": "<SUBSCRIPTION>"
}


The Worker decodes:

message.data


The decoded Gmail notification contains information such as:

emailAddress

historyId


============================================================
28. GMAIL WATCH
============================================================

Gmail Watch is activated through:

/gmail-watch


The Worker calls:

users.watch


using the configured Pub/Sub Topic.


Current label filtering:

INBOX


Behavior:

INCLUDE


This means the automation is interested in Inbox-related changes rather
than processing every possible mailbox label.


============================================================
29. GMAIL WATCH STATE
============================================================

When Gmail Watch is successfully activated, Gmail returns:

historyId

expiration


The Worker stores these values in KV.


Conceptually:

gmail:last_history_id
    = <HISTORY_ID>


gmail:watch_expiration
    = <EXPIRATION_TIMESTAMP>


Production History IDs and timestamps are intentionally not included in
this public documentation.


============================================================
30. GMAIL WATCH BASELINE
============================================================

When Gmail Watch is activated, the returned History ID establishes the
automation baseline.


This prevents existing Inbox messages from automatically being interpreted
as newly-arriving IT requests.


Conceptually:

Existing Mail
    |
    |  Gmail Watch activated here
    v
---------------- BASELINE ----------------
    |
    v
New Mail
    |
    v
Automation


Only mailbox changes after the established baseline should be processed.


============================================================
31. GMAIL WATCH EXPIRATION
============================================================

Gmail Watch registrations expire periodically.


The current implementation can manually activate or renew Gmail Watch
through:

/gmail-watch


Automatic Watch renewal has not yet been implemented.


Planned future design:

Cloudflare Scheduled Trigger
    |
    v
Check/Renew Gmail Watch
    |
    v
Gmail users.watch


This is part of the planned reliability/security hardening phase.


============================================================
32. GMAIL MESSAGE RETRIEVAL
============================================================

New messages identified through Gmail History are retrieved using:

users.messages.get


Format:

full


The Worker extracts useful headers including:

Subject

From

To

Cc

Date

Message-ID


============================================================
33. EMAIL BODY EXTRACTION
============================================================

The Worker attempts to retrieve:

text/plain


before falling back to:

text/html


If HTML is used, the Worker converts the content into plain text before
sending it for AI analysis.


This reduces unnecessary markup and makes the request easier for the model
to interpret.


============================================================
34. EMAIL SIZE LIMITING
============================================================

Large email bodies are truncated before AI analysis.


The current implementation limits the email body sent for analysis to
approximately:

15,000 characters


This reduces:

- Unnecessary token usage
- Processing cost
- Extremely large email chains
- Excessive unrelated context


Further data minimization is planned.


============================================================
35. SELF-SENT EMAIL HANDLING
============================================================

The Worker can compare the email's From header with the connected mailbox.


Messages originating from the connected IT mailbox can be ignored and
marked as processed.


This helps prevent the automation from turning the IT technician's own
outgoing email into a new task.


============================================================
36. OPENAI API
============================================================

The automation uses the OpenAI API.


API style:

Responses API


Endpoint:

https://api.openai.com/v1/responses


The specific production model may be configured in the Worker.


Public placeholder:

<OPENAI_MODEL>


The OpenAI API key is stored in:

OPENAI_API_KEY


The actual API key is never included in source code or documentation.


============================================================
37. OPENAI STORAGE SETTING
============================================================

Responses API requests currently specify:

store: false


This configuration should remain enabled unless there is a deliberate
reason to change the data handling design.


============================================================
38. STRUCTURED AI OUTPUT
============================================================

The system does not rely on free-form AI prose for automation.


OpenAI Structured Outputs / JSON Schema are used.


For an actionable IT request, the expected task structure includes:

{
    "name": "...",
    "list": "...",
    "category": "...",
    "priority": "...",
    "description": "...",
    "location": "...",
    "nextAction": "...",
    "waitingOn": "...",
    "recommendation": "...",
    "subtasks": []
}


This makes the AI output predictable enough to use programmatically.


============================================================
39. EMAIL ACTIONABILITY CLASSIFICATION
============================================================

Every new Gmail message is analyzed individually.


The first decision is:

isTask = true

or

isTask = false


Conceptual output:

{
    "isTask": true,
    "reason": "...",
    "task": {
        ...
    }
}


or:

{
    "isTask": false,
    "reason": "...",
    "task": null
}


============================================================
40. ACTIONABLE EMAIL EXAMPLES
============================================================

Examples include:

- Computer problem
- Chromebook problem
- Mac problem
- Printer problem
- Projector problem
- Wi-Fi problem
- Network problem
- Login/account problem
- Software problem
- Device deployment request
- Configuration request
- Installation request
- IT purchasing request
- Vendor communication requiring IT action
- Documentation request
- Follow-up requiring additional IT action


============================================================
41. NON-ACTIONABLE EMAIL EXAMPLES
============================================================

Examples include:

- Newsletters
- Marketing
- Spam
- General announcements
- Informational automated notifications
- Thank-you messages where no work remains
- Replies requiring no additional IT action
- Calendar notifications with no IT action


If an email is classified as non-actionable:

No ClickUp task is created.


The Gmail message is still marked as processed so the system does not
repeatedly send it to OpenAI.


============================================================
42. AI ACCURACY RULES
============================================================

The system prompt instructs the AI to:

- Never invent facts
- Never invent locations
- Never invent deadlines
- Never invent users
- Never invent devices
- Never invent causes
- Never invent solutions as established facts
- Use an empty location when unknown
- Use an empty Waiting On value when appropriate
- Avoid unnecessary subtasks
- Ignore irrelevant signatures and footer content
- Avoid creating busywork
- Separate factual request information from technical recommendations


============================================================
43. CLICKUP INTEGRATION
============================================================

ClickUp acts as the central operational task-management destination.


Production identifiers have intentionally been removed.


Workspace:

<CLICKUP_WORKSPACE>


Workspace ID:

<CLICKUP_WORKSPACE_ID>


Space:

<CLICKUP_SPACE>


Space ID:

<CLICKUP_SPACE_ID>


============================================================
44. CLICKUP LISTS
============================================================

The automation currently routes work into the following logical lists:


Support Tickets

Production ID:

<CLICKUP_SUPPORT_LIST_ID>


------------------------------------------------------------


IT Projects

Production ID:

<CLICKUP_PROJECTS_LIST_ID>


------------------------------------------------------------


Maintenance

Production ID:

<CLICKUP_MAINTENANCE_LIST_ID>


------------------------------------------------------------


Devices & Deployments

Production ID:

<CLICKUP_DEVICES_LIST_ID>


------------------------------------------------------------


Waiting / Follow-up

Production ID:

<CLICKUP_WAITING_LIST_ID>


------------------------------------------------------------


Documentation

Production ID:

<CLICKUP_DOCUMENTATION_LIST_ID>


All production List IDs have intentionally been removed.


============================================================
45. CLICKUP LIST ROUTING
============================================================

Support Tickets:

Normal end-user IT support issues.


IT Projects:

Substantial multi-step IT projects.


Maintenance:

Recurring or preventive IT work.


Devices & Deployments:

Device preparation, deployment, replacement, setup, or enrollment.


Waiting / Follow-up:

Tasks primarily blocked by another person, vendor, purchase, approval, or
external dependency.


Documentation:

Tasks where documentation or a guide is the primary deliverable.


============================================================
46. CLICKUP CATEGORIES
============================================================

Allowed categories:

Support

Project

Maintenance

Device / Deployment

Documentation


Waiting is intentionally not represented as a Category.


Waiting state is represented separately using:

Waiting On


============================================================
47. CLICKUP PRIORITIES
============================================================

AI priorities map to ClickUp priorities.


Urgent
    -->
ClickUp Priority 1


High
    -->
ClickUp Priority 2


Normal
    -->
ClickUp Priority 3


Low
    -->
ClickUp Priority 4


============================================================
48. PRIORITY LOGIC
============================================================

Urgent:

- Major outage
- Security incident
- Safety-related technology incident
- Widespread service failure
- Work-stopping issue requiring immediate response


High:

- Teaching significantly affected
- Testing affected
- Multiple users affected
- Significant staff productivity impact
- Clearly time-sensitive request


Normal:

- Typical IT request
- Limited user impact
- No major urgency


Low:

- Minor improvement
- Cosmetic issue
- Optional request
- Non-time-sensitive work


============================================================
49. CLICKUP CUSTOM FIELDS
============================================================

Production ClickUp Custom Field IDs and Option IDs have intentionally been
removed from this repository.


Logical fields currently used include:


Waiting On

Field ID:

<CUSTOM_FIELD_WAITING_ON>


Allowed values:

User

Vendor

Purchase

Management

Facilities

Other


Each option has a production ClickUp Option ID that is maintained outside
this public documentation.


------------------------------------------------------------


Category

Field ID:

<CUSTOM_FIELD_CATEGORY>


Allowed values:

Support

Project

Maintenance

Device / Deployment

Documentation


------------------------------------------------------------


Related Email

Field ID:

<CUSTOM_FIELD_RELATED_EMAIL>

Type:

URL


------------------------------------------------------------


Location

Field ID:

<CUSTOM_FIELD_LOCATION>

Type:

Short Text


------------------------------------------------------------


Ticket URL

Field ID:

<CUSTOM_FIELD_TICKET_URL>

Type:

URL


------------------------------------------------------------


Next Action

Field ID:

<CUSTOM_FIELD_NEXT_ACTION>

Type:

Short Text


============================================================
50. CLICKUP DESCRIPTION DESIGN
============================================================

The AI-generated Description contains the factual request summary.


The AI-generated Recommendation is appended separately.


Conceptually:

<REQUEST DESCRIPTION>


Recommendation:

<AI TECHNICAL RECOMMENDATION>


This separation is intentional.


It helps distinguish:

What the user actually reported

from:

What the AI recommends doing about it


============================================================
51. CLICKUP SUBTASKS
============================================================

OpenAI may generate subtasks when they provide meaningful operational
value.


The Worker first creates the parent ClickUp task.


It then creates each subtask as a child of that parent.


Conceptual example:

Parent:

Troubleshoot classroom printer offline


Subtasks:

- Verify printer power/network connection
- Verify printer IP address
- Test connectivity
- Check workstation print queue
- Perform test print


Subtasks are not required for every request.


============================================================
52. RELATED GMAIL MESSAGE
============================================================

For Gmail-generated tasks, the Worker creates a Gmail link associated with
the original Message ID.


Conceptual format:

https://mail.google.com/mail/u/0/#all/<MESSAGE_ID>


The URL is placed into the ClickUp:

Related Email

custom field.


This allows the IT technician to move from the ClickUp task back to the
original Gmail conversation.


============================================================
53. JIRA SERVICE MANAGEMENT
============================================================

Jira Service Management is another input source for the automation.


Production project information has been generalized.


Project:

<JIRA_PROJECT>


Project Key:

<JIRA_PROJECT_KEY>


Jira Automation Trigger:

Work item created


Action:

Send web request


Destination:

https://<WORKER_DOMAIN>/jira-webhook


Method:

POST


============================================================
54. JIRA WEBHOOK PAYLOAD
============================================================

The Jira Automation sends structured information similar to:

{
    "key": "{{issue.key}}",
    "summary": {{issue.summary.asJsonString}},
    "description": {{issue.description.asJsonString}},
    "reporter": {{issue.reporter.displayName.asJsonString}},
    "issueType": {{issue.issueType.name.asJsonString}},
    "priority": {{issue.priority.name.asJsonString}},
    "requestType": {{issue.Request Type.requestType.name.asJsonString}},
    "url": "{{baseUrl}}/browse/{{issue.key}}"
}


============================================================
55. JIRA WEBHOOK AUTHENTICATION
============================================================

The Jira webhook includes a custom header:

X-Ridgeline-Webhook-Secret


The corresponding Worker secret is:

JIRA_WEBHOOK_SECRET


The production value is not included in this repository.


The Worker verifies the provided value before processing the Jira event.


Invalid requests receive:

HTTP 401 Unauthorized


============================================================
56. JIRA AI PROCESSING
============================================================

Useful Jira information is converted into AI input.


This may include:

- Ticket Key
- Summary
- Description
- Request Type
- Issue Type
- Jira Priority
- Reporter


OpenAI then generates the same standardized task structure used elsewhere
in the system.


The original Jira ticket URL is written into the ClickUp:

Ticket URL

custom field.


============================================================
57. MANUAL AI TASK CREATOR
============================================================

The Cloudflare Worker also hosts a manual interface.


This allows IT staff to create AI-assisted tasks that did not originate
from Gmail or Jira.


Examples:

- Verbal request from staff
- Problem discovered during troubleshooting
- Infrastructure improvement idea
- Maintenance work
- Device deployment
- Documentation task
- Administrative IT project


============================================================
58. MANUAL AI WORKFLOW
============================================================

IT Request
    |
    v
Paste into Web Interface
    |
    v
POST /analyze
    |
    v
OpenAI
    |
    v
Structured Task
    |
    v
Editable Preview
    |
    v
Create in ClickUp
    |
    v
POST /create
    |
    v
ClickUp API


============================================================
59. MANUAL TASK PREVIEW
============================================================

Before creating a task, the interface allows editing fields including:

- Task Name
- List
- Category
- Priority
- Waiting On
- Location
- Next Action
- Description
- AI Recommendation
- Subtasks


This provides human review before manual AI-generated tasks are submitted.


============================================================
60. SYSTEM STATUS
============================================================

The Worker includes:

/status


This endpoint checks whether required configuration is available.


A healthy system conceptually reports:

{
    "success": true,
    "services": {
        "openai": true,
        "clickup": true,
        "jira": true,
        "googleClient": true,
        "googleOAuthSecurity": true,
        "gmailAuthorized": true,
        "gmailStateKV": true
    }
}


This endpoint reports whether configuration is present.


It does NOT return secret values.


============================================================
61. OPENAI CONNECTION TEST
============================================================

Endpoint:

/test-openai


Purpose:

Verify that:

- OPENAI_API_KEY is configured
- OpenAI API is reachable
- The configured model is available
- The Worker can receive a valid response


============================================================
62. GMAIL CONNECTION TEST
============================================================

Endpoint:

/test-gmail


Purpose:

Verify that:

- Google OAuth credentials work
- Refresh token works
- Access token generation works
- Gmail API is reachable
- Connected mailbox is accessible


Mailbox information should not be included in public documentation.


============================================================
63. SUCCESSFUL END-TO-END GMAIL TEST
============================================================

After implementing Gmail History API and KV deduplication, the complete
workflow was tested.


Test sequence:

1. Gmail Watch was activated.

2. Gmail History baseline was stored.

3. Pub/Sub Push Subscription was enabled.

4. A completely new test email was sent.

5. Gmail generated a mailbox change.

6. Gmail published the event to Pub/Sub.

7. Pub/Sub pushed the event to the Cloudflare Worker.

8. The Worker retrieved the previous History ID.

9. The Worker queried Gmail History.

10. The Worker identified the new Message ID.

11. The Worker verified that the Message ID had not been processed.

12. The Worker retrieved the email.

13. OpenAI analyzed the email.

14. OpenAI classified the email as actionable.

15. OpenAI generated structured task information.

16. The Worker created the ClickUp parent task.

17. The Worker created relevant custom fields/subtasks.

18. The Worker marked the Gmail Message ID as processed.

19. The Worker advanced the Gmail History checkpoint.

20. Exactly ONE ClickUp task was created.


This confirmed that the previously observed duplicate-task problem had
been resolved during testing.


============================================================
64. CURRENT DATA FLOW
============================================================

For an actionable Gmail message:

Gmail
    |
    v
Cloudflare Worker
    |
    v
Relevant Email Content
    |
    v
OpenAI API
    |
    v
Structured Task Data
    |
    v
Cloudflare Worker
    |
    v
ClickUp


The complete Gmail body is not intentionally stored in Workers KV.


============================================================
65. SECURITY ARCHITECTURE ALREADY PRESENT
============================================================

The project already includes several security controls.


1. Secrets are stored outside source code using Cloudflare Worker Secrets.


2. Gmail uses a readonly OAuth scope.


3. Google OAuth state is cryptographically signed.


4. OAuth state has a short expiration period.


5. Jira webhook requests require a secret.


6. OpenAI requests currently use:

store: false


7. Gmail email bodies are not intentionally persisted in Workers KV.


8. Public documentation does not contain production secrets.


9. Infrastructure identifiers in this public README are intentionally
generalized.


============================================================
66. SECURITY HARDENING NOT YET COMPLETE
============================================================

IMPORTANT:

The system is functional, but final security hardening has not yet been
completed.


The remaining security work is intentionally documented rather than hidden.


Primary remaining areas:

1. Gmail Pub/Sub webhook authentication

2. AI data minimization

3. School privacy/data policy review

4. Production logging review

5. Credential rotation procedures

6. Automatic Gmail Watch renewal

7. Final production security assessment


============================================================
67. GMAIL WEBHOOK SECURITY
============================================================

Current endpoint:

POST /gmail-webhook


Conceptual production URL:

https://<WORKER_DOMAIN>/gmail-webhook


The endpoint is internet-accessible.


The current Gmail Pub/Sub Push configuration has not yet received final
authentication hardening.


This is a known security item.


Future implementation should verify that webhook requests genuinely
originate through the intended Google Pub/Sub path.


Potential approaches include:

- Google Pub/Sub authenticated push
- OIDC-based verification
- Additional request validation
- Appropriate Cloudflare-side access controls


This should be completed before considering the system fully hardened for
production.


============================================================
68. AI DATA MINIMIZATION
============================================================

The current system sends relevant email content to OpenAI so that the AI
can determine whether the message represents an IT task.


Future improvements should reduce unnecessary information before external
processing.


Potential improvements include:

- Remove quoted email chains
- Remove signatures
- Remove unnecessary email addresses
- Remove unrelated conversation history
- Redact sensitive identifiers when practical
- Avoid processing attachments unless explicitly required
- Restrict AI processing to appropriate IT-related content
- Define which school data categories are permitted to be processed


============================================================
69. SCHOOL DATA PRIVACY
============================================================

Technical security does not automatically equal organizational
authorization.


Before processing potentially sensitive school information, the production
deployment should consider applicable:

- School policies
- Student privacy requirements
- Staff privacy requirements
- Data-processing agreements
- Vendor agreements
- Administrative approval
- Applicable legal requirements


The project should use data minimization wherever practical.


============================================================
70. SECRET MANAGEMENT
============================================================

Secrets should continue to be managed through Cloudflare Worker Secrets or
an equivalent secure secret-management mechanism.


Secrets should never appear in:

- Git repositories
- README files
- Source code
- Screenshots
- Support tickets
- Application logs
- Public documentation


Secrets should be rotated when appropriate.


============================================================
71. LOGGING
============================================================

Production logging should avoid unnecessarily recording:

- Email bodies
- Student information
- Staff private information
- OAuth access tokens
- OAuth refresh tokens
- API keys
- Webhook secrets
- Sensitive request payloads


Operational logging should follow the principle:

Log enough to troubleshoot the automation,
but not enough to unnecessarily reproduce private content.


============================================================
72. CREDENTIAL ROTATION
============================================================

Future operational documentation should define procedures for rotating:

OPENAI_API_KEY

CLICKUP_API_TOKEN

GOOGLE_CLIENT_SECRET

GOOGLE_OAUTH_STATE_SECRET

GMAIL_REFRESH_TOKEN

JIRA_WEBHOOK_SECRET


The documentation should also describe what services must be restarted or
reauthorized after each credential is rotated.


============================================================
73. GMAIL WATCH AUTOMATION
============================================================

Gmail Watch expires periodically.


Currently it can be renewed manually.


Future implementation should automate renewal.


Possible design:

Cloudflare Cron Trigger
    |
    v
Scheduled Worker Handler
    |
    v
Check Watch Expiration
    |
    v
Renew Gmail Watch


This removes the operational risk of Gmail automation silently stopping
because a Watch expired.


============================================================
74. KV CONSISTENCY CONSIDERATION
============================================================

Cloudflare Workers KV provides practical state storage and deduplication for
the current low-volume IT workflow.


However, Workers KV is eventually consistent.


It is not a transactional database and should not be treated as a strict
distributed lock.


The current architecture significantly improves reliability compared with
the original recent-message scanning implementation.


If future requirements demand stronger exact-once processing guarantees,
possible alternatives include:

Cloudflare Durable Objects

or

Cloudflare D1 with a unique database constraint


============================================================
75. CURRENT RELIABILITY MODEL
============================================================

The current system uses multiple layers to reduce duplicate Gmail tasks:


Layer 1:

Gmail History API

Only mailbox changes after the previous checkpoint are requested.


Layer 2:

In-memory Set

Duplicate Message IDs inside a History response are collapsed.


Layer 3:

gmail:processed:<MESSAGE_ID>

Previously completed messages are skipped.


Layer 4:

gmail:processing:<MESSAGE_ID>

Temporary processing state reduces overlapping work.


Layer 5:

gmail:pubsub:<PUBSUB_MESSAGE_ID>

Previously handled Pub/Sub notifications can be recognized.


Together these provide practical duplicate prevention for the current
workflow.


============================================================
76. IMPORTANT GITHUB SECURITY RULES
============================================================

NEVER COMMIT:

OPENAI_API_KEY

CLICKUP_API_TOKEN

GOOGLE_CLIENT_SECRET

GOOGLE_OAUTH_STATE_SECRET

GMAIL_REFRESH_TOKEN

JIRA_WEBHOOK_SECRET


Also never commit:

- OAuth access tokens
- Refresh tokens
- Production API tokens
- Private email contents
- Student records
- Sensitive staff information
- Production logs containing private information
- Screenshots containing credentials
- Local secret files


============================================================
77. RECOMMENDED .GITIGNORE
============================================================

At minimum, local environment/secret files should be excluded.


Example:

.env
.env.*
.dev.vars
*.secret
secrets.json


Additional IDE/runtime-specific exclusions can be added as necessary.


============================================================
78. PUBLIC VS PRIVATE CONFIGURATION
============================================================

The public repository should contain:

- Application source code
- Generic architecture
- Generic documentation
- Setup instructions
- Environment variable names
- Placeholder resource identifiers


The public repository should NOT contain:

- Production credentials
- Production OAuth tokens
- Production refresh tokens
- Production webhook secrets
- Sensitive school data


Internal infrastructure identifiers should also be generalized when they
are not necessary for understanding the project.


============================================================
79. EXAMPLE PUBLIC CONFIGURATION
============================================================

A public example configuration may look like:

WORKER_DOMAIN=<WORKER_DOMAIN>

GOOGLE_PROJECT_ID=<GOOGLE_PROJECT_ID>

PUBSUB_TOPIC=<PUBSUB_TOPIC>

PUBSUB_SUBSCRIPTION=<PUBSUB_SUBSCRIPTION>

CLICKUP_WORKSPACE_ID=<CLICKUP_WORKSPACE_ID>

CLICKUP_SUPPORT_LIST_ID=<CLICKUP_SUPPORT_LIST_ID>

OPENAI_MODEL=<OPENAI_MODEL>


Secret values should only be represented by variable names:

OPENAI_API_KEY=<SECRET>

CLICKUP_API_TOKEN=<SECRET>

GOOGLE_CLIENT_SECRET=<SECRET>

GMAIL_REFRESH_TOKEN=<SECRET>

JIRA_WEBHOOK_SECRET=<SECRET>


============================================================
80. CURRENT END-TO-END ARCHITECTURE
============================================================

                         GMAIL
                           |
                           v
                    Gmail API Watch
                           |
                           v
                  Google Cloud Pub/Sub
                           |
                           v
                   Push Subscription
                           |
                           v
                 +-------------------+
                 | Cloudflare Worker |
                 +---------+---------+
                           |
                +----------+----------+
                |                     |
                v                     v
        Gmail History API      Cloudflare KV
                |
                v
         Retrieve Message
                |
                v
           OpenAI API
                |
                v
       Structured Task Data
                |
                v
            ClickUp API
                |
          +-----+------+
          |            |
          v            v
      Parent Task   Subtasks


                         JIRA
                           |
                           v
                    Jira Automation
                           |
                           v
                 Authenticated Webhook
                           |
                           v
                 Cloudflare Worker
                           |
                           v
                     OpenAI API
                           |
                           v
                       ClickUp


                       MANUAL
                           |
                           v
                  Worker Web Interface
                           |
                           v
                     OpenAI API
                           |
                           v
                    Editable Preview
                           |
                           v
                       ClickUp


============================================================
81. PROJECT PHASES
============================================================

PHASE 1 — CORE AUTOMATION

[X] Cloudflare Worker

[X] ClickUp integration

[X] OpenAI integration

[X] Manual AI Task Creator

[X] Jira automation


------------------------------------------------------------


PHASE 2 — GMAIL AUTOMATION

[X] Google Cloud project

[X] OAuth

[X] Gmail API

[X] Gmail Watch

[X] Pub/Sub Topic

[X] Pub/Sub Subscription

[X] Gmail → Worker

[X] Gmail → AI

[X] AI → ClickUp


------------------------------------------------------------


PHASE 3 — GMAIL RELIABILITY

[X] Identify duplicate-task problem

[X] Disable faulty subscription during troubleshooting

[X] Create Workers KV state storage

[X] Replace recent-message scanning

[X] Implement Gmail History API

[X] Implement Message ID deduplication

[X] Implement processing state

[X] Implement History checkpoint

[X] Re-enable Pub/Sub

[X] Perform end-to-end test

[X] Confirm one email creates one task during testing


------------------------------------------------------------


PHASE 4 — SECURITY HARDENING

[ ] Authenticate Gmail webhook

[ ] Improve data minimization

[ ] Review sensitive school information handling

[ ] Review production logs

[ ] Establish credential rotation

[ ] Final security assessment


------------------------------------------------------------


PHASE 5 — OPERATIONAL RELIABILITY

[ ] Automatic Gmail Watch renewal

[ ] Monitoring/alerting

[ ] Error reporting

[ ] Recovery documentation

[ ] Production runbook

[ ] Backup/recovery strategy for configuration


============================================================
82. FUTURE IMPROVEMENTS
============================================================

Possible future improvements include:

- Authenticated Gmail Pub/Sub Push
- Automatic Gmail Watch renewal
- More aggressive email-content sanitization
- Sensitive-data redaction
- Improved quoted-thread removal
- Centralized error monitoring
- Automation health dashboard
- Failed-task retry queue
- Dead-letter processing
- Better observability
- ClickUp task duplicate detection
- Jira duplicate detection
- Stronger transactional state storage
- Durable Objects or D1
- Automated secret rotation procedures
- Administrative audit logging
- Automated workflow health notifications


============================================================
83. DESIGN PRINCIPLES
============================================================

The project follows several core design principles:


1. AUTOMATE REPETITIVE ADMINISTRATION

IT time should be spent solving problems rather than repeatedly copying
information between systems.


2. KEEP A HUMAN-READABLE TASK SYSTEM

ClickUp remains the central place where IT work can be reviewed and
managed.


3. USE AI FOR CLASSIFICATION AND ORGANIZATION

AI assists with understanding and structuring requests rather than directly
performing unrestricted administrative actions.


4. USE STRUCTURED OUTPUT

AI responses are constrained through JSON Schema rather than relying on
free-form text.


5. MINIMIZE PERMISSIONS

For example, Gmail currently uses readonly access rather than broader Gmail
permissions.


6. DO NOT MARK WORK COMPLETE BEFORE THE DESTINATION CONFIRMS SUCCESS

A Gmail message is not marked processed until required downstream work
succeeds.


7. PREVENT DUPLICATE WORK

History checkpoints and per-message state are used to reduce duplicate
tasks.


8. DO NOT STORE SECRETS IN SOURCE CODE

Credentials belong in secure environment configuration.


9. DOCUMENT KNOWN SECURITY LIMITATIONS

Known security issues should be documented and fixed rather than hidden.


10. MINIMIZE SENSITIVE DATA

Only information necessary for the workflow should be processed or retained.


============================================================
84. PROJECT SUMMARY
============================================================

Ridgeline IT Automation is an AI-assisted IT workflow system connecting:

Gmail

Jira Service Management

Cloudflare Workers

Google Cloud Pub/Sub

Gmail History API

Cloudflare Workers KV

OpenAI API

ClickUp


The system can automatically identify incoming IT work, determine whether
it requires action, classify it, prioritize it, generate useful operational
information, and create organized ClickUp tasks.


The original Gmail implementation relied on scanning recent Inbox messages
after every Pub/Sub notification.


That approach resulted in duplicate ClickUp tasks.


The Gmail architecture was redesigned around:

Gmail History API
+
Cloudflare Workers KV
+
Per-message processing state
+
History checkpoints


After the redesign, end-to-end testing successfully produced one ClickUp
task from one new test email without the previously observed duplication.


The core automation and reliability phases are therefore functional.


The next major project phase is:

SECURITY HARDENING


This includes:

- Authenticating Gmail webhook traffic
- Reducing unnecessary AI data exposure
- Reviewing school data/privacy requirements
- Improving logging practices
- Establishing credential rotation procedures
- Automating Gmail Watch renewal
- Performing a final production security review


============================================================
85. PUBLIC REPOSITORY DISCLAIMER
============================================================

This repository documents the architecture and implementation of the
project while intentionally excluding production-sensitive information.


Names and descriptions of technologies and APIs are retained where useful
for understanding the architecture.


However, infrastructure-specific identifiers have been replaced with
generic placeholders.


Examples include:

<WORKER_DOMAIN>

<GOOGLE_PROJECT_ID>

<GOOGLE_PROJECT_NUMBER>

<PUBSUB_TOPIC>

<PUBSUB_SUBSCRIPTION>

<SCHOOL_IT_EMAIL>

<CLICKUP_WORKSPACE_ID>

<CLICKUP_SPACE_ID>

<CLICKUP_LIST_ID>

<CUSTOM_FIELD_ID>


These placeholders are intentional.


They should NOT be replaced with production values in the public
repository.


Production configuration should remain in the appropriate secured
administrative systems and environment configuration.
