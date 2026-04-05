# Workflow 9: AI Personal Assistant

## Overview

This is a Slack-powered AI personal assistant that can manage your CRM, calendar, and email on command. When you @mention the bot in Slack, a Dynamic AI Selector agent first picks the best LLM for the task (Gemini Flash for simple queries, GPT-4.1 Mini for tool calls, etc.), then the main Agentic PA routes the request to one of three specialized sub-agents: a CRM Agent (HubSpot lookups and updates), a Calendar Agent (Google Calendar events), or an Email Agent (Gmail read/send/draft). It also has Perplexity integration for real-time web searches and an SMS sending tool.

**Important:** This workflow consists of 4 separate n8n workflows that must all be imported. The parent orchestrator calls the three sub-agents via "Execute Workflow" tool nodes, so the sub-workflows need to be imported first and their IDs linked in the parent.

---

## Required Services

- [ ] **Slack** -- https://slack.com/ (workspace with a Slack app configured)
- [ ] **OpenRouter** -- https://openrouter.ai/ (multi-model LLM routing)
- [ ] **OpenAI** -- https://platform.openai.com/ (GPT model for sub-agents and dynamic selector)
- [ ] **Perplexity** -- https://www.perplexity.ai/ (real-time web search)
- [ ] **HubSpot** -- https://www.hubspot.com/ (CRM -- used by the CRM sub-agent)
- [ ] **Gmail** -- Google account with Gmail API access (used by the Email sub-agent)
- [ ] **Google Calendar** -- Google account with Calendar API access (used by the Calendar sub-agent)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Slack OAuth | Slack App > OAuth & Permissions | Slack OAuth2 |
| OpenRouter API Key | openrouter.ai > Keys | OpenRouter API |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |
| Perplexity API Key | perplexity.ai > API Settings | Perplexity API |
| HubSpot OAuth | n8n Credentials > HubSpot > OAuth2 | HubSpot OAuth2 |
| Gmail OAuth | n8n Credentials > Gmail > OAuth2 | Gmail OAuth2 |
| Google Calendar OAuth | n8n Credentials > Google Calendar > OAuth2 | Google Calendar OAuth2 |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_OPENROUTER_API_KEY` | OpenRouter key for dynamic model selection | Gemini 2.0 Flash node (parent workflow) |
| `YOUR_OPENAI_API_KEY` | OpenAI key for sub-agents and selector | OpenAI Chat Model nodes (all 4 workflows) |
| `YOUR_PERPLEXITY_API_KEY` | Perplexity key for web search | Perplexity Tool node (parent workflow) |
| `YOUR_SLACK_CREDENTIAL` | Slack OAuth credential name | Slack Trigger + Send a Message nodes |
| `YOUR_SLACK_CHANNEL_ID` | Channel ID the bot listens in | Slack Trigger node config |
| `YOUR_HUBSPOT_CREDENTIAL` | HubSpot OAuth credential name | CRM Agent sub-workflow |
| `YOUR_GMAIL_CREDENTIAL` | Gmail OAuth credential name | Email Agent sub-workflow |
| `YOUR_GOOGLE_CALENDAR_CREDENTIAL` | Calendar OAuth credential name | Calendar Agent sub-workflow |
| `YOUR_GOOGLE_CALENDAR_EMAIL` | Calendar email to manage | Calendar Agent event nodes |

---

## Workflow Files & Import Order

You **must** import the sub-agents first, then the parent workflow. Here are the four JSON files:

| Order | File | Workflow Name | Role |
|-------|------|---------------|------|
| 1 | `CRM_Agent__PA_(1).json` | CRM Agent (PA) | Sub-agent: HubSpot contact lookups and updates |
| 2 | `__Calendar_Agent(1).json` | Calendar Agent (PA) | Sub-agent: Google Calendar event CRUD |
| 3 | `__Email_Agent(1).json` | Email Agent (PA) | Sub-agent: Gmail send, read, draft, label, reply |
| 4 | `AI_Personal_Assistant(1)(1).json` | AI Personal Assistant | Parent orchestrator with Slack trigger |

### Linking Sub-Workflow IDs

After importing all four workflows:

1. Open each sub-agent workflow in n8n and note its **Workflow ID** (visible in the URL: `https://your-n8n.com/workflow/WORKFLOW_ID`)
2. Open the parent **AI Personal Assistant** workflow
3. Click on the **CRM Agent** tool node > Settings > Workflow ID > Select or paste the CRM Agent workflow ID
4. Click on the **Calendar Agent** tool node > Settings > Workflow ID > Select or paste the Calendar Agent workflow ID
5. Click on the **Email Agent** tool node > Settings > Workflow ID > Select or paste the Email Agent workflow ID

If n8n auto-detects the imported workflows in the dropdown, you can simply select them by name.

---

## Node-by-Node Configuration

### Parent Workflow: AI Personal Assistant

#### Trigger
1. **Slack Trigger** -- Fires on `app_mention` events in your specified Slack channel.
   - Set your Slack OAuth credential
   - Set the channel ID where the bot should listen
   - The bot responds when you type `@YourBotName do something`

#### Dynamic Model Selection
2. **Dynamic AI Selector** -- An AI agent that reads the user's request and picks the best LLM:
   - `google/gemini-2.0-flash-001` -- Fast, lightweight, conversational tasks
   - `openai/gpt-4.1-mini` -- Tool calling, CRM lookups, calendar events
   - Other models can be added to the system prompt
   - Uses OpenAI Chat Model for the selection decision itself

#### Main Agent
3. **Agentic Personal Assistant** -- The core agent that processes the request using the selected model:
   - System prompt includes current date/time for scheduling context
   - Has access to these tools: CRM Agent, Calendar Agent, Email Agent, Perplexity, Send SMS

#### Tools
4. **CRM Agent** (Execute Workflow) -- Routes CRM-related queries to the CRM sub-workflow
5. **Calendar Agent** (Execute Workflow) -- Routes calendar queries to the Calendar sub-workflow
6. **Email Agent** (Execute Workflow) -- Routes email queries to the Email sub-workflow
7. **Message a model in Perplexity** -- Real-time web search for questions the AI cannot answer from its training data
8. **Send SMS** (HTTP Request Tool) -- Sends SMS via Contiguity API (optional -- remove if you do not use SMS)

#### Response
9. **Send a Message** (Slack) -- Posts the agent's response back to the same Slack channel

### Sub-Workflow: CRM Agent

1. **When Executed by Another Workflow** -- Trigger (receives input from parent)
2. **Contact Agent** -- AI agent with HubSpot tools:
   - Get a contact in HubSpot
   - Create or update a contact in HubSpot
   - Get a company in HubSpot
3. **Response / Try Again** -- Returns result to parent or retries on failure

### Sub-Workflow: Calendar Agent

1. **When Executed by Another Workflow** -- Trigger
2. **Calendar Agent** -- AI agent with Google Calendar tools:
   - Create Event / Create Event with Attendee
   - Get Events
   - Update Event
   - Delete Event
3. **Success / Try Again** -- Returns result or retries

### Sub-Workflow: Email Agent

1. **When Executed by Another Workflow** -- Trigger
2. **Email Agent** -- AI agent with Gmail tools:
   - Send Email
   - Get Emails
   - Create Draft
   - Email Reply
   - Get Labels / Label Emails
   - Mark Unread
3. **Success / Try Again** -- Returns result or retries

---

## Slack App Setup

1. Go to https://api.slack.com/apps and click **Create New App**
2. Choose **From scratch**, name it (e.g., "AI Assistant"), and select your workspace
3. Go to **OAuth & Permissions** and add these Bot Token Scopes:
   - `app_mentions:read`
   - `chat:write`
   - `channels:history`
   - `channels:read`
4. Install the app to your workspace
5. Copy the **Bot User OAuth Token** (starts with `xoxb-`)
6. In n8n, create a Slack OAuth2 credential using this token
7. Invite the bot to your channel: type `/invite @YourBotName` in the channel

### Finding Your Slack Channel ID

Right-click the channel name in Slack > **View channel details** > scroll to the bottom. The Channel ID looks like `C08XXXXXXXX`.

---

## Testing Steps

1. **Import all 4 workflows** in the correct order (sub-agents first)
2. **Link the sub-workflow IDs** in the parent workflow
3. **Activate all 4 workflows** (they all need to be active)
4. **Test CRM:** In Slack, type `@YourBot look up John Smith in the CRM`
5. **Test Calendar:** Type `@YourBot what meetings do I have tomorrow?`
6. **Test Email:** Type `@YourBot draft an email to john@example.com about our meeting`
7. **Test Perplexity:** Type `@YourBot what is the latest news about AI?`
8. **Check Slack:** Verify the bot responds in the channel with the correct information

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Bot does not respond | Slack app not installed or bot not in channel | Reinstall app; invite bot to channel with `/invite` |
| "Workflow not found" error | Sub-workflow IDs not linked correctly | Open parent workflow, update each tool node's Workflow ID |
| CRM queries fail | HubSpot credential not set in CRM Agent | Open CRM Agent sub-workflow, set HubSpot OAuth credential on each tool node |
| Calendar/email queries fail | Google credentials not set in sub-agents | Set Gmail and Calendar OAuth in the respective sub-workflows |
| Wrong model selected | Dynamic AI Selector prompt needs tuning | Edit the system prompt in the Dynamic AI Selector node to better match your use cases |
| Sub-agents not active | Forgot to activate sub-workflows | All 4 workflows must be active for the system to work |
| Slow responses | Large model selected for simple query | The Dynamic AI Selector should route simple queries to Gemini Flash; adjust its prompt if needed |
