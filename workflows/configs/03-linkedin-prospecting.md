# Workflow 3: LinkedIn Prospecting Flow

## Overview

This workflow automates LinkedIn DM outreach at scale. It reads leads from a Google Sheet, researches each one (company website, LinkedIn profile, recent posts), generates a hyper-personalized first message using AI, creates a voice note via ElevenLabs, and sends both the message and voice note through LinkedIn via the Unipile API. It includes daily send limits and tracks every interaction.

The workflow also includes an Apollo-based lead finding form that populates the Google Sheet, making it a self-contained prospecting system.

---

## Required Services

- [ ] **Unipile** -- https://www.unipile.com/ (LinkedIn messaging API, ~$49/mo)
- [ ] **Apollo.io** -- https://www.apollo.io/ (lead search and enrichment)
- [ ] **ElevenLabs** -- https://elevenlabs.io/ (AI voice generation)
- [ ] **Apify** -- https://apify.com/ (LinkedIn profile scraping, website scraping, Google search)
- [ ] **Pinecone** -- https://www.pinecone.io/ (vector DB for company context)
- [ ] **Google Sheets** -- Lead tracking spreadsheet
- [ ] **OpenAI** -- https://platform.openai.com/ (message generation)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Unipile API Key | Unipile Dashboard > API Keys | HTTP Header Auth |
| Unipile DSN | Unipile Dashboard > Connection Info | (used in URL) |
| Unipile LinkedIn Account ID | Unipile Dashboard > Accounts | (used in API calls) |
| Apollo API Key | Apollo Settings > API | HTTP Header Auth |
| ElevenLabs API Key | elevenlabs.io > Profile > API Key | HTTP Header Auth |
| ElevenLabs Voice ID | elevenlabs.io > Voices > Select Voice > ID | (used in URL) |
| Apify API Token | Apify Console > Settings > Integrations | (used in URL query param) |
| Pinecone API Key | Pinecone Console > API Keys | Pinecone API |
| Google Sheets OAuth | n8n Credentials > Google Sheets | Google Sheets OAuth2 |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_UNIPILE_API_KEY` | Unipile access token | Authorization headers |
| `YOUR_UNIPILE_DSN` | Unipile API endpoint (e.g., `api8.unipile.com:13814`) | URL base in HTTP Request nodes |
| `YOUR_UNIPILE_LINKEDIN_ACCOUNT_ID` | Your LinkedIn account ID in Unipile | API call parameters |
| `YOUR_APOLLO_API_KEY` | Apollo API key | Apollo form and search nodes |
| `YOUR_ELEVENLABS_API_KEY` | ElevenLabs API key | `xi-api-key` header |
| `YOUR_ELEVENLABS_VOICE_ID` | Voice to use for voice notes | URL path in ElevenLabs API call |
| `YOUR_APIFY_API_KEY` | Apify token | `?token=` query parameter |
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store nodes |
| `YOUR_PINECONE_INDEX_NAME` | Index for company knowledge | Pinecone node config |
| `YOUR_GOOGLE_SHEETS_ID` | Lead tracking sheet ID | Google Sheets nodes |
| `YOUR_OPENAI_API_KEY` | OpenAI key | AI message generation |
| `YOUR_COMPANY_NAME` | Your company name | Message templates |
| `YOUR_NAME` | Your name | Message signatures |

---

## Node-by-Node Configuration

### Lead Acquisition (Apollo Form)
1. **n8n Form Trigger** -- Presents a form to input Apollo search criteria (job titles, industries, company size, location).
2. **Apollo API Request** -- Fetches up to 100 leads matching the form criteria.
3. **Structure Apollo Data** -- Formats lead data into clean rows.
4. **Google Sheets (append)** -- Adds leads to your tracking sheet.

### Daily Outreach Loop
5. **Schedule Trigger** -- Runs on your preferred schedule.
6. **Google Sheets (read)** -- Reads leads where `1st message sent` = `no`.
7. **Daily Count Check** -- Checks how many messages were sent today (enforces daily limit).
8. **Filter** -- Stops execution if daily limit reached.

### Research Pipeline (per lead)
9. **Unipile Get Profile** -- Fetches LinkedIn profile data via Unipile.
10. **Apify LinkedIn Profile Scraper** -- Deep scrape of LinkedIn profile (experience, posts, about section).
11. **Apify LinkedIn Posts Scraper** -- Scrapes recent posts for personalization.
12. **Apify Website Scraper** -- Scrapes the lead's company website.
13. **Apify Google Search** -- Searches for recent news about the lead or company.
14. **Pinecone Vector Store (query)** -- Retrieves relevant company context.

### Message Generation
15. **AI Agent (message writer)** -- Takes all research and generates a personalized LinkedIn DM. System prompt instructs it to be conversational, reference specific details, and include a soft CTA.
16. **ElevenLabs TTS** -- Converts a short voice message into an audio file.
    - URL: `https://api.elevenlabs.io/v1/text-to-speech/YOUR_ELEVENLABS_VOICE_ID/stream`
    - Headers: `xi-api-key: YOUR_ELEVENLABS_API_KEY`

### Message Sending
17. **Unipile Send Message** -- Sends the text DM via LinkedIn.
    - URL: `https://YOUR_UNIPILE_DSN/api/v1/chats`
    - Creates a new chat with the lead and sends the message.
18. **Unipile Send Voice Note** -- Attaches the voice note to the conversation.
19. **Google Sheets (update)** -- Marks `1st message sent` = `yes` with timestamp.

### Loop Control
20. **Wait** -- Pauses between leads to avoid rate limits (30-60 seconds recommended).
21. **Loop Back** -- Processes next lead until daily limit reached.

---

## Google Sheet Structure

| Column | Description |
|--------|-------------|
| First Name | Lead first name |
| Last Name | Lead last name |
| LinkedIn Profile URL | Full LinkedIn URL |
| Company | Company name |
| Title | Job title |
| 1st Message Sent | `yes` or `no` |
| Message Sent Date | Timestamp |
| Reply Received | `yes` or `no` |
| Reply Date | Timestamp |
| Notes | Any additional context |

---

## ElevenLabs Voice Setup

1. Go to https://elevenlabs.io/
2. Navigate to **Voices** in the sidebar
3. Either use a pre-built voice or clone your own
4. Click on the voice and copy the **Voice ID** from the URL or settings
5. The voice ID looks like: `EXAVITQu4vr4xnSDxMaL`

---

## Testing Steps

1. **Test Apollo form:** Submit the form with a small search (1-2 leads)
2. **Check Google Sheet:** Verify leads appeared
3. **Run outreach manually:** Execute the Schedule Trigger path
4. **Check research nodes:** Click each Apify/Unipile node to verify data
5. **Check AI message:** Review the generated DM for quality
6. **Check ElevenLabs:** Verify voice note was generated (listen to output)
7. **Check LinkedIn:** Verify the message was sent (check your LinkedIn Sent messages)
8. **Start with your own profile:** Test by sending to yourself or a team member first

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Unipile 401 error | API key expired or invalid | Regenerate key in Unipile dashboard |
| LinkedIn rate limit | Sending too many messages/day | Reduce daily limit to 20-30; add longer Wait between sends |
| ElevenLabs audio empty | Voice ID incorrect or quota exceeded | Verify Voice ID; check ElevenLabs usage dashboard |
| Apify scraper timeout | Large profiles or slow targets | Increase timeout in Apify actor settings; add retry logic |
| Messages feel generic | Not enough research data feeding the AI | Check that Pinecone has content; verify all research nodes return data |
| Unipile "account not found" | LinkedIn not connected properly | Re-link LinkedIn in Unipile; make sure account is active |

---

## Daily Limits & Best Practices

- **LinkedIn connection requests:** Max 100/week (LinkedIn's limit)
- **LinkedIn DMs to non-connections:** Very limited; focus on connection requests first
- **Recommended daily volume:** 20-30 messages/day to stay under radar
- **Personalization quality:** The AI message should reference at least one specific detail about the person or their company. Review the first 10 manually to tune the prompt.
