# Workflow 13: Full Newsletter System

## Overview

This workflow automates newsletter content curation and delivery. It has three main functions: (1) managing subscriber sign-ups and removals via webhooks, (2) ingesting RSS feed content into Pinecone for AI-powered content retrieval, and (3) generating a weekly digest newsletter using AI that pulls the most relevant recent content from Pinecone and formats it as a professional newsletter. The digest is saved as a Google Doc and can be connected to your email sending platform.

---

## Required Services

- [ ] **Pinecone** -- https://www.pinecone.io/ (vector storage for content)
- [ ] **OpenAI** -- https://platform.openai.com/ (content curation + writing)
- [ ] **Google Docs** -- For newsletter draft output
- [ ] **RSS Feeds** -- URLs of content sources you want to curate from
- [ ] **Notion** (optional) -- For subscriber management

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Pinecone API Key | Pinecone Console > API Keys | Pinecone API |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |
| Google Service Account | Google Cloud Console > IAM > Service Accounts | Google API (Service Account) |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store nodes |
| `YOUR_PINECONE_INDEX_NAME` | Index for newsletter content | Pinecone node config |
| `YOUR_OPENAI_API_KEY` | OpenAI key | Content generation + embeddings |
| `YOUR_GOOGLE_SERVICE_ACCOUNT_CREDENTIAL` | Google service account | Google Docs creation |
| `YOUR_RSS_FEED_URLS` | Array of RSS feed URLs | RSS Feed Reader nodes |
| `YOUR_NEWSLETTER_NAME` | Name of your newsletter | Newsletter header/branding |
| `YOUR_COMPANY_NAME` | Company name | Newsletter footer |

---

## Node-by-Node Configuration

### Part 1: Subscriber Management

#### New Subscriber Webhook
1. **Webhook (add subscriber)** -- Receives POST when someone subscribes:
   - Path: `add-new-newsletter-subscriber`
   - Expects: subscriber name and email
2. **Edit Fields** -- Extracts subscriber's full name from the payload.
3. **Code (parse name)** -- Extracts first name for personalization.
4. **Create Subscription** -- Stores the subscriber (in your database, Notion, or a Google Sheet).

#### Unsubscribe Webhook
5. **Webhook (delete subscriber)** -- Receives POST for unsubscribes:
   - Path: `delete-newsletter-subscriber`
6. **Edit Fields** -- Extracts subscriber ID.
7. **Delete Subscription** -- Removes the subscriber from your list.

### Part 2: Content Ingestion (RSS to Pinecone)

8. **Schedule Trigger** -- Runs daily or weekly to check for new content.
9. **RSS Feed Reader** -- Reads articles from your configured RSS feeds:
   - Add multiple RSS Read nodes for different sources
   - Configure each with a feed URL
10. **Code (deduplicate)** -- Filters out articles already indexed.
11. **Text Splitter** -- Splits article content into chunks.
12. **Embeddings OpenAI** -- Generates vector embeddings.
13. **Pinecone Vector Store (insert)** -- Stores content with metadata (source, date, title, URL).

### Part 3: Newsletter Generation

14. **Schedule Trigger (weekly)** -- Fires once per week (e.g., every Monday).
15. **Pinecone Vector Store (query)** -- Retrieves the most relevant/recent content:
    - Query: topics relevant to your audience
    - Filter: content from the last 7 days
    - Top K: 10-20 articles
16. **AI Agent (newsletter writer)** -- Takes the curated content and generates:
    - A compelling subject line
    - An intro paragraph with a personal note
    - 5-8 curated article summaries with links
    - A closing section with CTA
    - HTML formatting for email clients
17. **Google Docs (create)** -- Creates a Google Doc with the newsletter draft for review.

---

## RSS Feed Recommendations

Choose feeds relevant to your audience. Examples by category:

### Sales & Marketing
- HubSpot Blog: `https://blog.hubspot.com/rss.xml`
- Sales Hacker: `https://www.saleshacker.com/feed/`

### AI & Tech
- OpenAI Blog: `https://openai.com/blog/rss.xml`
- TechCrunch AI: `https://techcrunch.com/category/artificial-intelligence/feed/`

### Business
- Harvard Business Review: `https://hbr.org/rss`
- First Round Review: `https://firstround.com/review/feed.xml`

Add your industry-specific feeds to make the newsletter relevant to your audience.

---

## Pinecone Index Setup

You can reuse the same Pinecone index as the knowledgebase, or create a separate one:

- Name: `newsletter-content` (or `company-knowledge` with a `newsletter` namespace)
- Dimensions: `1536`
- Metric: `cosine`

Using namespaces is recommended to keep newsletter content separate from sales knowledge.

---

## Testing Steps

1. **Test subscriber webhook:**
   ```bash
   curl -X POST YOUR_N8N_WEBHOOK_URL/add-new-newsletter-subscriber \
     -H "Content-Type: application/json" \
     -d '{"data": {"properties": {"Name": {"title": [{"plain_text": "Test User"}]}, "Email": {"email": "test@example.com"}}}}'
   ```
2. **Test RSS ingestion:** Run the content ingestion path manually. Check that articles are stored in Pinecone.
3. **Test newsletter generation:** Run the weekly generation path. Check the Google Doc output.
4. **Review the newsletter:** Open the doc and evaluate:
   - Is the content relevant?
   - Are the summaries accurate?
   - Are the links correct?
   - Does the formatting look good?

---

## Connecting to Email Sending

The workflow generates a newsletter as a Google Doc. To actually send it:

**Option A: Manual send** -- Copy the content from the Google Doc into your email platform (Mailchimp, ConvertKit, Beehiiv, etc.)

**Option B: Add an email node** -- After the Google Docs node, add:
- A Mailchimp/ConvertKit/SendGrid node to send to your subscriber list
- Or use Gmail/SMTP to send directly (only for small lists)

**Option C: Connect to Beehiiv API** -- If you use Beehiiv, add an HTTP Request node to create a draft post via their API.

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| RSS feed returns empty | Feed URL invalid or site blocks scraping | Verify the URL opens in a browser; try a different feed from the same site |
| Duplicate articles each week | No dedup logic | Add a check against Pinecone metadata for existing article URLs |
| Newsletter too long | Too many articles retrieved from Pinecone | Reduce Top K to 5-8; add "Keep it under 1000 words" to the AI prompt |
| Newsletter off-topic | RSS feeds not relevant | Curate your feed list carefully; add topic filtering in the Pinecone query |
| Subscriber webhook format wrong | Payload structure mismatch | Adjust the Edit Fields node to match your actual webhook payload format |
| Google Doc formatting issues | HTML content pasted as plain text | Use Google Docs "insert" action with rich text, or format as Markdown |
