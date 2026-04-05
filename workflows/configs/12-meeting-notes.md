# Workflow 12: Meeting Notes Agent

## Overview

This workflow automatically generates structured meeting notes from sales call transcripts, stores them in multiple places, and notifies your team. When triggered via webhook with a call transcript and summary, it uses AI to produce detailed, organized notes, creates a Google Doc with the content, indexes the notes in Pinecone (so other AI workflows can reference past conversations), and sends a Slack notification with a link to the document. It is designed to run after every sales call, giving your team instant access to organized notes without manual effort.

---

## Required Services

- [ ] **Google Docs** -- Google account with Docs API access
- [ ] **Google Drive** -- For document storage (same Google account)
- [ ] **Pinecone** -- https://www.pinecone.io/ (indexing notes for AI context)
- [ ] **OpenAI** -- https://platform.openai.com/ (note generation + embeddings)
- [ ] **Slack** -- https://slack.com/ (team notifications)
- [ ] **Google Service Account** -- For programmatic access to Docs and Drive

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Google Service Account | Google Cloud Console > IAM > Service Accounts | Google Service Account |
| Pinecone API Key | Pinecone Console > API Keys | Pinecone API |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |
| Slack OAuth | Slack App > OAuth & Permissions | Slack OAuth2 |

### Google APIs to Enable

In the Google Cloud Console, go to **APIs & Services** > **Library** and enable:
- Google Docs API
- Google Drive API

If you already set up a service account for the Presentation Creator workflow (#11), you can reuse the same one here.

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_GOOGLE_SERVICE_ACCOUNT_CREDENTIAL` | Google service account credential | Google Docs create + update nodes |
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store node |
| `YOUR_PINECONE_INDEX_NAME` | Index name for meeting notes | Pinecone node config |
| `YOUR_OPENAI_API_KEY` | OpenAI key for text generation + embeddings | OpenAI Chat Model + Embeddings nodes |
| `YOUR_SLACK_CREDENTIAL` | Slack OAuth credential name | Slack notification node |
| `YOUR_SLACK_CHANNEL_ID` | Channel for meeting note alerts | Slack node config |
| `YOUR_WEBHOOK_PATH` | Webhook path for receiving call data | Webhook trigger (default: `propsect-notes-to-closer`) |

### Finding Your Slack Channel ID

1. Open Slack in a browser
2. Navigate to the target channel
3. The URL will look like: `https://app.slack.com/client/TXXXXXX/CXXXXXXXX`
4. The `CXXXXXXXX` part is the Channel ID

Alternatively: right-click the channel name > **View channel details** > scroll to the bottom to find the ID.

---

## Node-by-Node Configuration

### Trigger
1. **Webhook** ("Prospect Notes Maker") -- Receives POST requests with call data.
   - Default path: `propsect-notes-to-closer`
   - After importing, copy the **Production Webhook URL** for use as the trigger endpoint.

### Note Generation
2. **Basic LLM Chain** ("generate text1") -- Sends the transcript and summary to the AI model with a detailed prompt. The AI generates structured meeting notes including:
   - Attendees and their roles
   - Key topics discussed
   - Pain points identified
   - Objections raised and how they were addressed
   - Next steps and action items
   - Follow-up date (if mentioned)
   - Overall sentiment and likelihood of close
   - Uses OpenAI Chat Model (the workflow defaults to a GPT model -- configurable in the model node)

### Google Doc Creation
3. **Google Docs** ("create doc") -- Creates a new Google Doc:
   - Authentication: Service Account
   - Title: Set dynamically (e.g., "Meeting Notes - {Company} - {Date}") or update the default "Doc Name" placeholder
   - Folder: Default is root Drive folder; optionally set a specific folder ID
4. **Google Docs** ("update doc text") -- Inserts the AI-generated notes into the document body.
   - Uses the document ID from the create step
   - Inserts the full text output from the LLM Chain

### Pinecone Indexing
5. **Embeddings OpenAI** -- Generates vector embeddings for the meeting notes text.
6. **Recursive Character Text Splitter** -- Splits longer notes into chunks for embedding.
7. **Default Data Loader** -- Prepares the text for the vector store.
8. **Pinecone Vector Store** -- Stores the embedded notes in your Pinecone index.
   - Mode: Insert
   - Index: `YOUR_PINECONE_INDEX_NAME`
   - This makes the notes available to other workflows (SDR, Email Sequencing, Objection Handling) for context when interacting with the same prospect in the future.

### Slack Notification
9. **Slack** -- Posts a message to your team channel with a link to the Google Doc.
   - Authentication: OAuth2
   - Channel: `YOUR_SLACK_CHANNEL_ID`
   - Message format: "Here you go with the prospect's doc: https://docs.google.com/document/d/{DOC_ID}"
   - Make sure the Slack bot is invited to the target channel

---

## Webhook Payload Format

The webhook expects this JSON structure:

```json
{
  "call transcript": "Full transcript of the sales call...",
  "call summary": "Brief summary of the call and key outcomes."
}
```

Optional additional fields you can include:
```json
{
  "call transcript": "...",
  "call summary": "...",
  "prospect_name": "John Smith",
  "company_name": "Acme Corp",
  "call_date": "2025-06-15"
}
```

This webhook can be triggered by:
- The Call Analysis workflow (#7) after classifying a call
- RetellAI's post-call webhook directly
- The Post-Call Follow Up workflow (#8) as a parallel action
- A manual POST request via cURL or Postman

---

## Google Doc Sharing

By default, documents created by the service account live in the service account's own Drive and are not visible to you. To access them:

**Option A (recommended):** Share a Google Drive folder with the service account, then configure the "create doc" node to use that folder. Documents created there will be visible to anyone with folder access.

**Option B:** After creating the doc, add a Google Drive "Share" node to grant your email address access. This requires an extra node but automates sharing.

**Option C:** The Slack notification includes the document link. If you have the document ID, you can access it directly -- but you may need to request access from the service account owner.

---

## Testing Steps

1. **Set up the Slack bot** and invite it to your target channel
2. **Configure Pinecone** with an index (1536 dimensions, cosine metric)
3. **Import the workflow** and set all credentials
4. **Send a test webhook:**
   ```bash
   curl -X POST YOUR_N8N_WEBHOOK_URL \
     -H "Content-Type: application/json" \
     -d '{
       "call transcript": "Hi John, thanks for taking the time to chat. We discussed your current lead gen process and how our platform could help automate it. You mentioned spending about 4 hours a day on manual prospecting...",
       "call summary": "Discussed lead gen automation. Prospect spending 4h/day on manual prospecting. Interested in seeing a demo next Tuesday."
     }'
   ```
5. **Check the LLM node:** Verify structured notes were generated
6. **Check Google Docs:** Open the created document and review the notes
7. **Check Pinecone Console:** Verify new vectors were inserted in your index
8. **Check Slack:** Verify the notification appeared with the document link

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Google Docs "permission denied" | Service account not set up or APIs not enabled | Check service account credential in n8n; enable Docs API and Drive API in Cloud Console |
| Slack message fails | Bot not in channel or wrong channel ID | Invite the bot to the channel with `/invite @botname`; verify the Channel ID |
| Pinecone insert fails | Wrong index name or dimension mismatch | Verify index exists in Pinecone Console and uses 1536 dimensions |
| Notes are too generic | Transcript or summary too short or vague | Ensure the call system provides detailed transcripts; add more specifics to the summary |
| Doc link returns 404 | Document lives in service account's private Drive | Set up folder sharing (see Google Doc Sharing section above) |
| Slack "not_in_channel" error | Bot not added to the target channel | Type `/invite @your-bot-name` in the Slack channel |
| Embeddings fail | OpenAI API key invalid or rate limited | Verify your API key; check usage limits on the OpenAI dashboard |
