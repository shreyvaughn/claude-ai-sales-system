# Claude AI Sales System

**12 AI-powered sales automation workflows for n8n. Built from 18,000+ real conversations and $5M+ pipeline.**

Stop doing sales manually. This system automates lead finding, cold outreach, call analysis, follow-ups, and meeting booking -- all running 24/7 on n8n with AI agents doing the heavy lifting.

---

## Quick Start

1. **Clone this repo**
   ```bash
   git clone https://github.com/shreyofficial/claude-ai-sales-system.git
   ```

2. **Open Claude**
   Go to [claude.ai](https://claude.ai), start a new conversation, and paste the contents of `CLAUDE.md`.

3. **Claude walks you through everything**
   It asks which workflows you want, collects your API keys, generates customized n8n JSON, and gives you import + testing instructions.

---

## The 12 Workflows

### Lead Generation & Research
| # | Workflow | What It Does |
|---|---------|-------------|
| 1 | **Lead Finder & Enrichment** | Searches Apollo for leads matching your ICP, enriches data, exports to Google Sheets |
| 2 | **Agentic AI SDR** | 7 AI sub-agents research leads, analyze competitors, scrape websites, and make AI phone calls |
| 3 | **LinkedIn Prospecting Flow** | Automated LinkedIn DM outreach with AI-generated voice notes via ElevenLabs |

### Outreach & Communication
| # | Workflow | What It Does |
|---|---------|-------------|
| 4 | **Email Sequencing** | Multi-stage cold email campaigns with AI-written personalized emails |
| 5 | **Email Objection Handling** | Monitors inbox replies and auto-responds using AI to handle objections |
| 6 | **Pre-Call SMS Follow-up** | Cal.com booking triggers SMS reminder sequences before calls |

### Call Management
| # | Workflow | What It Does |
|---|---------|-------------|
| 7 | **Advanced Call Analysis** | Analyzes call transcripts and classifies outcomes (booked / future / rejected) |
| 8 | **Post-Call Follow Up** | Auto-generates personalized follow-up email sequences after sales calls |

### Knowledge & Content
| # | Workflow | What It Does |
|---|---------|-------------|
| 9 | **Pinecone Knowledgebase** | Ingests your company docs into a vector database for AI context |
| 10 | **AI Presentation Creator** | Converts call transcripts into Google Slides presentations |
| 11 | **Meeting Notes Agent** | Generates meeting notes, saves to Google Docs, indexes in Pinecone, notifies Slack |

### Operations
| # | Workflow | What It Does |
|---|---------|-------------|
| 12 | **AI Personal Assistant** | Slack-triggered PA with CRM, calendar, and email sub-agents |
| 13 | **Newsletter System** | Curates RSS content into Pinecone, generates weekly digest newsletters |

---

## What's Inside

```
claude-ai-sales-system/
├── CLAUDE.md                      # Master setup wizard -- paste this into Claude
├── workflows/
│   ├── README.md                  # Workflow overview, setup order, data flow diagrams
│   ├── json/                      # 15 n8n workflow JSON files (ready to import)
│   │   ├── Research___First_Touchpoint_SDR_(1).json
│   │   ├── Advanced_Call_Analysis__SDR_(1).json
│   │   ├── Email_sequencing_SDR.json
│   │   ├── Email_Back___Forth_Objection_Handling__SDR_Workflow_.json
│   │   ├── Pre_Call_Follow_up_SMS_(1).json
│   │   ├── Post_Call_Follow_Up(1).json
│   │   ├── AI_Personal_Assistant(1)(1).json
│   │   ├── CRM_Agent__PA_(1).json
│   │   ├── __Calendar_Agent(1).json
│   │   ├── __Email_Agent(1).json
│   │   ├── Company_Knowledgebase.json
│   │   ├── presentation_creator(1).json
│   │   ├── 11_Labs__LI_DM_Outreach_Workflow__Guide.json
│   │   ├── meeting_notes(1).json
│   │   └── Newsletter_Management(1).json
│   └── configs/                   # Step-by-step setup guides for each workflow
│       ├── 01-lead-finder.md
│       ├── 02-agentic-sdr.md
│       ├── 03-linkedin-prospecting.md
│       ├── 04-email-sequencing.md
│       ├── 05-email-objection-handling.md
│       ├── 06-pre-call-sms.md
│       ├── 07-call-analysis.md
│       ├── 08-post-call-followup.md
│       ├── 09-ai-personal-assistant.md
│       ├── 10-pinecone-knowledgebase.md
│       ├── 11-presentation-creator.md
│       ├── 12-meeting-notes.md
│       └── 13-newsletter.md
```

---

## Requirements

- **n8n instance** (self-hosted or n8n Cloud) -- v1.30+ recommended
- **API keys** for the services each workflow uses:
  - OpenAI (most workflows)
  - Apollo.io (lead finding)
  - Apify (web scraping)
  - RetellAI (AI phone calls)
  - Unipile (LinkedIn messaging)
  - ElevenLabs (AI voice notes)
  - Gmail / SMTP (email)
  - Google Sheets, Docs, Slides, Drive, Calendar
  - Pinecone (vector database)
  - Slack (notifications + PA trigger)
  - HubSpot (CRM)
  - Cal.com (booking)
  - Contiguity (SMS)
  - OpenRouter, Perplexity (PA sub-agents)

You only need the keys for the workflows you choose to set up. Claude will tell you exactly which ones you need.

---

## How It All Connects

```
[Lead Finder] + [LinkedIn Prospecting]
         |
         v
   Google Sheets (lead DB)
         |
    +----+----+
    |         |
    v         v
[AI SDR]  [Email Sequencing]
(calls)    (cold emails)
    |         |
    v         v
[Call      [Objection
Analysis]   Handling]
    |         |
    v         v
[Post-Call  [Pre-Call
Follow Up]   SMS]
    |         |
    +----+----+
         |
    Call happens
         |
    +----+----+----+
    |    |         |
    v    v         v
[Notes] [Slides] [Analysis]

Independent: [Personal Assistant] [Newsletter] [Knowledgebase]
```

---

## License

MIT
