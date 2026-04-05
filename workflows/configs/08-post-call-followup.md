# Workflow 8: Post-Call Follow Up

## Overview

This workflow automatically generates and sends personalized follow-up email sequences after a sales call. It receives call data via webhook (transcript, summary, lead info, next meeting date), uses AI to craft 1-3 follow-up emails (case study, product demo, feature offer, etc.), then spaces them out evenly between now and the next scheduled meeting. Each email is timed to arrive during business hours and is sent via SMTP.

---

## Required Services

- [ ] **SMTP Email** -- Any SMTP-capable email provider (Gmail SMTP, SendGrid, Mailgun, etc.)
- [ ] **OpenAI** -- https://platform.openai.com/ (email generation)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |
| SMTP Credentials | Your email provider's SMTP settings | SMTP |

### SMTP Settings (Common Providers)

| Provider | Host | Port | Encryption |
|---------|------|------|-----------|
| Gmail | smtp.gmail.com | 587 | STARTTLS |
| Google Workspace | smtp.gmail.com | 587 | STARTTLS |
| SendGrid | smtp.sendgrid.net | 587 | STARTTLS |
| Mailgun | smtp.mailgun.org | 587 | STARTTLS |
| Amazon SES | email-smtp.{region}.amazonaws.com | 587 | STARTTLS |

For Gmail: You need an **App Password** (not your regular password). Go to Google Account > Security > App Passwords.

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_OPENAI_API_KEY` | OpenAI key | LLM Chain for email generation |
| `YOUR_SMTP_HOST` | SMTP server hostname | SMTP credential |
| `YOUR_SMTP_PORT` | SMTP port (usually 587) | SMTP credential |
| `YOUR_SMTP_USER` | SMTP username/email | SMTP credential |
| `YOUR_SMTP_PASSWORD` | SMTP password or app password | SMTP credential |
| `YOUR_SENDER_EMAIL` | "From" email address | Email Send nodes |
| `YOUR_NAME` | Your name (sales closer) | Email signatures |
| `YOUR_COMPANY_NAME` | Company name | Email context |

---

## Node-by-Node Configuration

### Trigger
1. **Webhook** -- Receives POST data containing:
   - Lead name, email, company
   - Call transcript (full text)
   - Call summary
   - Next appointment date
   - Selected email types to generate (array)
   - Sales closer name

### Email Generation
2. **Edit Fields** -- Extracts and formats the webhook payload:
   - Lead name, email
   - Follow-up date
   - Selected email types array
   - Transcript + summary
3. **Basic LLM Chain** -- Sends all data to GPT with a detailed prompt that:
   - Reads the full transcript for context
   - Generates one email per selected type (Case Study, Call Recording, Product Demo, Feature Offer, Lead Database, etc.)
   - Outputs JSON array with `emailType`, `subject`, and `body` for each
   - Signs each email as the closer
4. **Code (parse emails)** -- Parses the LLM output JSON into structured `Email1`, `Email2`, `Email3` objects.

### Scheduling Logic
5. **Code (calculate schedule)** -- Determines when to send each email:
   - Calculates days until the next meeting
   - Distributes emails evenly across that time period
   - Assigns random times within business hours (9 AM - 6 PM EST)
   - Outputs wait durations in seconds

### Sending Routes
6. **Switch (number of emails)** -- Routes based on how many emails were generated:
   - 1 email: send immediately
   - 2 emails: send first now, schedule second
   - 3 emails: space all three out evenly
7. **Wait (per email)** -- Waits the calculated duration before each email.
8. **Send Email (per email)** -- Sends via SMTP:
   - From: `YOUR_SENDER_EMAIL`
   - To: Lead's email
   - Subject: from AI-generated content
   - Body: from AI-generated content (HTML format)

---

## Webhook Payload Format

The webhook expects this JSON structure:

```json
{
  "data": {
    "properties": {
      "Name": { "title": [{ "plain_text": "John Smith" }] },
      "Email Address": { "email": "john@example.com" },
      "Company": { "rich_text": [{ "plain_text": "Acme Corp" }] }
    }
  },
  "transcript": "Full call transcript text here...",
  "summary": "Brief summary of the call...",
  "next_meeting_date": "2025-07-15",
  "selected_email_types": ["Case Study", "Product Demo", "Feature Offer"],
  "closer_name": "YOUR_NAME"
}
```

You can trigger this webhook from:
- The Call Analysis workflow (#7) when a call is classified as "booked" or "future"
- Your CRM (HubSpot, etc.) via automation rules
- A manual Postman/cURL request

---

## Email Types Supported

| Type | When to Use | What AI Generates |
|------|-------------|------------------|
| Case Study | Prospect mentioned a pain point you have solved before | Success story referencing their specific concern |
| Call Recording | Prospect wants to share internally | Link to call recording with key highlights |
| Product Demo | Prospect needs to show team | Walkthrough demo link with features relevant to them |
| Feature Offer | Prospect mentioned a specific need | Highlight of a feature that directly solves it |
| Lead Database | Prospect is interested in lead generation | Sample data or trial access offer |

---

## Testing Steps

1. **Send a test webhook** using cURL or Postman:
   ```bash
   curl -X POST YOUR_N8N_WEBHOOK_URL \
     -H "Content-Type: application/json" \
     -d '{
       "data": {"properties": {"Name": {"title": [{"plain_text": "Test User"}]}, "Email Address": {"email": "your-test@email.com"}}},
       "transcript": "This is a test transcript. The prospect mentioned they need help with lead generation.",
       "summary": "Prospect is interested but wants to see a demo first.",
       "next_meeting_date": "2025-12-01",
       "selected_email_types": ["Case Study", "Product Demo"],
       "closer_name": "Your Name"
     }'
   ```
2. **Check the LLM Chain node:** Verify it generated 2 emails
3. **Check the Code nodes:** Verify parsing and schedule calculation
4. **Check your inbox:** Verify the first email arrived
5. **For timed emails:** Temporarily set Wait to 10 seconds for testing

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| SMTP auth failure | Wrong credentials or missing App Password | For Gmail, generate an App Password; verify SMTP settings |
| Emails land in spam | No SPF/DKIM records | Set up SPF and DKIM on your domain; use a dedicated sending domain |
| LLM returns malformed JSON | Token limit or parsing issue | Add explicit JSON formatting instructions to the prompt; use GPT-4o |
| Schedule calculation wrong | Timezone mismatch | Set the timezone offset in the Code node (default is EST/EDT) |
| No emails generated | Selected email types array empty | Ensure the webhook payload includes `selected_email_types` |
| Wait node takes too long | Testing with a far-future meeting date | Use a near-future date or reduce Wait times for testing |
