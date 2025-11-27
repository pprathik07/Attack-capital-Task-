# 🚀 Attack Capital - Automated B2B Lead Generation & Outreach

**Fully automated B2B lead generation pipeline** that scrapes company directories, enriches contact data with decision-maker emails, and generates personalized AI-powered outreach emails—all without manual intervention.

Built for **SaaS companies, agencies, and sales teams** who want to scale outbound prospecting 10x faster.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Workflow Architecture](#workflow-architecture)
- [Prerequisites](#prerequisites)
- [API Setup Guide](#api-setup-guide)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Workflow Breakdown](#workflow-breakdown)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

**Attack Capital** automates the entire B2B lead generation process:

1. **Scrapes** company data from Clutch, DesignRush, and Google Search
2. **Filters** competitors and deduplicates entries
3. **Enriches** contacts using Apollo.io and Apify email finders
4. **Generates** personalized emails using Google Gemini AI
5. **Creates** Gmail drafts and logs everything to Google Sheets

Perfect for: Sales teams, marketing agencies, growth hackers, and B2B startups looking to automate cold outreach.

---

## ✨ Features

-  **Multi-Source Lead Scraping** - Pulls companies from Clutch, DesignRush, and Google Search
-  **Smart Competitor Filtering** - Automatically removes competitors (Aircall, Close.com, etc.) and deduplicates
-  **Decision-Maker Email Enrichment** - Finds verified emails for CEOs, Founders, VPs, Directors
-  **AI-Powered Personalization** - Google Gemini generates custom backlink partnership emails
-  **Automated Gmail Drafts** - Creates ready-to-send drafts in your Gmail account
-  **Google Sheets Integration** - Logs all enriched data for tracking and analytics
-  **Rate Limiting** - Built-in delays to respect API limits
-  **Batch Processing** - Handles multiple companies in parallel

---

## 🏗️ Workflow Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                             │
├─────────────────────────────────────────────────────────────────┤
│  Clutch Directory  │  DesignRush  │  Google Search (3 queries) │
└──────────┬──────────┴──────┬───────┴────────────┬───────────────┘
           │                 │                    │
           └─────────────────┴────────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Merge Sources   │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Filter & Dedupe   │
                    │ (Remove competitors)│
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Limit (50 max)  │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Apollo.io Enrich │
                    │  (Company Data)   │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Apify Email Finder│
                    │ (Decision Makers) │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Format & Filter   │
                    │ (Decision Makers) │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Google Sheets    │
                    │  (Log Contacts)   │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Google Gemini AI │
                    │ (Generate Emails) │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Create Gmail     │
                    │  Drafts           │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Creates Sheets   │
                    │  Data             │
                    └───────────────────┘
```

---

## 📋 Prerequisites

Before you begin, ensure you have:

- ✅ **n8n instance** (self-hosted or n8n.cloud)
- ✅ **Google Account** (for Gmail and Sheets)
- ✅ **Apify Account** (for web scraping)
- ✅ **Apollo.io Account** (for company enrichment)
- ✅ **Google Cloud Project** (for Gemini AI and Custom Search)

---

## 🔑 API Setup Guide

### 1️⃣ Apify API (Web Scraping)

Apify provides the web scraping actors for Clutch, DesignRush, and email finding.

**Steps:**

1. Go to [apify.com](https://apify.com) and create a free account
2. Navigate to **Settings** → **Integrations** → **API Tokens**
3. Click **Create new token** and give it a name (e.g., "n8n-workflow")
4. Copy the token (format: `apify_api_XXXXXXXXXXXX`)
5. In n8n, create a new **Apify API** credential and paste the token

**Required Actors:**
- `curious_coder~clutch-scraper` (scrapes Clutch directory)
- `dionysus_way~designrush-agency-scraper` (scrapes DesignRush)
- `snipercoder~decision-maker-email-finder` (finds decision-maker emails)

**Cost:** Free tier includes 5,000 API calls/month

---

### 2️⃣ Apollo.io API (Company Enrichment)

Apollo.io enriches company data and provides organizational information.

**Steps:**

1. Go to [apollo.io](https://www.apollo.io) and sign up for a free account
2. Navigate to **Settings** → **API**
3. Click **Create API Key** and copy it (format: `XXXXXXXXXXXXXXX`)
4. In the workflow's **Apollo(org-fetch)** node, replace the hardcoded API key:
```json
   {
     "name": "x-api-key",
     "value": "YOUR_APOLLO_API_KEY_HERE"
   }
```

**Cost:** Free tier includes 50 credits/month (1 credit per enrichment)

---

### 3️⃣ Google Gemini API (AI Email Generation)

Google Gemini generates personalized outreach emails.

**Steps:**

1. Go to [ai.google.dev](https://ai.google.dev)
2. Click **Get API Key** → **Create API key in new project**
3. Copy the API key (format: `AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXX`)
4. In n8n, create a new **Google Gemini (PaLM) API** credential
5. Paste the API key

**Cost:** Free tier includes 60 requests/minute

---

### 4️⃣ Google Custom Search API (Web Search)

Used to find SaaS blogs and additional company sources.

**Steps:**

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project or select an existing one
3. Enable **Custom Search API**:
   - Go to **APIs & Services** → **Library**
   - Search for "Custom Search API" and enable it
4. Create credentials:
   - Go to **APIs & Services** → **Credentials**
   - Click **Create Credentials** → **API Key**
   - Copy the API key (format: `AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXX`)
5. Create a Custom Search Engine:
   - Go to [programmablesearchengine.google.com](https://programmablesearchengine.google.com)
   - Click **Add** and set "Search the entire web"
   - Copy the **Search Engine ID** (format: `XXXXXXXXXXXXXXXXX`)
6. In the **Google Search: SaaS Blogs1** node, replace:
```
   https://www.googleapis.com/customsearch/v1?key=YOUR_API_KEY&cx=YOUR_SEARCH_ENGINE_ID
```

**Cost:** Free tier includes 100 queries/day

---

### 5️⃣ Gmail API (Create Drafts)

Allows the workflow to create email drafts in your Gmail account.

**Steps:**

1. In n8n, go to **Credentials** → **Add Credential**
2. Select **Gmail OAuth2**
3. Click **Connect my account** and authorize with Google
4. n8n will handle the OAuth flow automatically

**Cost:** Free (included with Gmail)

---

### 6️⃣ Google Sheets API (Data Logging)

Logs all enriched contacts to a Google Sheet for tracking.

**Steps:**

1. Create a new Google Sheet with these column headers:
```
   Company Name  | Company URL | Source | Contact Name | Contact Role | Email | Phone number | LinkedIn | Personalized Email Copy
```
2. Copy the Sheet URL
3. In n8n, create a new **Google Sheets OAuth2** credential
4. Click **Connect my account** and authorize
5. In the **Append to Google Sheet1** node, paste your Sheet URL

**Cost:** Free (included with Google Workspace)

---

## 📦 Installation

### Option 1: Import JSON Workflow
```bash
# 1. Download the workflow JSON file
wget https://raw.githubusercontent.com/yourusername/attack-capital/main/Attack%20capital%20(1).json

# 2. In your n8n instance:
# - Click "Workflows" → "Import from File"
# - Select the downloaded JSON file
# - Click "Import"
```

### Option 2: Manual Setup
```bash
# 1. Clone this repository
git clone https://github.com/yourusername/attack-capital.git
cd attack-capital

# 2. Open n8n (or install if you don't have it)
npx n8n

# 3. Create a new workflow and import the JSON file
```

---

## ⚙️ Configuration

### 1. Update API Credentials

After importing, configure these nodes:

| Node Name | Credential Type | Required |
|-----------|----------------|----------|
| Source A: Clutch Directory | Apify API | ✅ |
| Source B: DesignRush | Apify API | ✅ |
| HTTP Request (Email Finder) | Apify API Token (in URL) | ✅ |
| Apollo(org-fetch) | Apollo.io API Key (in headers) | ✅ |
| Google Search: SaaS Blogs1 | Google Custom Search API | ✅ |
| Google Gemini Chat Model | Google Gemini API | ✅ |
| Append to Google Sheet1 | Google Sheets OAuth2 | ✅ |
| Create a draft | Gmail OAuth2 | ✅ |

### 2. Customize Search Queries

In the **Code2** node, modify the search queries:
```javascript
const queries = [
  'outbound sales blog',  // Change these to your target keywords
  'SaaS sales strategies',
  'cold calling best practices'
];
```

### 3. Update Competitor List

In the **Filter Competitors & Deduplicate** node:
```javascript
const competitors = [
  'aircall', 'justcall', 'close.com', 'dialpad',  // Add your competitors
  'cloudtalk', 'phoneburner', 'ringcentral'
];
```

### 4. Adjust Limits

- **Limit** node: Controls how many companies to process (default: 50)
- **Limit2** node: Controls enrichment batch size (default: 20)
- **Limit3** node: Controls email generation batch size (default: 5)

---

## 🚀 Usage

### Running the Workflow

1. **Manual Trigger**: Click **Execute Workflow** in n8n
2. **Schedule**: Add a **Schedule** trigger node for automatic runs (e.g., daily at 9 AM)
3. **Webhook**: Add a **Webhook** trigger for API-based execution

### Expected Output

After execution, you'll get:

- ✅ **Google Sheet** populated with enriched contacts
- ✅ **Gmail drafts** with personalized emails ready to send
- ✅ **Console logs** showing progress and stats

### Example Results
```
✅ Filtered: 150 → 47 companies (after deduplication)
📧 Found 23 decision-maker contacts
🤖 Generated 5 personalized emails
📨 Created 5 Gmail drafts
```

---

## 🔧 Workflow Breakdown

### Stage 1: Data Collection

**Nodes:**
- `Source A: Clutch Directory` - Scrapes sales/marketing agencies
- `Source B: DesignRush` - Scrapes design/marketing agencies
- `Google Search: SaaS Blogs1` - Searches for SaaS content sites

**Output:** Raw company data with names, websites, descriptions

---

### Stage 2: Data Cleaning

**Nodes:**
- `Merge All Sources` - Combines data from all sources
- `Filter Competitors & Deduplicate` - Removes competitors and duplicates by domain
- `Limit` - Caps processing at 50 companies

**Output:** Clean, unique company list

---

### Stage 3: Contact Enrichment

**Nodes:**
- `Apollo(org-fetch)` - Gets company details (size, industry, LinkedIn)
- `HTTP Request` (Apify) - Finds decision-maker emails
- `Code6` - Filters for C-level, founders, VPs, directors only

**Output:** Verified decision-maker contacts with emails

---

### Stage 4: Email Generation

**Nodes:**
- `AI Agent` + `Google Gemini Chat Model` - Generates custom partnership emails
- `Code1` - Extracts subject line and body from AI response

**Output:** Personalized email copy for each prospect

---

### Stage 5: Output

**Nodes:**
- `Append to Google Sheet1` - Logs all data to Google Sheets
- `Create a draft` - Creates Gmail drafts for manual review/sending

**Output:** Trackable database + ready-to-send emails

---

## 🎨 Customization

### Change Email Template

Edit the prompt in the **AI Agent** node to customize email style:
```plaintext
# Current: Backlink partnership outreach
# Options:
- Product demo requests
- Consultation offers
- Content collaboration
- Partnership proposals
```

### Add More Data Sources

Add new **HTTP Request** nodes to scrape:
- LinkedIn Sales Navigator results
- Industry-specific directories (G2, Capterra)
- Company databases (Crunchbase, ZoomInfo)

### Modify Filtering Logic

Edit **Filter Competitors & Deduplicate** to:
- Change company size filters (e.g., only 50-200 employees)
- Add location filtering (e.g., only US companies)
- Filter by industry keywords

---

## 🐛 Troubleshooting

### Issue: "API rate limit exceeded"

**Solution:** Increase wait times in `Wait Between Searches` and `Wait Between Enrichments` nodes

---

### Issue: "No emails found"

**Solution:**
- Check Apify actor is configured correctly
- Verify Apollo.io credits haven't expired
- Try different search queries in Code2 node

---

### Issue: "Gmail drafts not created"

**Solution:**
- Re-authenticate Gmail OAuth2 credential
- Check Gmail API is enabled in Google Cloud Console
- Verify email format in Code1 node

---

### Issue: "Google Sheets not updating"

**Solution:**
- Verify Sheet URL in `Append to Google Sheet1` node
- Check column headers match exactly (case-sensitive)
- Re-authorize Google Sheets OAuth2 credential

---

## 🙏 Acknowledgments

- Built with [n8n](https://n8n.io) - Fair-code workflow automation
- Powered by [Google Gemini](https://ai.google.dev) for AI email generation
- Data enrichment by [Apollo.io](https://apollo.io) and [Apify](https://apify.com)

---

## 📧 Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/attack-capital/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/attack-capital/discussions)
- **Email**: your.email@example.com

---

**Made with passion by [Prathik]**

⭐ Star this repo if you find it helpful!
