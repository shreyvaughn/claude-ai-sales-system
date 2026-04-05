# Workflow 10: Pinecone Knowledgebase

## Overview

This workflow ingests your company documents into a Pinecone vector database for retrieval-augmented generation (RAG). It downloads a file from Google Drive, splits the text into chunks, generates OpenAI embeddings for each chunk, and upserts them into a Pinecone index. Once populated, other workflows (Email Sequencing, Objection Handling, Meeting Notes, etc.) can query this knowledgebase to generate context-aware, personalized content instead of generic AI output.

This is a supporting workflow -- it does not run on a schedule or trigger automatically. You run it manually whenever you want to add or update documents in your knowledge base.

---

## Required Services

- [ ] **Google Drive** -- Google account with Drive access (to host source documents)
- [ ] **Pinecone** -- https://www.pinecone.io/ (vector database for storing embeddings)
- [ ] **OpenAI** -- https://platform.openai.com/ (embedding model)

---

## API Keys & Credentials Needed

| Credential | Where to Get It | n8n Credential Type |
|-----------|----------------|-------------------|
| Google Drive OAuth | n8n Credentials > Google Drive > OAuth2 | Google Drive OAuth2 |
| Pinecone API Key | Pinecone Console > API Keys | Pinecone API |
| OpenAI API Key | platform.openai.com > API Keys | OpenAI API |

---

## Variables to Customize

| Variable | Description | Where Used |
|----------|-------------|-----------|
| `YOUR_OPENAI_API_KEY` | OpenAI key for generating embeddings | Embeddings OpenAI node |
| `YOUR_PINECONE_API_KEY` | Pinecone API key | Pinecone Vector Store node |
| `YOUR_PINECONE_INDEX_NAME` | Name of your Pinecone index | Pinecone Vector Store node config |
| `YOUR_GOOGLE_DRIVE_FILE_ID` | ID of the document to ingest | Download File node |

---

## Pinecone Setup

### 1. Create an Account

Go to https://www.pinecone.io/ and sign up. The free Starter plan provides enough capacity for most small-to-medium knowledge bases.

### 2. Create an Index

1. In the Pinecone Console, click **Create Index**
2. Configure:
   - **Index Name:** Choose a descriptive name (e.g., `company-knowledge`, `sales-context`)
   - **Dimensions:** `1536` (this matches OpenAI's `text-embedding-ada-002` model)
   - **Metric:** `cosine`
   - **Cloud/Region:** Choose the region closest to your n8n server
3. Click **Create Index**
4. Note the index name -- you will use it in the workflow

### 3. Get Your API Key

1. Go to **API Keys** in the Pinecone Console
2. Copy your API key
3. Create a Pinecone credential in n8n using this key

---

## Node-by-Node Configuration

### Trigger
1. **Manual Trigger** -- Click "Execute Workflow" to run. No schedule or webhook needed.

### Document Download
2. **Download File** (Google Drive) -- Downloads a document from Google Drive.
   - **Configure:**
     - Operation: Download
     - File ID: `YOUR_GOOGLE_DRIVE_FILE_ID`
   - **Finding the File ID:** Open the document in Google Drive. The ID is in the URL:
     ```
     https://docs.google.com/document/d/YOUR_GOOGLE_DRIVE_FILE_ID/edit
     ```
   - Supports Google Docs, plain text, PDFs, and other text-based formats.

### Text Processing
3. **Default Data Loader** -- Converts the downloaded file's binary data into text that the splitter can process.
   - Data Type: Binary
   - No changes needed unless you switch to a different input format.

4. **Recursive Character Text Splitter** -- Splits the document text into smaller chunks for embedding.
   - Default chunk size and overlap work well for most documents.
   - **Optional tuning:**
     - Increase chunk size (e.g., 1000-2000 characters) for documents with long, connected paragraphs
     - Increase overlap (e.g., 200 characters) to preserve context across chunk boundaries

### Embedding & Storage
5. **Embeddings OpenAI** -- Generates vector embeddings for each text chunk using OpenAI's embedding model.
   - Uses `text-embedding-ada-002` by default (1536 dimensions)
   - Make sure your Pinecone index dimensions match (1536)

6. **Pinecone Vector Store** -- Upserts the embedded chunks into your Pinecone index.
   - Mode: Insert
   - Index: `YOUR_PINECONE_INDEX_NAME`
   - Each chunk becomes a vector with the original text stored as metadata

---

## What Documents to Ingest

For best results with your sales workflows, ingest these types of content:

| Document Type | Why It Helps |
|---------------|-------------|
| Product/service descriptions | AI can reference specific features in emails |
| Case studies | AI can cite real results in outreach |
| FAQ / objection responses | Improves objection handling workflow |
| Pricing information | AI can reference tiers and plans |
| Company background | Adds credibility context to cold emails |
| Competitor comparisons | AI can position against alternatives |
| Ideal customer profiles | Helps AI tailor messaging by industry/role |

### Tips for Document Preparation

- **One document per run:** The workflow processes a single Google Drive file at a time. Run it once per document.
- **Plain text works best:** Google Docs are ideal. If using PDFs, ensure they contain selectable text (not scanned images).
- **Keep documents focused:** A single document covering one topic (e.g., "Product Overview") produces better retrieval results than a massive document covering everything.
- **Update regularly:** When your product or messaging changes, re-run this workflow with updated documents. Old vectors remain in Pinecone unless you delete the index and re-create it.

---

## Ingesting Multiple Documents

To ingest several documents, you have two options:

**Option A: Run the workflow multiple times**
Change the Google Drive File ID each time and execute. All chunks go into the same Pinecone index.

**Option B: Modify the workflow for batch processing**
1. Replace the Manual Trigger + Download File nodes with a Google Drive "List Files" node that reads all files from a specific folder
2. Add a Loop node to process each file
3. This is a more advanced modification but saves time for large document sets

---

## Testing Steps

1. **Prepare a test document** in Google Drive (even a short paragraph works)
2. **Copy the File ID** from the document URL
3. **Paste it** into the Download File node's File ID field
4. **Run manually:** Click "Execute Workflow"
5. **Check the Download File node:** Verify the document content was retrieved
6. **Check the Text Splitter node:** Verify it produced multiple chunks
7. **Check the Pinecone Vector Store node:** Verify it reports successful upserts
8. **Verify in Pinecone Console:** Go to your index > Browse > you should see vectors with your document text in the metadata

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Download fails | File ID wrong or permissions issue | Double-check the File ID; ensure the Google Drive credential has access to the file |
| Pinecone dimension mismatch | Index dimensions do not match embedding model | Delete and recreate the index with 1536 dimensions (for OpenAI ada-002) |
| Empty chunks | Document is an image-only PDF or empty file | Use text-based documents (Google Docs work best) |
| "Index not found" error | Wrong index name in the node | Copy the exact index name from Pinecone Console |
| Embeddings fail | OpenAI API key invalid or quota exceeded | Verify your API key; check your OpenAI usage dashboard |
| Old content still appearing in queries | Pinecone does not auto-delete old vectors | Delete the index and recreate it, then re-ingest all documents |
