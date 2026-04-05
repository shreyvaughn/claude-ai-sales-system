# Workflow 11: AI Presentation Creator

## Overview

This workflow converts sales call transcripts into polished Google Slides presentations. It receives call data via webhook (transcript and summary), uses an AI model to generate structured presentation content, duplicates a Google Slides template in your Drive, and replaces placeholder text in the template with the AI-generated content. The result is a ready-to-share presentation tailored to the specific call -- useful for sending recaps to prospects or briefing your team.

---

## Required Services

- [ ] **Google Slides** -- Google account with Slides API access
- [ ] **Google Drive** -- Google account with Drive access (for template duplication)
- [ ] **OpenAI** -- https://platform.openai.com/ (content generation)
- [ ] **Google Service Account** -- For programmatic access to Slides and Drive

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Google Service Account | Google Cloud Console > IAM > Service Accounts | Google Service Account |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_OPENAI_API_KEY` | OpenAI key for content generation | OpenAI Chat Model node |
| `YOUR_GOOGLE_SERVICE_ACCOUNT_CREDENTIAL` | Service account credential name | Google Drive + Google Slides nodes |
| `YOUR_TEMPLATE_PRESENTATION_ID` | ID of your Google Slides template | Duplicate Template node |
| `YOUR_WEBHOOK_PATH` | Webhook path for receiving call data | Webhook trigger node (default: `presentation-builder`) |

---

## Google Service Account Setup

A service account lets n8n access Google Slides and Drive programmatically without OAuth pop-ups. This is required because the Google Slides and Drive nodes in this workflow use service account authentication.

### 1. Create a Service Account

1. Go to https://console.cloud.google.com/
2. Select (or create) a project
3. Navigate to **IAM & Admin** > **Service Accounts**
4. Click **Create Service Account**
5. Give it a name (e.g., "n8n-automation")
6. Click **Create and Continue**
7. Skip the optional role assignment (or add "Editor" for broad access)
8. Click **Done**

### 2. Generate a Key

1. Click on the service account you just created
2. Go to the **Keys** tab
3. Click **Add Key** > **Create New Key** > **JSON**
4. Download the JSON key file -- you will paste its contents into n8n

### 3. Enable Required APIs

In the Google Cloud Console, go to **APIs & Services** > **Library** and enable:
- Google Slides API
- Google Drive API

### 4. Share Your Template with the Service Account

The service account has its own email address (looks like `name@project.iam.gserviceaccount.com`). You need to share your template presentation with this email:

1. Open your Google Slides template
2. Click **Share**
3. Paste the service account email address
4. Give it **Editor** access
5. Click **Send**

### 5. Create the n8n Credential

1. In n8n, go to **Credentials** > **Add Credential** > **Google Service Account**
2. Paste the contents of the JSON key file
3. Save

---

## Google Slides Template Setup

The workflow duplicates a template presentation and replaces placeholder text with AI-generated content.

### 1. Create Your Template

1. Create a new Google Slides presentation
2. Design your slides with your branding (logo, colors, fonts)
3. Add placeholder text where dynamic content should go. Use a distinctive marker, such as:
   ```
   {{CONTENT}}
   ```
4. The AI-generated text will replace this placeholder

### 2. Get the Template ID

The Presentation ID is in the URL:
```
https://docs.google.com/presentation/d/YOUR_TEMPLATE_PRESENTATION_ID/edit
```

### 3. Template Design Tips

- Keep the template simple -- 1-3 slides works well for call recaps
- Use a single large text box with the placeholder on each content slide
- The AI generates the full text block, so you do not need multiple placeholders per slide
- Title slide can be static (your company name/logo) or include a `{{TITLE}}` placeholder

---

## Node-by-Node Configuration

### Trigger
1. **Webhook** ("Presentation Builder") -- Receives POST requests with call data.
   - Default path: `presentation-builder`
   - After importing, copy the **Production Webhook URL** to use as the trigger endpoint.

### Content Generation
2. **Generate Text** (Basic LLM Chain) -- Sends the transcript and summary to the AI model with a prompt to generate structured presentation content.
   - **Customize the prompt** to match your presentation style:
     - What sections to include (overview, key points, next steps, etc.)
     - Tone and length preferences
     - Any standard sections your presentations always have
   - Uses OpenAI Chat Model (configurable model -- GPT-4o recommended for quality)

### Template Duplication
3. **Duplicate Template** (Google Drive) -- Copies your template presentation to create a new file.
   - Authentication: Service Account
   - Operation: Copy
   - File ID: `YOUR_TEMPLATE_PRESENTATION_ID`
   - The new file name defaults to "File Name" -- customize this in the node (e.g., use the prospect's company name from the webhook payload)

### Text Replacement
4. **Google Slides - Replace Text** -- Replaces the placeholder text in the duplicated presentation with the AI-generated content.
   - Authentication: Service Account
   - Presentation ID: Automatically uses the ID from the duplicated file
   - Find: Your placeholder text (e.g., `{{CONTENT}}`)
   - Replace with: The AI-generated text from step 2

---

## Webhook Payload Format

The webhook expects this JSON structure:

```json
{
  "call transcript": "Full transcript of the sales call...",
  "call summary": "Brief summary of the call and key outcomes..."
}
```

You can trigger this webhook from:
- The Call Analysis workflow (#7) after a call is classified
- The Meeting Notes workflow (#12) as an additional output
- A manual cURL/Postman request
- Your CRM via automation rules

---

## Testing Steps

1. **Create a Google Slides template** with a `{{CONTENT}}` placeholder
2. **Share the template** with your service account email (Editor access)
3. **Import the workflow** and set the Template Presentation ID
4. **Send a test webhook:**
   ```bash
   curl -X POST YOUR_N8N_WEBHOOK_URL \
     -H "Content-Type: application/json" \
     -d '{
       "call transcript": "Sales rep: Hi, thanks for taking the time today. Prospect: Of course, we have been looking at solutions like yours. We struggle with lead generation and our current process is very manual...",
       "call summary": "Prospect is interested in automating their lead generation. They currently spend 10 hours per week on manual outreach. Next step: product demo scheduled for next Tuesday."
     }'
   ```
5. **Check the Generate Text node:** Verify the AI produced structured content
6. **Check Google Drive:** A new presentation should appear (copy of your template)
7. **Open the new presentation:** Verify the placeholder was replaced with AI content

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| "File not found" on template | Service account does not have access | Share the template with the service account email as Editor |
| Google Slides API error | API not enabled in Google Cloud | Enable Google Slides API and Google Drive API in the Cloud Console |
| Placeholder not replaced | Placeholder text in node does not match template exactly | Copy the exact placeholder text from your template (including any special characters) |
| AI content too long for slides | Generated text exceeds slide boundaries | Add length constraints to the AI prompt (e.g., "Keep each section under 50 words") |
| New file appears in service account's Drive | Default behavior for service account copies | Add a "Move File" step to transfer the file to a shared folder, or share the service account's root folder with yourself |
| Webhook never fires | Using Test URL instead of Production URL | Copy the Production URL (not the Test URL) from the webhook node |
