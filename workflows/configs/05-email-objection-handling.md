# Workflow 5: Email Objection Handling

## Overview

This workflow monitors your Gmail inbox for replies to outreach emails, then uses AI to analyze the reply, determine if it is a positive response, objection, or rejection, and automatically drafts and sends an appropriate follow-up. It uses your Pinecone knowledgebase to pull relevant information when handling specific objections (pricing, competitors, features). Think of it as an AI SDR that handles the back-and-forth email conversation for you.

---

## Required Services

- [ ] **Gmail** -- Google account with Gmail API access
- [ ] **Google Sheets** -- To cross-reference which replies are from outreach leads
- [ ] **OpenAI** -- https://platform.openai.com/ (reply analysis and response generation)
- [ ] **Pinecone** -- https://www.pinecone.io/ (knowledge for handling objections)

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
| `YOUR_GMAIL_CREDENTIAL` | Gmail OAuth credential name | Gmail Trigger and Gmail send nodes |
| `YOUR_GOOGLE_SHEETS_ID` | Lead tracking sheet ID | Google Sheets lookup node |
| `YOUR_OPENAI_API_KEY` | OpenAI key | AI classification and response agents |
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store nodes |
| `YOUR_PINECONE_INDEX_NAME` | Knowledge index name | Pinecone node config |
| `YOUR_COMPANY_NAME` | Company name for context | AI agent prompts |
| `YOUR_NAME` | Sender name for replies | Email signature |

---

## Node-by-Node Configuration

### Trigger & Validation
1. **Gmail Trigger** -- Polls for new emails every hour. Triggers on any new incoming email.
2. **Gmail (get thread)** -- Fetches the full email thread to understand context.
3. **Google Sheets (lookup)** -- Checks if the sender's email exists in your lead database. If not, the email is ignored (it is not from an outreach lead).
4. **If (valid lead)** -- Routes: known lead -> process; unknown sender -> skip.

### Classification
5. **Switch (stage routing)** -- Routes based on the lead's current stage in Google Sheets:
   - Stage 1 reply -> first objection handling path
   - Stage 2 reply -> follow-up objection handling path
   - Other -> skip

### Objection Analysis
6. **AI Agent (classify reply)** -- Reads the full email thread and classifies the reply:
   - **Positive** -- They want to learn more or book a call
   - **Objection** -- They have concerns (price, timing, competition, relevance)
   - **Rejection** -- They are not interested, asked to be removed
   - **Out of Office** -- Auto-reply, not a real response

### Response Generation
7. **Pinecone Vector Store (query)** -- Pulls relevant knowledge to address the specific objection.
8. **AI Agent (write response)** -- Generates a reply that:
   - Acknowledges their concern
   - Provides relevant information from Pinecone
   - Includes a soft next step
   - Matches the tone of the conversation
9. **Wait** -- Pauses 30-60 minutes before sending (makes it look human).
10. **Gmail (reply)** -- Sends the response as a thread reply (not a new email).
11. **Google Sheets (update)** -- Updates the lead's stage and notes.

### Rejection Handling
12. **If (rejected)** -- If the reply is a rejection, the workflow:
    - Updates Google Sheets to mark lead as "rejected"
    - Sets `shouldSendEmail` to FALSE
    - Does NOT send a reply (respects their wish)

---

## AI Prompt Customization

### Classification Prompt
The classification agent needs to understand your specific product context. Update its system prompt to include:
- Common objections you hear (price too high, not the right time, using a competitor, etc.)
- What constitutes a "positive" reply in your context

### Response Prompt
The response agent should know:
- Your pricing structure (at a high level) for price objections
- Your key differentiators vs. competitors
- Case studies or proof points to reference
- Your preferred meeting booking link

---

## Testing Steps

1. **Send yourself a test outreach email** using the Email Sequencing workflow (#4)
2. **Reply to it** from a different email address with an objection (e.g., "Sounds interesting but we already use [Competitor]")
3. **Wait for the trigger** to fire (up to 1 hour, or manually execute)
4. **Check the classification node:** Verify it correctly identified the reply type
5. **Check the response node:** Review the AI-generated reply for accuracy and tone
6. **Check your inbox:** Verify the reply was sent as a thread response
7. **Test rejection:** Reply with "Not interested, please remove me" and verify no response is sent

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Workflow never triggers | Gmail Trigger not activated | Ensure the workflow is Active (toggle in top right) |
| Processes non-outreach emails | Google Sheets lookup not filtering correctly | Verify the lookup matches on the sender's email column |
| AI response too aggressive | System prompt not calibrated | Add "Never be pushy. If someone is not interested, respect that." to the prompt |
| Reply sent as new email (not thread) | Wrong Gmail node configuration | Use "Reply" operation on Gmail node, not "Send"; pass the correct threadId and messageId |
| Handles personal emails | No filter on which emails to process | Add a label filter or only process emails from addresses in your lead sheet |
| Too fast response time | No wait node | Add a Wait node (30-60 min) before the reply to appear human |
