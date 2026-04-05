# Workflow 2: Agentic AI SDR

## Overview

This is the most complex and powerful workflow in the system. It deploys 7 AI sub-agents that work together to research each lead thoroughly, then makes an automated AI phone call using RetellAI. The sub-agents analyze competitors, verify lead data, research company websites, scrape Twitter/social presence, analyze websites for personalization angles, and compile everything into a briefing before the call.

The workflow reads leads from a Google Sheet, runs them through the research pipeline, and then initiates phone calls with dynamically generated openers personalized to each lead.

---

## Required Services

- [ ] **Apollo.io** -- https://www.apollo.io/ (lead data)
- [ ] **Apify** -- https://apify.com/ (web scraping -- website content, LinkedIn profiles, contact info)
- [ ] **RetellAI** -- https://www.retellai.com/ (AI phone calls)
- [ ] **Unipile** -- https://www.unipile.com/ (LinkedIn messaging for social sniping)
- [ ] **Pinecone** -- https://www.pinecone.io/ (vector DB for company knowledge context)
- [ ] **Google Sheets** -- Lead tracking spreadsheet
- [ ] **OpenAI** -- https://platform.openai.com/ (powers all 7 sub-agents)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Apollo API Key | Apollo Settings > API | HTTP Header Auth |
| Apify API Token | Apify Console > Settings > Integrations | HTTP Header Auth |
| RetellAI API Key | RetellAI Dashboard > API Keys | HTTP Header Auth |
| RetellAI Phone Number | RetellAI Dashboard > Phone Numbers | (used in JSON body) |
| RetellAI Agent ID | RetellAI Dashboard > Agents > your agent | (used in JSON body) |
| Unipile API Key | Unipile Dashboard > API | HTTP Header Auth |
| Unipile DSN | Unipile Dashboard > Connection Info | (used in URL) |
| Pinecone API Key | Pinecone Console > API Keys | Pinecone API |
| Pinecone Index Name | Pinecone Console > Indexes | (used in node config) |
| Google Sheets OAuth | n8n Credentials > Google Sheets | Google Sheets OAuth2 |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_APOLLO_API_KEY` | Apollo API key | Apollo HTTP Request nodes |
| `YOUR_APIFY_API_KEY` | Apify token | Apify scraping nodes (in URL query param `?token=`) |
| `YOUR_RETELL_API_KEY` | RetellAI API key | Authorization header on call nodes |
| `YOUR_RETELL_PHONE_NUMBER` | Your RetellAI phone number | `from_number` field in call body |
| `YOUR_RETELL_AGENT_ID` | RetellAI agent ID | (if using agent-based calls) |
| `YOUR_UNIPILE_API_KEY` | Unipile access token | Authorization header |
| `YOUR_UNIPILE_DSN` | Unipile API endpoint | URL base (e.g., `api11.unipile.com:14142`) |
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store nodes |
| `YOUR_PINECONE_INDEX_NAME` | Pinecone index for company knowledge | Pinecone Vector Store nodes |
| `YOUR_GOOGLE_SHEETS_ID` | Lead tracking sheet ID | Google Sheets nodes |
| `YOUR_OPENAI_API_KEY` | OpenAI key | All AI agent nodes |
| `YOUR_COMPANY_NAME` | Your company name | Call opener scripts |
| `YOUR_NAME` | Your name (or SDR name) | Call opener scripts |

---

## Node-by-Node Configuration

### Trigger & Lead Retrieval
1. **Schedule Trigger** -- Runs daily. Configure time of day.
2. **Google Sheets (get data)** -- Reads leads from your sheet. Set `YOUR_GOOGLE_SHEETS_ID` and sheet tab name. Filters for leads not yet contacted.

### Research Pipeline (7 Sub-Agents)

Each sub-agent is an AI Agent node connected to an OpenAI Chat Model and a specific tool:

3. **Competitor Analysis Agent** -- Uses Apify website scraper to analyze competitor websites found on the lead's company page. Outputs competitive positioning angles.

4. **Lead Verifier Agent** -- Cross-references Apollo data with scraped LinkedIn profile to verify the lead is real, still at the company, and has the right title.

5. **Company Research Agent** -- Scrapes the lead's company website via Apify. Extracts what the company does, recent news, tech stack, and pain points.

6. **Twitter/Social Agent** -- Scrapes the lead's social presence. Looks for recent posts, interests, and engagement topics for personalization.

7. **Website Analysis Agent** -- Deep analysis of the lead's website. Identifies potential needs based on their current stack and gaps.

8. **Social Sniping Agent** -- Uses Unipile to check LinkedIn activity. Identifies connection opportunities.

9. **Data Analysis Agent** -- Compiles all research from agents 3-8. Produces a lead briefing with personalization angles and recommended openers.

### Call Execution

10. **Prepare Lead Data** -- Formats all research into the call payload.
11. **Random Opener Selector** -- Randomly selects 1 of 3 call openers for A/B testing.
12. **Send Call (Opener 1/2/3)** -- HTTP Request to RetellAI API:
    - URL: `https://api.retellai.com/v2/create-phone-call`
    - Authorization: `Bearer YOUR_RETELL_API_KEY`
    - Body includes `retell_llm_dynamic_variables` with company name and opener
    - `from_number`: `YOUR_RETELL_PHONE_NUMBER`
    - `to_number`: from lead data

### Tracking

13. **Google Sheets (update)** -- Marks leads as contacted with timestamp and call status.
14. **Google Sheets (metrics)** -- Updates daily metrics sheet (calls sent, humans picked up, voicemails, appointments booked, cost incurred).

---

## Google Sheet Structure

### Leads Sheet
| Column | Description |
|--------|-------------|
| First Name | Lead first name |
| Last Name | Lead last name |
| Email | Lead email |
| Phone Number | Lead phone (required for calls) |
| Company Website | Company URL |
| LinkedIn URL | Lead LinkedIn profile |
| Stage | Current stage (new/contacted/researched) |
| Last Contact Date | When last contacted |

### Metrics Sheet
| Column | Description |
|--------|-------------|
| Date | YYYY-MM-DD |
| Calls Sent | Total calls made |
| Human Calls Picked | Calls answered by a person |
| Voicemails | Calls that went to voicemail |
| Appointments Booked | Calls resulting in a booking |
| Offer Rejected | Calls where prospect declined |
| Cost Incurred | Total RetellAI cost for the day |

---

## Testing Steps

1. **Test with 1 lead:** Put a single test lead in your Google Sheet (use your own phone number)
2. **Run manually:** Execute the workflow
3. **Check each agent:** Click each AI Agent node to see its research output
4. **Verify the call:** You should receive a test call from your RetellAI number
5. **Check metrics:** Verify the metrics sheet updated
6. **Review call recording:** Check RetellAI dashboard for the call recording and transcript

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Apify scraping fails | Website blocks scraping or rate limit | Add Wait nodes between Apify calls; check Apify logs |
| RetellAI call fails | Invalid phone number format | Ensure phone numbers include country code (e.g., +1xxxxxxxxxx) |
| Unipile LinkedIn error | Account not connected or rate limited | Re-link LinkedIn in Unipile dashboard; reduce daily message volume |
| OpenAI context too long | Research too verbose for context window | Use GPT-4o or truncate research summaries in Data Analysis agent |
| Google Sheets auth error | Token expired | Re-authenticate Google Sheets credential in n8n |
| All agents return empty | Pinecone index empty | Set up Knowledgebase workflow (#9) first to populate context |

---

## Cost Estimates

- **RetellAI:** ~$0.10-0.30 per call (depends on duration)
- **Apify:** ~$0.01-0.05 per scrape (depends on actor)
- **OpenAI:** ~$0.02-0.10 per lead (7 agent calls with GPT-4o)
- **Unipile:** Subscription-based (~$49/mo for LinkedIn access)
- **Total per lead:** approximately $0.15-0.50 depending on research depth
