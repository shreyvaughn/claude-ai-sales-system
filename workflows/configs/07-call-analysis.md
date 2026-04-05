# Workflow 7: Advanced Call Analysis

## Overview

This workflow receives call data from RetellAI (or any call system that sends webhooks), analyzes the transcript using AI, and classifies the outcome into one of four categories: BOOKED (appointment set), FUTURE (interested but wants a later callback), REJECTED (declined the offer), or UNKNOWN (unclear outcome). Based on the classification, it routes the data to the appropriate Google Sheets tab and triggers downstream actions.

---

## Required Services

- [ ] **RetellAI** -- https://www.retellai.com/ (or any call system with webhook support)
- [ ] **Google Sheets** -- For call results tracking
- [ ] **OpenAI** -- https://platform.openai.com/ (transcript analysis)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |
| Google Sheets OAuth | n8n Credentials > Google Sheets > OAuth2 | Google Sheets OAuth2 |
| RetellAI Webhook | RetellAI Dashboard > Agents > Webhook URL | (set n8n webhook URL in RetellAI) |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_OPENAI_API_KEY` | OpenAI key | Classification AI node |
| `YOUR_GOOGLE_SHEETS_ID` | Call results spreadsheet ID | Google Sheets nodes |
| `YOUR_RETELL_WEBHOOK_PATH` | n8n webhook path | RetellAI webhook configuration |

---

## Node-by-Node Configuration

### Trigger
1. **Webhook** -- Receives POST requests from RetellAI after each call ends.
   - RetellAI sends call data including: transcript, summary, recording URL, call duration, metadata.
   - After importing, copy the **Production Webhook URL** and paste it into your RetellAI agent's webhook settings.

### Data Extraction
2. **Edit Fields** -- Extracts key data from the webhook payload:
   - `call_analysis.custom_analysis_data.summary` -- Call summary
   - `to_number` -- Prospect's phone number
   - `recording_url` -- Link to call recording
   - `retell_llm_dynamic_variables.company_name` -- Company name
   - `metadata.email_in_list` -- Prospect's email
   - Metrics fields for daily tracking
3. **Format Date** -- Formats the call date for consistent sheet entry.

### AI Classification
4. **OpenAI (find interest)** -- Sends the call summary to GPT with this prompt:
   > Given above is a summary of a cold call. Analyze and find out if the prospect has:
   > 1. Booked an appointment (MENTIONED A DATE AND TIME)
   > 2. Interested but wants to be called in the future
   > 3. None of those 2 situations
   > 4. Ended the call or rejected the offer
   >
   > Respond with ONLY one of: "BOOKED", "FUTURE", "UNKNOWN", "REJECTED"

### Outcome Routing
5. **Filter (booked)** -- Routes to "booked" sheet tab if classification = BOOKED.
6. **Filter (rejected)** -- Routes to "rejected" sheet tab if classification = REJECTED.
7. **Filter (future)** -- Routes to "future" sheet tab if classification = FUTURE.
8. **Filter (unknown)** -- Routes to "unknown" sheet tab if classification = UNKNOWN.

### Google Sheets (per outcome)
9. **Google Sheets (booked)** -- Appends to "booked" tab:
   - Name, phone, email, summary, recording URL, booked date/time, call date
10. **Google Sheets (future)** -- Appends to "future" tab:
    - Name, phone, email, summary, recording URL, callback preference, call date
11. **Google Sheets (rejected)** -- Appends to "rejected" tab:
    - Name, phone, email, summary, recording URL, rejection reason, call date
12. **Google Sheets (unknown)** -- Appends to "unknown" tab:
    - Name, phone, email, summary, recording URL, call date

### Metrics Update
13. **Google Sheets (metrics)** -- Updates the daily metrics row:
    - Increments the appropriate counter (calls sent, human picked up, booked, rejected, voicemail)
    - Updates cost incurred

---

## Google Sheet Structure

Create a spreadsheet with 5 tabs:

### Tab: "booked"
| Column | Description |
|--------|-------------|
| name | Company/lead name |
| phone_number | Prospect phone |
| summary | Call summary |
| recording url | Link to recording |
| their email | Prospect email |
| what time and date | Booked appointment time |
| when was the call sent | Date of this call |

### Tab: "future"
Same columns as "booked" but "what time and date" contains "Future" marker.

### Tab: "rejected"
Same columns as "booked" but "what time and date" contains rejection reason.

### Tab: "unknown"
Same columns as "booked."

### Tab: "metrics" (daily tracking)
| Column | Description |
|--------|-------------|
| date | YYYY-MM-DD |
| calls sent | Total calls that day |
| human calls picked | Calls answered by a human |
| voicemails | Calls that hit voicemail |
| appointments booked | Successful bookings |
| offer rejected | Hard rejections |
| cost incurred | RetellAI cost |

---

## RetellAI Webhook Setup

1. Log in to RetellAI dashboard
2. Go to your Agent settings
3. Find the **Post-Call Webhook** or **Webhook URL** field
4. Paste your n8n webhook Production URL
5. Ensure the webhook sends: transcript, summary, recording URL, and metadata

---

## Testing Steps

1. **Make a test call** through your RetellAI agent (call yourself)
2. **Check the webhook node:** Verify the payload arrived with all expected fields
3. **Check the classification node:** Click it to see if GPT returned BOOKED/FUTURE/REJECTED/UNKNOWN
4. **Check Google Sheets:** Verify the data landed in the correct tab
5. **Check metrics:** Verify the counters incremented
6. **Test all 4 outcomes:** Make calls that simulate each scenario

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Webhook never fires | RetellAI webhook URL wrong | Use the Production URL, not the Test URL |
| Classification wrong | Call summary too vague | Improve your RetellAI agent's prompt to produce clearer summaries |
| Missing email in sheet | Email not in RetellAI metadata | Ensure you pass email in the metadata when making calls (from the SDR workflow) |
| Metrics sheet errors | Row not found for today's date | Pre-populate the metrics sheet with date rows, or add a "create row if missing" step |
| Duplicate entries | Webhook fires multiple times | Add dedup check on call ID before writing to sheets |
