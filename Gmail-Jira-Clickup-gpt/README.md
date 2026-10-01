# IT Workflow Automation

**Gmail + Jira + OpenAI API + Cloudflare Workers + ClickUp**

**Project Status:** Working Prototype / Pre-Security-Hardening

---

## Security Notice

This repository intentionally uses generic placeholders for internal infrastructure information.

The following values have been removed, redacted, or replaced with generic examples for security and privacy reasons:

- Production Worker hostname
- Organization email addresses
- Google Cloud Project ID and Project Number
- Google OAuth client information
- Google Pub/Sub resource identifiers
- ClickUp Workspace and Space IDs
- ClickUp List IDs
- ClickUp Custom Field IDs
- ClickUp Custom Field Option IDs
- Internal account identifiers
- OAuth tokens
- API tokens
- Webhook secrets
- Organization-specific names and identifiers
- Other environment-specific information

Examples in this documentation use placeholders such as:

```text
<WORKER_DOMAIN>
<GOOGLE_PROJECT_ID>
<PUBSUB_TOPIC>
<CLICKUP_LIST_ID>
<CUSTOM_FIELD_ID>
<IT_EMAIL>
```

These placeholders do **not** represent production values.

Production credentials and identifiers are maintained separately through the appropriate administrative platforms and environment configuration.

**No production secret should ever be committed to this repository.**

---

## 1. Project Purpose

IT Workflow Automation is an automation system designed to reduce the manual administrative work involved in IT support.

The system connects:

- Gmail
- Jira Service Management
- OpenAI API
- Cloudflare Workers
- Cloudflare Workers KV
- Google Cloud Pub/Sub
- ClickUp

Instead of manually reviewing every email or Jira ticket and manually re-entering that information into ClickUp, the system can automatically:

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

The long-term goal is to create a centralized and largely automated IT operations workflow while reducing repetitive administrative work.

---

## 2. Current Project Status

### Working

- [x] Gmail → AI → ClickUp
- [x] Jira → AI → ClickUp
- [x] Manual AI Task Creator → ClickUp
- [x] Gmail OAuth
- [x] Gmail API
- [x] Gmail Watch
- [x] Google Cloud Pub/Sub
- [x] Gmail History API
- [x] Cloudflare Workers KV
- [x] Gmail message deduplication
- [x] AI actionable/non-actionable classification
- [x] Structured AI task generation
- [x] ClickUp task creation
- [x] ClickUp custom fields
- [x] ClickUp subtask creation
- [x] Related Gmail message links
- [x] Jira webhook authentication

### Not Yet Complete

- [ ] Gmail Pub/Sub webhook authentication hardening
- [ ] Additional sensitive-data minimization
- [ ] Organization privacy/data-policy review
- [ ] Automatic Gmail Watch renewal
- [ ] Production logging review
- [ ] Credential rotation procedures
- [ ] Final production security review

---

## 3. High-Level Architecture

### Gmail Workflow

```text
Gmail Inbox
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
Cloudflare Worker
    |
    +------------------------+
    |                        |
    v                        v
Gmail History API      Cloudflare KV
    |                        |
    |                        +--> History checkpoint
    |                        +--> Processed Message IDs
    |                        +--> Processing state
    |                        +--> Pub/Sub state
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
```

### Jira Workflow

```text
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
```

### Manual Workflow

```text
IT Workflow Automation UI
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
```

---

## 4. Technologies Used

### Cloudflare

- Cloudflare Workers
- Cloudflare Workers KV
- Worker Secrets

### Google

- Google Cloud
- Google Auth Platform
- OAuth 2.0
- Gmail API
- Gmail History API
- Gmail Watch
- Google Cloud Pub/Sub

### AI

- OpenAI API
- OpenAI Responses API
- Structured Outputs / JSON Schema

### IT Service Management

- Jira Service Management
- Jira Automation

### Task Management

- ClickUp API
- ClickUp Lists
- ClickUp Custom Fields
- ClickUp Subtasks

---

## 5. Cloudflare Worker

The central integration service is implemented as a Cloudflare Worker.

Production hostname:

```text
<WORKER_DOMAIN>
```

Example:

```text
https://<WORKER_DOMAIN>
```

The production hostname has intentionally been removed from this public documentation.

### Worker Routes

| Route | Purpose |
|---|---|
| `/` | Main IT Workflow Automation interface |
| `/status` | Reports configuration status |
| `/test-openai` | Tests OpenAI API connectivity |
| `/google-auth` | Starts Google OAuth authorization |
| `/google-oauth-callback` | Receives Google OAuth callback |
| `/test-gmail` | Tests Gmail API connectivity |
| `/gmail-watch` | Activates or renews Gmail Watch |
| `/gmail-state` | Displays Gmail History state |
| `/gmail-webhook` | Receives Gmail Pub/Sub notifications |
| `/analyze` | Performs manual AI task analysis |
| `/create` | Creates a ClickUp task |
| `/jira-webhook` | Receives Jira Automation events |

---

## 6. Cloudflare Worker Secrets

Production secrets are configured through Cloudflare Worker Secrets.

Required environment secrets:

```text
OPENAI_API_KEY
CLICKUP_API_TOKEN
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
GOOGLE_OAUTH_STATE_SECRET
GMAIL_REFRESH_TOKEN
JIRA_WEBHOOK_SECRET
```

The actual values are **not** included in this repository.

The Worker accesses them through the environment.

Example:

```javascript
env.OPENAI_API_KEY
env.CLICKUP_API_TOKEN
```

Secrets must never be hardcoded into source code.

---

## 7. Cloudflare Workers KV

A Cloudflare Workers KV namespace is bound to the Worker.

Public documentation name:

```text
<GMAIL_STATE_KV_NAMESPACE>
```

Worker binding:

```text
GMAIL_STATE
```

The actual production namespace identifier has intentionally been omitted.

KV maintains Gmail processing state and helps prevent duplicate processing.

### Important KV Keys

```text
gmail:last_history_id

gmail:watch_expiration

gmail:processed:<MESSAGE_ID>

gmail:processing:<MESSAGE_ID>

gmail:pubsub:<PUBSUB_MESSAGE_ID>
```

### Key Purposes

**`gmail:last_history_id`**

Stores the Gmail History ID representing mailbox history already processed.

**`gmail:watch_expiration`**

Stores the expiration timestamp returned by Gmail Watch.

**`gmail:processed:<MESSAGE_ID>`**

Indicates that a Gmail message has already been handled.

**`gmail:processing:<MESSAGE_ID>`**

Temporary state while a message is being processed.

**`gmail:pubsub:<PUBSUB_MESSAGE_ID>`**

Tracks Pub/Sub notification processing.

---

## 8. Processed Message Retention

Processed Gmail Message IDs are currently retained in KV for approximately:

```text
90 days
```

The full Gmail email body is not intentionally stored in KV.

KV primarily contains:

- Message IDs
- History IDs
- Processing state
- Pub/Sub state
- Watch expiration metadata

---

## 9. Original Gmail Duplication Problem

The first Gmail automation implementation used this approach:

```text
Pub/Sub Notification
        |
        v
Cloudflare Worker
        |
        v
Search Recent Inbox Messages
        |
        v
Process Recent Messages
```

This created an important reliability problem.

A Gmail Pub/Sub notification represents a **mailbox change**. It does not necessarily represent one unique email that should be processed exactly once.

Multiple Gmail notifications could occur while the same email remained inside the recent-message search window.

Every notification caused the Worker to search recent Inbox messages again.

As a result, the same email could be:

- Retrieved repeatedly
- Sent to OpenAI repeatedly
- Classified repeatedly
- Converted into multiple ClickUp tasks

During testing, one email generated several similar ClickUp tasks.

The Pub/Sub subscription was temporarily removed while the Gmail architecture was redesigned.

---

## 10. Duplicate Prevention Redesign

The Gmail implementation was redesigned around:

```text
Gmail History API
        +
Cloudflare Workers KV
```

New architecture:

```text
Gmail receives new message
        |
        v
Mailbox history event
        |
        v
Pub/Sub notification
        |
        v
Cloudflare Worker
        |
        v
Read previous History ID from KV
        |
        v
Gmail History API
        |
        v
Find messageAdded events
        |
        v
Extract unique Message IDs
        |
        v
Check KV
        |
        +---- Already processed ---> Skip
        |
        +---- New message
                  |
                  v
            Retrieve Message
                  |
                  v
              Analyze
                  |
                  v
          Create ClickUp Task
                  |
                  v
          Mark Message Processed
```

This architecture successfully resolved the observed duplicate-task problem during testing.

---

## 11. Gmail History API

The Gmail History API is the primary mechanism used to determine which messages are actually new.

The Worker receives:

```text
historyId
```

from Gmail through Pub/Sub.

The Worker retrieves:

```text
gmail:last_history_id
```

from KV.

It then calls Gmail conceptually using:

```text
users.history.list
```

with:

```text
startHistoryId=<LAST_PROCESSED_HISTORY_ID>
historyTypes=messageAdded
labelId=INBOX
```

In practical terms, the Worker asks Gmail:

> Which messages were added to the Inbox after the history point that I already processed?

---

## 12. Gmail History Pagination

Gmail History responses can contain multiple pages.

The Worker supports:

```text
nextPageToken
```

and continues retrieving history until all available pages have been processed.

Message IDs are collected into an in-memory `Set`.

This provides another deduplication layer before processing begins.

---

## 13. Per-Message Processing

For every Message ID returned by Gmail History:

1. Check:

```text
gmail:processed:<MESSAGE_ID>
```

2. If the key exists, skip the message.

3. Otherwise create temporary processing state:

```text
gmail:processing:<MESSAGE_ID>
```

4. Retrieve the Gmail message.

5. Verify that it currently has the `INBOX` label.

6. Parse the message.

7. Check whether it originated from the connected mailbox.

8. Send relevant message content to OpenAI.

9. Determine whether the email represents actionable IT work.

10. If non-actionable, mark it processed.

11. If actionable, create the ClickUp task.

12. Only after successful ClickUp creation, mark the Gmail message processed.

13. Remove temporary processing state.

---

## 14. Failure Handling

An important reliability rule is:

> An actionable Gmail message is not marked successfully processed until ClickUp confirms that its task was created.

Correct sequence:

```text
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
```

If OpenAI or ClickUp fails, the message is not permanently marked as successfully processed.

This allows it to be retried later.

---

## 15. History Checkpoint Advancement

The Worker advances:

```text
gmail:last_history_id
```

only after the relevant Gmail History batch is processed successfully.

Example:

```text
Message A --> Success
Message B --> Success
Message C --> Failure
```

The overall history checkpoint is not advanced.

On retry:

```text
Message A --> Already processed --> Skip
Message B --> Already processed --> Skip
Message C --> Not processed     --> Retry
```

This reduces the possibility of permanently losing an IT request because one operation failed.

---

## 16. Invalid History ID Recovery

Gmail History IDs are not permanent.

If the stored History ID becomes too old, Gmail may reject it.

The Worker detects this condition.

Instead of scanning arbitrary recent emails and risking duplicate task creation, the Worker can reset the processing baseline to the current Gmail History ID.

This design prioritizes safe recovery and duplicate prevention.

---

## 17. Google Cloud Project

The system uses a dedicated Google Cloud project.

Public placeholders:

```text
Project ID:
<GOOGLE_PROJECT_ID>

Project Number:
<GOOGLE_PROJECT_NUMBER>
```

The real production identifiers have intentionally been removed.

The project contains or configures:

- Gmail API
- Google OAuth
- Google Cloud Pub/Sub
- Pub/Sub Topic
- Pub/Sub Subscription
- Gmail publishing permissions

---

## 18. Google OAuth

A Google OAuth Web Application is configured for the Worker.

```text
OAuth Client:
<GOOGLE_OAUTH_CLIENT>

Application Type:
Web Application
```

Authorized redirect URI:

```text
https://<WORKER_DOMAIN>/google-oauth-callback
```

Production OAuth client information and Worker hostname are intentionally omitted.

---

## 19. Gmail OAuth Scope

The Gmail integration currently uses:

```text
https://www.googleapis.com/auth/gmail.readonly
```

The readonly scope was intentionally selected.

The automation needs to:

- Read Gmail messages
- Read Gmail metadata
- Read Gmail History
- Support Gmail Watch processing

The automation does **not** require permission to:

- Send email
- Delete email
- Modify email

Least-privilege access is preferred wherever practical.

---

## 20. Google OAuth Flow

Authorization begins at:

```text
/google-auth
```

The Worker redirects the administrator to Google OAuth.

After authorization, Google redirects to:

```text
/google-oauth-callback
```

The Worker exchanges the temporary authorization code for OAuth tokens.

The resulting refresh token is stored as the Cloudflare secret:

```text
GMAIL_REFRESH_TOKEN
```

The refresh token is **not** stored in this repository.

When Gmail API access is required, the Worker exchanges the refresh token for a temporary Google access token.

---

## 21. OAuth State Protection

The OAuth implementation includes a signed state value.

Secret:

```text
GOOGLE_OAUTH_STATE_SECRET
```

The Worker signs OAuth state using:

```text
HMAC SHA-256
```

The state contains:

- Timestamp
- Cryptographic signature

When Google redirects back to the Worker, the Worker verifies:

- State exists
- Signature is valid
- State has not expired

The state lifetime is approximately:

```text
15 minutes
```

This helps protect the OAuth flow against forged callback requests and CSRF-style OAuth attacks.

---

## 22. Gmail Connection Test

The Gmail connection can be tested through:

```text
/test-gmail
```

The endpoint calls:

```text
users.getProfile
```

A successful response confirms that:

- Google OAuth credentials are functioning
- The refresh token is valid
- Gmail API is enabled
- The Worker can obtain an access token
- The intended mailbox is accessible

The production mailbox address is intentionally omitted.

Public placeholder:

```text
<IT_EMAIL>
```

---

## 23. Google Cloud Pub/Sub

Google Cloud Pub/Sub provides event delivery between Gmail and the Cloudflare Worker.

Topic:

```text
<PUBSUB_TOPIC>
```

Conceptual full resource:

```text
projects/<GOOGLE_PROJECT_ID>/topics/<PUBSUB_TOPIC>
```

Production resource identifiers are intentionally omitted.

---

## 24. Gmail Pub/Sub Publisher

Gmail requires permission to publish notifications into the configured Pub/Sub Topic.

The Google-managed Gmail publishing identity is granted:

```text
Pub/Sub Publisher
```

on the appropriate Topic.

---

## 25. Pub/Sub Subscription

A Push Subscription connects the Pub/Sub Topic to the Cloudflare Worker.

Subscription:

```text
<PUBSUB_SUBSCRIPTION>
```

Delivery type:

```text
Push
```

Push endpoint:

```text
https://<WORKER_DOMAIN>/gmail-webhook
```

Payload unwrapping:

```text
Disabled
```

The Worker therefore receives the normal Google Pub/Sub message envelope.

---

## 26. Pub/Sub Message Format

Conceptually, the Worker receives:

```json
{
  "message": {
    "data": "<BASE64_GMAIL_NOTIFICATION>",
    "messageId": "<PUBSUB_MESSAGE_ID>"
  },
  "subscription": "<SUBSCRIPTION>"
}
```

The Worker decodes:

```text
message.data
```

The decoded Gmail notification contains information such as:

```text
emailAddress
historyId
```

---

## 27. Gmail Watch

Gmail Watch is activated through:

```text
/gmail-watch
```

The Worker calls:

```text
users.watch
```

using the configured Pub/Sub Topic.

Current label filtering:

```text
INBOX
```

Behavior:

```text
INCLUDE
```

The automation is therefore interested primarily in Inbox-related changes.

---

## 28. Gmail Watch State

When Gmail Watch is successfully activated, Gmail returns:

```text
historyId
expiration
```

The Worker stores these values in KV.

Conceptually:

```text
gmail:last_history_id = <HISTORY_ID>

gmail:watch_expiration = <EXPIRATION_TIMESTAMP>
```

Production History IDs and timestamps are intentionally omitted.

---

## 29. Gmail Watch Baseline

When Gmail Watch is activated, the returned History ID establishes the automation baseline.

```text
Existing Mail
     |
     | Gmail Watch activated
     v
-----------------------------
       BASELINE
-----------------------------
     |
     v
New Mail
     |
     v
Automation
```

Existing Inbox messages are therefore not automatically interpreted as newly-arriving requests.

---

## 30. Gmail Watch Expiration

Gmail Watch registrations expire periodically.

The current implementation allows manual activation or renewal through:

```text
/gmail-watch
```

Automatic renewal has not yet been implemented.

Planned future architecture:

```text
Cloudflare Scheduled Trigger
        |
        v
Check Gmail Watch
        |
        v
Renew Gmail Watch
        |
        v
Gmail users.watch
```

---

## 31. Gmail Message Retrieval

New messages identified through Gmail History are retrieved using:

```text
users.messages.get
```

Format:

```text
full
```

Useful headers extracted include:

- Subject
- From
- To
- Cc
- Date
- Message-ID

---

## 32. Email Body Extraction

The Worker attempts to retrieve:

```text
text/plain
```

before falling back to:

```text
text/html
```

If HTML is used, the content is converted into plain text before AI analysis.

---

## 33. Email Size Limiting

Large email bodies are truncated before AI analysis.

Current approximate maximum:

```text
15,000 characters
```

This reduces:

- Token usage
- API cost
- Large quoted email chains
- Unnecessary context

Additional data minimization is planned.

---

## 34. Self-Sent Email Handling

The Worker compares the `From` header with the connected mailbox.

Messages originating from the connected IT mailbox can be ignored and marked as processed.

This helps prevent the IT account's own messages from becoming new tasks.

---

## 35. OpenAI API

The automation uses the OpenAI Responses API.

Endpoint:

```text
https://api.openai.com/v1/responses
```

Production model:

```text
<OPENAI_MODEL>
```

The API key is stored as:

```text
OPENAI_API_KEY
```

The actual API key is never included in source code or public documentation.

---

## 36. OpenAI Storage Setting

Responses API requests currently specify:

```javascript
store: false
```

This configuration should remain enabled unless the data-handling design is deliberately changed.

---

## 37. Structured AI Output

The system does not rely on free-form AI prose for automation.

OpenAI Structured Outputs / JSON Schema are used.

For an actionable request, the expected structure is approximately:

```json
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
```

This makes AI output predictable enough for programmatic processing.

---

## 38. Email Actionability Classification

Every new Gmail message is analyzed individually.

The primary decision is:

```text
isTask = true
```

or:

```text
isTask = false
```

Conceptual actionable response:

```json
{
  "isTask": true,
  "reason": "...",
  "task": {
    "...": "..."
  }
}
```

Conceptual non-actionable response:

```json
{
  "isTask": false,
  "reason": "...",
  "task": null
}
```

---

## 39. Actionable Email Examples

Examples include:

- Computer problems
- Chromebook problems
- Mac problems
- Printer problems
- Projector problems
- Wi-Fi problems
- Network problems
- Login/account problems
- Software problems
- Device deployment requests
- Configuration requests
- Installation requests
- IT purchasing requests
- Vendor communication requiring IT action
- Documentation requests
- Follow-ups requiring additional IT work

---

## 40. Non-Actionable Email Examples

Examples include:

- Newsletters
- Marketing
- Spam
- General announcements
- Informational automated notifications
- Thank-you messages where no work remains
- Replies requiring no additional IT action
- Calendar notifications with no IT action

If an email is classified as non-actionable, no ClickUp task is created.

The Gmail message is still marked as processed so it is not repeatedly analyzed.

---

## 41. AI Accuracy Rules

The AI instructions include rules to:

- Never invent facts
- Never invent locations
- Never invent deadlines
- Never invent users
- Never invent devices
- Never invent causes
- Never present speculative solutions as established facts
- Use an empty location when unknown
- Use an empty Waiting On value when appropriate
- Avoid unnecessary subtasks
- Ignore irrelevant signatures and footer content
- Avoid creating busywork
- Separate factual request information from technical recommendations

---

## 42. ClickUp Integration

ClickUp acts as the central operational task-management destination.

Production identifiers have intentionally been removed.

```text
Workspace:
<CLICKUP_WORKSPACE>

Workspace ID:
<CLICKUP_WORKSPACE_ID>

Space:
<CLICKUP_SPACE>

Space ID:
<CLICKUP_SPACE_ID>
```

---

## 43. ClickUp Lists

### Support Tickets

```text
<CLICKUP_SUPPORT_LIST_ID>
```

### IT Projects

```text
<CLICKUP_PROJECTS_LIST_ID>
```

### Maintenance

```text
<CLICKUP_MAINTENANCE_LIST_ID>
```

### Devices & Deployments

```text
<CLICKUP_DEVICES_LIST_ID>
```

### Waiting / Follow-up

```text
<CLICKUP_WAITING_LIST_ID>
```

### Documentation

```text
<CLICKUP_DOCUMENTATION_LIST_ID>
```

All production List IDs have intentionally been removed.

---

## 44. ClickUp List Routing

**Support Tickets**

Normal end-user IT support issues.

**IT Projects**

Substantial multi-step IT projects.

**Maintenance**

Recurring or preventive IT work.

**Devices & Deployments**

Device preparation, deployment, replacement, setup, or enrollment.

**Waiting / Follow-up**

Tasks primarily blocked by another person, vendor, purchase, approval, or external dependency.

**Documentation**

Tasks where documentation or a guide is the primary deliverable.

---

## 45. ClickUp Categories

Allowed categories:

```text
Support
Project
Maintenance
Device / Deployment
Documentation
```

`Waiting` is intentionally not represented as a Category.

Waiting state is represented separately through:

```text
Waiting On
```

---

## 46. ClickUp Priorities

| AI Priority | ClickUp Value |
|---|---:|
| Urgent | 1 |
| High | 2 |
| Normal | 3 |
| Low | 4 |

### Priority Logic

**Urgent**

- Major outage
- Security incident
- Safety-related technology incident
- Widespread service failure
- Work-stopping issue requiring immediate response

**High**

- Core operations significantly affected
- Testing or scheduled activities affected
- Multiple users affected
- Significant productivity impact
- Clearly time-sensitive request

**Normal**

- Typical IT request
- Limited user impact
- No major urgency

**Low**

- Minor improvement
- Cosmetic issue
- Optional request
- Non-time-sensitive work

---

## 47. ClickUp Custom Fields

Production ClickUp Custom Field IDs and Option IDs have intentionally been removed.

### Waiting On

```text
Field ID:
<CUSTOM_FIELD_WAITING_ON>
```

Allowed values:

- User
- Vendor
- Purchase
- Management
- Facilities
- Other

### Category

```text
Field ID:
<CUSTOM_FIELD_CATEGORY>
```

Allowed values:

- Support
- Project
- Maintenance
- Device / Deployment
- Documentation

### Related Email

```text
Field ID:
<CUSTOM_FIELD_RELATED_EMAIL>

Type:
URL
```

### Location

```text
Field ID:
<CUSTOM_FIELD_LOCATION>

Type:
Short Text
```

### Ticket URL

```text
Field ID:
<CUSTOM_FIELD_TICKET_URL>

Type:
URL
```

### Next Action

```text
Field ID:
<CUSTOM_FIELD_NEXT_ACTION>

Type:
Short Text
```

---

## 48. ClickUp Description Design

The AI-generated Description contains the factual request summary.

The AI-generated Recommendation is appended separately.

Conceptually:

```text
<REQUEST DESCRIPTION>

Recommendation:

<AI TECHNICAL RECOMMENDATION>
```

This intentionally separates:

**What the user actually reported**

from:

**What the AI recommends doing about it**

---

## 49. ClickUp Subtasks

OpenAI may generate subtasks when they provide meaningful operational value.

The Worker first creates the parent ClickUp task.

It then creates each subtask as a child of that parent.

Example:

```text
Parent:

Troubleshoot printer offline


Subtasks:

- Verify printer power/network connection
- Verify printer IP address
- Test connectivity
- Check workstation print queue
- Perform test print
```

Subtasks are not required for every request.

---

## 50. Related Gmail Message

For Gmail-generated tasks, the Worker creates a Gmail link associated with the original Message ID.

Conceptual format:

```text
https://mail.google.com/mail/u/0/#all/<MESSAGE_ID>
```

The URL is placed into the ClickUp `Related Email` custom field.

This allows the technician to move from ClickUp back to the original Gmail conversation.

---

## 51. Jira Service Management

Jira Service Management is another input source for the automation.

Production project information has been generalized.

```text
Project:
<JIRA_PROJECT>

Project Key:
<JIRA_PROJECT_KEY>
```

Jira Automation trigger:

```text
Work item created
```

Action:

```text
Send web request
```

Destination:

```text
https://<WORKER_DOMAIN>/jira-webhook
```

Method:

```text
POST
```

---

## 52. Jira Webhook Payload

Jira sends structured JSON similar to:

```json
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
```

---

## 53. Jira Webhook Authentication

The Jira webhook includes a custom authentication header.

Public documentation placeholder:

```text
X-Automation-Webhook-Secret
```

The corresponding Worker secret is represented as:

```text
JIRA_WEBHOOK_SECRET
```

The production header/value configuration is not included in this repository.

The Worker verifies the provided value before processing the event.

Invalid requests receive:

```text
HTTP 401 Unauthorized
```

---

## 54. Jira AI Processing

Useful Jira information is converted into AI input.

This may include:

- Ticket Key
- Summary
- Description
- Request Type
- Issue Type
- Jira Priority
- Reporter

OpenAI generates the same standardized task structure used by the Gmail and manual workflows.

The original Jira ticket URL is written into the ClickUp `Ticket URL` field.

---

## 55. Manual AI Task Creator

The Cloudflare Worker also hosts a manual interface.

The interface is referred to in this documentation as:

```text
IT Workflow Automation
```

It allows IT staff to create AI-assisted tasks that did not originate from Gmail or Jira.

Examples:

- Verbal user requests
- Problems discovered during troubleshooting
- Infrastructure improvement ideas
- Maintenance work
- Device deployments
- Documentation tasks
- Administrative IT projects

---

## 56. Manual AI Workflow

```text
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
```

---

## 57. Manual Task Preview

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

This provides human review before manually generated tasks are submitted.

---

## 58. System Status

The Worker includes:

```text
/status
```

A healthy system conceptually reports:

```json
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
```

This endpoint reports configuration presence.

It does **not** return secret values.

---

## 59. OpenAI Connection Test

Endpoint:

```text
/test-openai
```

Purpose:

- Verify `OPENAI_API_KEY`
- Verify OpenAI API connectivity
- Verify the configured model is available
- Verify the Worker can receive a valid response

---

## 60. Gmail Connection Test

Endpoint:

```text
/test-gmail
```

Purpose:

- Verify Google OAuth credentials
- Verify the refresh token
- Verify access-token generation
- Verify Gmail API connectivity
- Verify mailbox access

Mailbox information is intentionally excluded from public documentation.

---

## 61. Successful End-to-End Gmail Test

After implementing Gmail History API and KV deduplication, the complete workflow was tested.

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
17. Relevant custom fields and subtasks were created.
18. The Worker marked the Gmail Message ID as processed.
19. The Worker advanced the Gmail History checkpoint.
20. Exactly **one** ClickUp task was created.

This confirmed that the previously observed duplicate-task problem had been resolved during testing.

---

## 62. Current Data Flow

For an actionable Gmail message:

```text
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
```

The complete Gmail body is not intentionally stored in Workers KV.

---

## 63. Security Controls Already Present

The project already includes several security controls:

1. Secrets are stored outside source code using Cloudflare Worker Secrets.
2. Gmail uses a readonly OAuth scope.
3. Google OAuth state is cryptographically signed.
4. OAuth state has a short expiration period.
5. Jira webhook requests require a secret.
6. OpenAI requests currently use `store: false`.
7. Gmail email bodies are not intentionally persisted in Workers KV.
8. Public documentation does not contain production secrets.
9. Infrastructure identifiers in this public README are intentionally generalized.
10. Organization-specific names and identifiers are intentionally excluded.

---

## 64. Security Hardening Not Yet Complete

The system is functional, but final security hardening has not yet been completed.

Primary remaining areas:

- [ ] Gmail Pub/Sub webhook authentication
- [ ] AI data minimization
- [ ] Organization privacy/data-policy review
- [ ] Production logging review
- [ ] Credential rotation procedures
- [ ] Automatic Gmail Watch renewal
- [ ] Final production security assessment

---

## 65. Gmail Webhook Security

Current endpoint:

```text
POST /gmail-webhook
```

Conceptual production URL:

```text
https://<WORKER_DOMAIN>/gmail-webhook
```

The endpoint is internet-accessible.

The current Gmail Pub/Sub Push configuration has not yet received final authentication hardening.

This is a known security item.

Future implementation should verify that webhook requests genuinely originate through the intended Google Pub/Sub path.

Potential approaches include:

- Google Pub/Sub authenticated push
- OIDC-based verification
- Additional request validation
- Appropriate Cloudflare-side access controls

This should be completed before considering the system fully hardened for production.

---

## 66. AI Data Minimization

The current system sends relevant email content to OpenAI so that the AI can determine whether a message represents an IT task.

Future improvements should reduce unnecessary information before external processing.

Potential improvements include:

- Remove quoted email chains
- Remove signatures
- Remove unnecessary email addresses
- Remove unrelated conversation history
- Redact sensitive identifiers when practical
- Avoid processing attachments unless explicitly required
- Restrict AI processing to appropriate IT-related content
- Define which organizational data categories are permitted to be processed

---

## 67. Data Privacy

Technical security does not automatically equal organizational authorization.

Before processing potentially sensitive information, the production deployment should consider applicable:

- Organization policies
- User privacy requirements
- Data-processing agreements
- Vendor agreements
- Administrative approval
- Applicable legal or regulatory requirements

The project should use data minimization wherever practical.

---

## 68. Secret Management

Secrets should continue to be managed through Cloudflare Worker Secrets or an equivalent secure secret-management mechanism.

Secrets should never appear in:

- Git repositories
- README files
- Source code
- Screenshots
- Support tickets
- Application logs
- Public documentation

Secrets should be rotated when appropriate.

---

## 69. Logging

Production logging should avoid unnecessarily recording:

- Email bodies
- Personally identifiable information
- Private user information
- OAuth access tokens
- OAuth refresh tokens
- API keys
- Webhook secrets
- Sensitive request payloads

Operational logging should follow this principle:

> Log enough to troubleshoot the automation, but not enough to unnecessarily reproduce private content.

---

## 70. Credential Rotation

Future operational documentation should define procedures for rotating:

```text
OPENAI_API_KEY
CLICKUP_API_TOKEN
GOOGLE_CLIENT_SECRET
GOOGLE_OAUTH_STATE_SECRET
GMAIL_REFRESH_TOKEN
JIRA_WEBHOOK_SECRET
```

The documentation should also describe what services must be restarted or reauthorized after each credential is rotated.

---

## 71. Gmail Watch Automation

Gmail Watch expires periodically.

Currently it can be renewed manually.

Future implementation should automate renewal.

```text
Cloudflare Cron Trigger
        |
        v
Scheduled Worker
        |
        v
Check Watch Expiration
        |
        v
Renew Gmail Watch
```

This reduces the risk of Gmail automation silently stopping because a Watch expired.

---

## 72. KV Consistency Consideration

Cloudflare Workers KV provides practical state storage and deduplication for the current low-volume IT workflow.

However, Workers KV is eventually consistent.

It is not a transactional database and should not be treated as a strict distributed lock.

The current architecture significantly improves reliability compared with the original recent-message scanning implementation.

If future requirements demand stronger exact-once processing guarantees, possible alternatives include:

- Cloudflare Durable Objects
- Cloudflare D1 with a unique constraint

---

## 73. Current Reliability Model

The system uses multiple layers to reduce duplicate Gmail tasks.

### Layer 1 — Gmail History API

Only mailbox changes after the previous checkpoint are requested.

### Layer 2 — In-Memory Set

Duplicate Message IDs inside a History response are collapsed.

### Layer 3 — Processed Message State

```text
gmail:processed:<MESSAGE_ID>
```

Previously completed messages are skipped.

### Layer 4 — Processing State

```text
gmail:processing:<MESSAGE_ID>
```

Temporary processing state reduces overlapping work.

### Layer 5 — Pub/Sub State

```text
gmail:pubsub:<PUBSUB_MESSAGE_ID>
```

Previously handled Pub/Sub notifications can be recognized.

Together, these provide practical duplicate prevention for the current workflow.

---

## 74. GitHub Security Rules

### Never Commit

```text
OPENAI_API_KEY
CLICKUP_API_TOKEN
GOOGLE_CLIENT_SECRET
GOOGLE_OAUTH_STATE_SECRET
GMAIL_REFRESH_TOKEN
JIRA_WEBHOOK_SECRET
```

Also never commit:

- OAuth access tokens
- Refresh tokens
- Production API tokens
- Private email contents
- Personally identifiable information
- Sensitive organizational information
- Production logs containing private information
- Screenshots containing credentials
- Local secret files

---

## 75. Recommended `.gitignore`

At minimum:

```gitignore
.env
.env.*
.dev.vars
*.secret
secrets.json
```

Additional IDE and runtime-specific exclusions can be added as necessary.

---

## 76. Public vs. Private Configuration

### Appropriate for the Public Repository

- Application source code
- Generic architecture
- Generic documentation
- Setup instructions
- Environment variable names
- Placeholder resource identifiers

### Keep Private

- Production credentials
- OAuth tokens
- Refresh tokens
- Webhook secrets
- Sensitive organizational data
- Unnecessary internal infrastructure identifiers

---

## 77. Example Public Configuration

A public example configuration can use placeholders:

```text
WORKER_DOMAIN=<WORKER_DOMAIN>

GOOGLE_PROJECT_ID=<GOOGLE_PROJECT_ID>

PUBSUB_TOPIC=<PUBSUB_TOPIC>

PUBSUB_SUBSCRIPTION=<PUBSUB_SUBSCRIPTION>

CLICKUP_WORKSPACE_ID=<CLICKUP_WORKSPACE_ID>

CLICKUP_SUPPORT_LIST_ID=<CLICKUP_SUPPORT_LIST_ID>

OPENAI_MODEL=<OPENAI_MODEL>
```

Secret variables should contain placeholders only:

```text
OPENAI_API_KEY=<SECRET>

CLICKUP_API_TOKEN=<SECRET>

GOOGLE_CLIENT_SECRET=<SECRET>

GMAIL_REFRESH_TOKEN=<SECRET>

JIRA_WEBHOOK_SECRET=<SECRET>
```

---

## 78. Complete End-to-End Architecture

```text
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
```

---

## 79. Project Phases

### Phase 1 — Core Automation

- [x] Cloudflare Worker
- [x] ClickUp integration
- [x] OpenAI integration
- [x] Manual AI Task Creator
- [x] Jira automation

### Phase 2 — Gmail Automation

- [x] Google Cloud project
- [x] OAuth
- [x] Gmail API
- [x] Gmail Watch
- [x] Pub/Sub Topic
- [x] Pub/Sub Subscription
- [x] Gmail → Worker
- [x] Gmail → AI
- [x] AI → ClickUp

### Phase 3 — Gmail Reliability

- [x] Identify duplicate-task problem
- [x] Disable faulty subscription during troubleshooting
- [x] Create Workers KV state storage
- [x] Replace recent-message scanning
- [x] Implement Gmail History API
- [x] Implement Message ID deduplication
- [x] Implement processing state
- [x] Implement History checkpoint
- [x] Re-enable Pub/Sub
- [x] Perform end-to-end test
- [x] Confirm one test email creates one task

### Phase 4 — Security Hardening

- [ ] Authenticate Gmail webhook
- [ ] Improve data minimization
- [ ] Review sensitive information handling
- [ ] Review production logs
- [ ] Establish credential rotation procedures
- [ ] Final security assessment

### Phase 5 — Operational Reliability

- [ ] Automatic Gmail Watch renewal
- [ ] Monitoring and alerting
- [ ] Error reporting
- [ ] Recovery documentation
- [ ] Production runbook
- [ ] Backup/recovery strategy for configuration

---

## 80. Future Improvements

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
- ClickUp duplicate detection
- Jira duplicate detection
- Stronger transactional state storage
- Durable Objects or D1
- Automated secret rotation procedures
- Administrative audit logging
- Automated workflow health notifications

---

## 81. Design Principles

### 1. Automate Repetitive Administration

IT time should be spent solving problems rather than repeatedly copying information between systems.

### 2. Keep a Human-Readable Task System

ClickUp remains the central place where IT work can be reviewed and managed.

### 3. Use AI for Classification and Organization

AI assists with understanding and structuring requests rather than directly performing unrestricted administrative actions.

### 4. Use Structured Output

AI responses are constrained through JSON Schema rather than relying on free-form text.

### 5. Minimize Permissions

For example, Gmail currently uses readonly access rather than broader Gmail permissions.

### 6. Confirm Downstream Success

A Gmail message is not marked processed until required downstream work succeeds.

### 7. Prevent Duplicate Work

History checkpoints and per-message state reduce duplicate tasks.

### 8. Never Store Secrets in Source Code

Credentials belong in secure environment configuration.

### 9. Document Known Security Limitations

Known security issues should be documented and fixed rather than hidden.

### 10. Minimize Sensitive Data

Only information necessary for the workflow should be processed or retained.

---

## 82. Project Summary

IT Workflow Automation is an AI-assisted IT workflow system connecting:

- Gmail
- Jira Service Management
- Cloudflare Workers
- Google Cloud Pub/Sub
- Gmail History API
- Cloudflare Workers KV
- OpenAI API
- ClickUp

The system can automatically identify incoming IT work, determine whether it requires action, classify it, prioritize it, generate operational information, and create organized ClickUp tasks.

The original Gmail implementation relied on scanning recent Inbox messages after every Pub/Sub notification.

That approach resulted in duplicate ClickUp tasks.

The Gmail architecture was redesigned around:

```text
Gmail History API
        +
Cloudflare Workers KV
        +
Per-Message Processing State
        +
History Checkpoints
```

After the redesign, end-to-end testing successfully produced one ClickUp task from one new test email without the previously observed duplication.

The core automation and reliability phases are functional.

The next major project phase is:

**Security Hardening**

This includes:

- Authenticating Gmail webhook traffic
- Reducing unnecessary AI data exposure
- Reviewing organizational data/privacy requirements
- Improving logging practices
- Establishing credential rotation procedures
- Automating Gmail Watch renewal
- Performing a final production security review

---

## 83. Public Repository Disclaimer

This repository documents the architecture and implementation of **IT Workflow Automation** while intentionally excluding production-sensitive and organization-specific information.

Names and descriptions of technologies and APIs are retained where useful for understanding the architecture.

Organization names, email addresses, internal hostnames, and infrastructure-specific identifiers have been removed or replaced with generic placeholders.

Examples include:

```text
<WORKER_DOMAIN>

<GOOGLE_PROJECT_ID>

<GOOGLE_PROJECT_NUMBER>

<PUBSUB_TOPIC>

<PUBSUB_SUBSCRIPTION>

<IT_EMAIL>

<CLICKUP_WORKSPACE_ID>

<CLICKUP_SPACE_ID>

<CLICKUP_LIST_ID>

<CUSTOM_FIELD_ID>
```

These placeholders are intentional.

They should **not** be replaced with production values in a public repository.

Production configuration should remain in the appropriate secured administrative systems and environment configuration.
