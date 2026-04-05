# Workflow 1: Lead Finder & Enrichment

## Overview

This workflow searches Apollo.io for leads matching your Ideal Customer Profile (ICP), enriches the data with additional details (email, phone, company info), and exports everything to a Google Sheet. It serves as the entry point for your entire sales pipeline -- every other outreach workflow pulls leads from the sheet this one populates.

Note: The Agentic AI SDR (Workflow 2) includes built-in lead finding. If you plan to use that workflow, you may not need this one separately. This standalone version is useful if you want to batch-find leads without triggering the full SDR research pipeline.

---

## Required Services

- [ ] **Apollo.io** -- https://www.apollo.io/ (free tier available, paid for higher volume)
- [ ] **Google Sheets** -- Google account with Sheets access
- [ ] **OpenAI** -- https://platform.openai.com/ (for data formatting/analysis)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Apollo API Key | Apollo Settings > API > Generate Key | HTTP Header Auth |
| Google Sheets OAuth | n8n Credentials > Google Sheets > OAuth2 | Google Sheets OAuth2 |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |

---

## Variables to Customize

| Variable | Description | Example |
|----------|-------------|---------|
| `YOUR_APOLLO_API_KEY` | Apollo API key for lead search | `ak_xxxxxxxxxx` |
| `YOUR_GOOGLE_SHEETS_ID` | ID of your lead tracking spreadsheet | `1abc123def456...` (from the URL) |
| `YOUR_OPENAI_API_KEY` | OpenAI API key | `sk-xxxxxxxxxx` |

### Google Sheet Setup

Create a Google Sheet with these columns in the first row:

| A | B | C | D | E | F | G | H | I | J |
|---|---|---|---|---|---|---|---|---|---|
| First Name | Last Name | Email | Phone Number | Company | Company Website | LinkedIn URL | Title | Stage | Last Contact Date |

The Sheet ID is the long string in the URL between `/d/` and `/edit`:
```
https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEETS_ID/edit
```

---

## Node-by-Node Configuration

### 1. Schedule Trigger (or Manual Trigger)
- **What it does:** Kicks off the lead search on a schedule or manually
- **Configure:** Set your preferred frequency (daily recommended)
- **Tip:** Start with manual trigger for testing, switch to schedule once verified

### 2. Apollo API Request
- **What it does:** Calls Apollo's People Search endpoint
- **Configure:**
  - Method: `POST`
  - URL: `https://api.apollo.io/v1/mixed_people/search`
  - Headers: `x-api-key: YOUR_APOLLO_API_KEY`
  - Body: JSON with your ICP filters (title, industry, company size, location)
- **Customize the search body:**
```json
{
  "page": 1,
  "per_page": 100,
  "person_titles": ["CEO", "CTO", "VP of Sales", "Head of Marketing"],
  "organization_num_employees_ranges": ["11,50", "51,200"],
  "person_locations": ["United States"],
  "organization_industry_tag_ids": []
}
```

### 3. Data Formatting (Code Node)
- **What it does:** Structures Apollo's response into clean rows
- **Configure:** Maps Apollo fields to your Google Sheet columns
- **No changes needed** unless you added custom columns to your sheet

### 4. Google Sheets (Append)
- **What it does:** Writes enriched leads to your spreadsheet
- **Configure:**
  - Document ID: `YOUR_GOOGLE_SHEETS_ID`
  - Sheet: Select your sheet tab name
  - Column mapping: Match to your header row

---

## Testing Steps

1. **Manual run:** Click "Execute Workflow" in n8n
2. **Check Apollo node:** Should return an array of people objects
3. **Check Code node:** Should output clean rows with the columns you defined
4. **Check Google Sheets:** Open your spreadsheet and verify new rows appeared
5. **Verify data quality:** Spot-check a few emails and LinkedIn URLs

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Apollo returns empty results | ICP filters too narrow | Broaden title or location filters |
| Google Sheets auth error | OAuth token expired | Re-authenticate in n8n Credentials |
| Duplicate leads | No dedup check | Add a Merge/Compare node before Sheets, or use a "filter existing" step |
| Rate limit errors | Too many Apollo calls | Add a Wait node (1-2 seconds) between API calls, or reduce batch size |
