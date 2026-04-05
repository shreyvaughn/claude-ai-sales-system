# Claude AI Sales System -- Workflow Overview

This package contains 12 n8n automation workflows that together form a complete AI-powered sales pipeline. Each workflow handles a specific part of the sales process, and they are designed to work both independently and as an integrated system.

---

## All Workflows at a Glance

| # | Workflow | What It Does | Key Services |
|---|----------|-------------|--------------|
| 1 | Lead Finder & Enrichment | Searches Apollo for leads matching your ICP, enriches data, exports to Google Sheets | Apollo, Google Sheets |
| 2 | Agentic AI SDR | 7 AI sub-agents research leads and make automated phone calls | Apollo, Apify, RetellAI, Unipile, Pinecone, OpenAI |
| 3 | LinkedIn Prospecting Flow | Automated LinkedIn DM outreach with AI-generated voice notes | Unipile, Apollo, ElevenLabs, Apify, Pinecone, Google Sheets |
| 4 | Email Sequencing | Multi-stage cold email campaigns with AI personalization | Gmail, Google Sheets, OpenAI, Pinecone |
| 5 | Email Objection Handling | Monitors reply emails and auto-responds using AI | Gmail, Google Sheets, OpenAI, Pinecone |
| 6 | Pre-Call SMS Follow-up | Sends SMS reminder sequences when someone books a call | Cal.com, Contiguity SMS, OpenAI |
| 7 | Advanced Call Analysis | Classifies call outcomes from transcripts (booked/future/rejected) | RetellAI, Google Sheets, OpenAI |
| 8 | Post-Call Follow Up | Generates personalized follow-up email sequences after calls | SMTP/Gmail, OpenAI |
| 9 | Pinecone Knowledgebase | Ingests company documents into a vector database | Google Drive, Pinecone, OpenAI Embeddings |
| 10 | AI Presentation Creator | Converts call transcripts into Google Slides presentations | Google Slides, Google Drive, OpenAI |
| 11 | Meeting Notes Agent | Generates meeting notes docs, indexes them, notifies team | Google Docs, Pinecone, Slack, OpenAI |
| 12 | AI Personal Assistant | Slack chatbot with CRM, calendar, and email sub-agents | Slack, OpenRouter, Perplexity, HubSpot, Gmail, Google Calendar |
| 13 | Full Newsletter System | Curates RSS content and generates weekly digest newsletters | RSS, Pinecone, OpenAI, Google Docs |

---

## Recommended Setup Order

If you are deploying the full system, set up workflows in this order. Each step builds on the previous one.

### Tier 1: Foundation (set up first)
1. **Pinecone Knowledgebase** (#9) -- All AI agents pull context from this. Load your company docs, FAQs, case studies, and product info here first.
2. **Lead Finder & Enrichment** (#1) -- Populate your Google Sheets lead database. Everything downstream needs leads.

### Tier 2: Outreach Engine
3. **Email Sequencing** (#4) -- Start with email outreach since it requires the fewest external services.
4. **Email Objection Handling** (#5) -- Pair with Email Sequencing to handle replies automatically.
5. **Agentic AI SDR** (#2) -- The most complex workflow. Set this up once you are comfortable with n8n.
6. **LinkedIn Prospecting Flow** (#3) -- Adds a second outreach channel alongside email.

### Tier 3: Call Management
7. **Advanced Call Analysis** (#7) -- Processes call outcomes from the SDR or manual calls.
8. **Pre-Call SMS Follow-up** (#6) -- Reduces no-shows for booked calls.
9. **Post-Call Follow Up** (#8) -- Automates the post-call nurture sequence.

### Tier 4: Productivity & Content
10. **Meeting Notes Agent** (#11) -- Quality-of-life automation for your sales team.
11. **AI Presentation Creator** (#10) -- Generates follow-up decks from call data.
12. **AI Personal Assistant** (#12) -- The most complex multi-workflow setup. Do this last.
13. **Full Newsletter System** (#13) -- Independent system, set up whenever.

---

## How Workflows Connect

```
                    +---------------------------+
                    |   Pinecone Knowledgebase   |
                    |   (shared context layer)   |
                    +---------------------------+
                          |       |       |
              +-----------+   +---+   +---+-----------+
              v               v               v
    +------------------+  +--------+  +------------------+
    | Agentic AI SDR   |  | Email  |  | Email Objection  |
    | (reads context   |  | Seq.   |  | Handling (reads   |
    |  for research)   |  | (reads)|  |  context)         |
    +------------------+  +--------+  +------------------+

LEAD GENERATION
===============
[Lead Finder] ---> Google Sheets (lead DB) <--- [LinkedIn Prospecting]
                          |
            +-------------+-------------+
            |                           |
            v                           v
    [Agentic AI SDR]           [Email Sequencing]
     (phone calls)              (cold emails)
            |                           |
            v                           v
    [Call Analysis]            [Objection Handling]
            |                           |
     +------+------+                    |
     |      |      |                    |
  Booked  Future  Reject         Meeting booked
     |      |                           |
     v      v                           v
    [Post-Call Follow Up]      [Pre-Call SMS Follow-up]
            |                           |
            v                           v
    Follow-up emails           SMS reminders sent
            |                           |
            +-------------+-------------+
                          |
                    Call happens
                          |
            +-------------+-------------+
            |             |             |
            v             v             v
    [Meeting Notes]  [Presentation  [Call Analysis]
      Agent           Creator]        (loop back)

INDEPENDENT SYSTEMS
===================
[AI Personal Assistant] -- Slack chatbot, works standalone
[Full Newsletter] -- Content curation, works standalone
```

---

## Service Dependencies

Here is every external service used across all workflows. You do not need all of them -- only the ones required by your selected workflows.

### AI / LLM
- **OpenAI API** -- Used by most workflows for text generation and embeddings
- **OpenRouter** -- Used by Personal Assistant for model routing
- **Perplexity** -- Used by Personal Assistant for web search
- **Google Gemini** -- Used by Post-Call Follow Up, Meeting Notes, Presentation Creator

### Lead Generation
- **Apollo.io** -- Lead search and enrichment API
- **Apify** -- Web scraping platform (LinkedIn profiles, websites, Google search)

### Communication
- **Gmail** -- Email sending and receiving
- **SMTP** -- Alternative email sending (Post-Call Follow Up)
- **Slack** -- Notifications and Personal Assistant trigger
- **Contiguity** -- SMS sending API
- **RetellAI** -- AI phone call platform
- **Unipile** -- LinkedIn messaging API
- **ElevenLabs** -- AI voice generation for LinkedIn voice notes

### Data & Storage
- **Google Sheets** -- Lead databases and tracking
- **Google Docs** -- Meeting notes output
- **Google Slides** -- Presentation generation
- **Google Drive** -- File storage and retrieval
- **Google Calendar** -- Event management
- **Pinecone** -- Vector database for AI context
- **HubSpot** -- CRM for contact management

### Scheduling
- **Cal.com** -- Booking page and webhook events

---

## Config Files

Detailed setup instructions for each workflow are in the `configs/` directory:

- `01-lead-finder.md` -- Lead Finder & Enrichment
- `02-agentic-sdr.md` -- Agentic AI SDR
- `03-linkedin-prospecting.md` -- LinkedIn Prospecting Flow
- `04-email-sequencing.md` -- Email Sequencing
- `05-email-objection-handling.md` -- Email Objection Handling
- `06-pre-call-sms.md` -- Pre-Call SMS Follow-up
- `07-call-analysis.md` -- Advanced Call Analysis
- `08-post-call-followup.md` -- Post-Call Follow Up
- `09-pinecone-knowledgebase.md` -- Pinecone Knowledgebase
- `10-presentation-creator.md` -- AI Presentation Creator
- `11-meeting-notes.md` -- Meeting Notes Agent
- `12-personal-assistant.md` -- AI Personal Assistant
- `13-newsletter.md` -- Full Newsletter System

---

## n8n Requirements

- **n8n version:** 1.30+ recommended (for latest LangChain node support)
- **Hosting:** Self-hosted or n8n Cloud. Self-hosted gives you more control over webhook URLs and execution limits.
- **Execution mode:** "queue" mode recommended for production use (handles parallel workflow runs better)

## Getting Help

Paste the `CLAUDE.md` file into Claude and it will walk you through setup interactively. The AI assistant knows the structure of every workflow and can generate customized JSON with your API keys and configuration values filled in.
