# Workflow 4: Email Sequencing

## Overview

This workflow runs a multi-stage cold email campaign. It reads leads from a Google Sheet, checks which stage each lead is in (Stage 1 or Stage 2), generates a personalized email using AI (with context from your Pinecone knowledgebase), and sends it via Gmail. It runs on a daily schedule and respects a minimum 2-day gap between emails to avoid spam-like behavior. Each stage has a different email angle and tone.

---

## Required Services

- [ ] **Gmail** -- Google account with Gmail API access
- [ ] **Google Sheets** -- Lead tracking spreadsheet
- [ ] **OpenAI** -- https://platform.openai.com/ (email generation)
- [ ] **Pinecone** -- https://www.pinecone.io/ (company knowledge for personalization)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Gmail OAuth | n8n Credentials > Gmail > OAuth2 | Gmail OAuth2 |
| Google Sheets OAuth | n8n Credentials > Google Sheets > OAuth2 | Google Sheets OAuth2 |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |
| Pinecone API Key | Pinecone Console > API Keys | Pinecone API |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_GMAIL_CREDENTIAL` | Gmail OAuth credential name | Gmail send nodes |
| `YOUR_GOOGLE_SHEETS_ID` | Lead tracking sheet ID | Google Sheets read/write nodes |
| `YOUR_OPENAI_API_KEY` | OpenAI key | AI Agent nodes |
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store nodes |
| `YOUR_PINECONE_INDEX_NAME` | Index for company knowledge | Pinecone node config |
| `YOUR_COMPANY_NAME` | Your company name | Email copy context |
| `YOUR_NAME` | Sender name | Email signatures |
| `YOUR_SENDER_EMAIL` | Gmail address to send from | Gmail node "from" field |

---

## Node-by-Node Configuration

### Trigger & Data Retrieval
1. **Schedule Trigger** -- Runs daily at 9:30 AM. Configurable.
2. **Google Sheets (get data)** -- Reads all leads from your sheet.
3. **Filter (days since last email)** -- Checks `shouldSendEmail` flag and that at least 2 days have passed since the last email. Prevents over-emailing.

### Stage Routing
4. **Switch** -- Routes leads based on their `Stage` column value:
   - Stage 1 -> First cold email path
   - Stage 2 -> Follow-up email path
   - Other -> No Operation (skip)

### Stage 1: First Cold Email
5. **AI Agent (Stage 1)** -- Generates the initial cold email:
   - Queries Pinecone for relevant company context
   - Uses lead data (name, company, title) for personalization
   - Writes a short, punchy email with a clear CTA
6. **AI Agent (Subject Line)** -- Generates a compelling subject line separately.
7. **Gmail (send)** -- Sends the email to the lead's email address.
8. **Google Sheets (update)** -- Updates `Stage` to 2, sets `Last Email Date` to today.

### Stage 2: Follow-up Email
9. **AI Agent (Stage 2)** -- Generates a follow-up email with a different angle:
   - References the first email ("Just wanted to follow up...")
   - Provides additional value (case study, insight, or question)
   - Shorter than Stage 1
10. **AI Agent (Subject Line 2)** -- Generates follow-up subject line (often "Re:" style).
11. **Gmail (send)** -- Sends the follow-up.
12. **Google Sheets (update)** -- Updates stage and timestamp.

### Pinecone Knowledge (Shared)
13. **Pinecone Vector Store (query)** -- Both stages query Pinecone for context about your product/service that is relevant to the lead's industry or role. This is what makes the emails feel personalized beyond just "Hi {First Name}."

---

## Google Sheet Structure

| Column | Description |
|--------|-------------|
| First Name | Lead first name |
| Last Name | Lead last name |
| Email | Lead email address |
| Company | Company name |
| Title | Job title |
| Stage | Current stage: `1` (not contacted), `2` (first email sent) |
| Last Email Date | Date of last email sent (YYYY-MM-DD) |
| shouldSendEmail | `TRUE` or `FALSE` -- master switch per lead |
| daysSinceLastEmail | Formula: `=IF(G2="", 999, TODAY()-DATEVALUE(G2))` |
| Reply Received | `yes` or `no` |

**Important:** The `daysSinceLastEmail` column should be a formula in Google Sheets. The workflow reads this value to decide whether enough time has passed.

---

## AI Prompt Customization

The AI agents use system prompts to write emails. You should customize these for your product/service. Key things to update:

### Stage 1 System Prompt (in the AI Agent node)
- What your product/service does (1-2 sentences)
- Your target customer's typical pain points
- Your unique value proposition
- Desired CTA (book a call, reply, visit a page)

### Stage 2 System Prompt
- Reference that a first email was sent
- Provide a different angle (social proof, case study, feature highlight)
- Softer CTA (question-based)

---

## Testing Steps

1. **Add 2-3 test leads** to your Google Sheet (use your own email addresses)
2. **Set Stage to 1** and `shouldSendEmail` to `TRUE`
3. **Run manually:** Execute the workflow
4. **Check your inbox:** Verify the email arrived and looks good
5. **Wait 2+ days** (or temporarily reduce the day gap in the Filter node)
6. **Change Stage to 2** and run again to test the follow-up email
7. **Review AI quality:** Read the emails. If they sound too generic, add more context to the Pinecone knowledgebase.

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| No emails sent | All leads filtered out by day gap or `shouldSendEmail` | Check sheet values; ensure `shouldSendEmail` = TRUE and enough days passed |
| Gmail sending limit | Gmail has daily send limits (500 for regular, 2000 for Workspace) | Stagger sends; consider Google Workspace for higher limits |
| Emails land in spam | New sending domain or too many sends | Warm up your email domain first; use an email warmup service |
| AI writes too-long emails | System prompt not constraining length | Add "Keep the email under 100 words" to the system prompt |
| Pinecone returns nothing | Index empty or wrong index name | Run Knowledgebase workflow (#9) first; verify index name |
| Stage not updating | Google Sheets write permission issue | Re-authenticate Sheets credential; check sheet permissions |
