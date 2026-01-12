# Real Estate Automation Suite - Setup Guide

This document covers the setup and configuration for three n8n workflows that automate real estate operations.

## 📦 Included Workflows

| # | Workflow Name | File | Purpose |
|---|--------------|------|---------|
| 1 | Web Lead Qualification & Scoring | `workflow-1-web-lead-qualification.json` | Capture, validate, classify, and score incoming web leads |
| 2 | AI Data Research & Content Generation | `workflow-2-data-research-content.json` | Fetch data, extract insights with AI, output to Google Docs/Sheets |
| 3 | Voice Call Lead Outreach | `workflow-3-voice-call-outreach.json` | Automated voice calls with AI-powered lead qualification |

---

## 🔧 Required Credentials (All Workflows)

### OpenAI API
**Used by:** All three workflows  
**Purpose:** GPT-4o Mini for intent classification, data extraction, and lead qualification

1. Go to [platform.openai.com](https://platform.openai.com)
2. Create an API key
3. In n8n: Settings → Credentials → Add credential → OpenAI API
4. Paste your API key

### Google Sheets OAuth2
**Used by:** Workflows 2 and 3  
**Purpose:** Logging extracted data and qualified leads

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create/select a project
3. Enable Google Sheets API
4. Create OAuth2 credentials (Desktop app type)
5. In n8n: Settings → Credentials → Add credential → Google Sheets OAuth2
6. Authorize with your Google account

### Google Docs OAuth2
**Used by:** Workflow 2  
**Purpose:** Updating analysis documents

Follow the same steps as Google Sheets, enabling Google Docs API.

---

## 📋 Workflow 1: Web Lead Qualification & Scoring

### What It Does
1. Receives leads via webhook from your web forms
2. Validates that the user message is present and meaningful
3. Uses AI to classify intent (buying/selling/renting/inquiry) and urgency (high/medium/low)
4. Checks against a property database API
5. Calculates a lead score (0-100) with priority tier (Hot/Warm/Medium/Low)
6. Returns structured lead data via webhook response

### Configuration Steps

1. **Import the workflow** into n8n
2. **Connect OpenAI credential** to the "LLM for Lead Intent Classification" node
3. **Update the Property Check API URL** in the "Call Property Check API" node
   - Replace `https://api.example.com/property-check` with your actual API
   - Or remove this node if you don't have a property database
4. **Activate the workflow** to get your webhook URL
5. **Integrate the webhook URL** into your web forms

### Webhook URL
After activation: `https://your-n8n-instance.com/webhook/incoming-lead`

### Expected Payload
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "555-123-4567",
  "userMessage": "I'm looking to buy a 3-bedroom home in the downtown area within the next 2 months. Budget around $500K.",
  "propertyAddress": "123 Main St"
}
```

### Response Format
```json
{
  "success": true,
  "leadId": "john@example.com-1704067200000",
  "score": 75,
  "priority": "Hot"
}
```

---

## 📋 Workflow 2: AI Data Research & Content Generation

### What It Does
1. Runs on schedule (hourly) or manually
2. Fetches data from an external URL
3. Uses AI to extract structured information
4. AI agent performs research with web search and calculator tools
5. Outputs analysis to Google Docs
6. Logs extracted items to Google Sheets
7. Generates executive summary and newsletter formats

### Configuration Steps

1. **Import the workflow** into n8n
2. **Connect credentials:**
   - OpenAI API → All LLM nodes
   - Google Sheets OAuth2 → "Log AI Analysis Data to Google Sheets"
   - Google Docs OAuth2 → "Update Google Doc with AI Agent Analysis"
   - SerpAPI (optional) → "SerpAPI Web Search Tool for AI Agent"
3. **Update the data source URL** in "Fetch External Data for AI Analysis"
4. **Create Google Sheets:**
   - Spreadsheet with columns: `Timestamp`, `Title`, `Category`, `Summary`, `Relevance Score`
   - Select it in the Google Sheets node
5. **Create a Google Doc** for analysis output and select it
6. **Adjust schedule** in "Scheduled Trigger" node (default: every hour)

### Optional: SerpAPI Setup
For web search capabilities:
1. Get API key from [serpapi.com](https://serpapi.com)
2. Add SerpAPI credential in n8n
3. Connect to the SerpAPI tool node

---

## 📋 Workflow 3: Voice Call Lead Outreach

### What It Does
1. Receives new leads via webhook
2. Generates personalized call script
3. Converts script to speech using ElevenLabs
4. Places automated call via Twilio
5. Captures DTMF responses (1=interested, 2=callback, 3=not interested)
6. AI analyzes call interaction and qualifies lead
7. Scores interested leads and logs to Google Sheets
8. Generates AI summary of each interaction

### Configuration Steps

#### 1. ElevenLabs Setup
1. Create account at [elevenlabs.io](https://elevenlabs.io)
2. Get your API key
3. Choose a voice ID (default: `21m00Tcm4TlvDq8ikWAM` - Rachel)
4. In n8n: Create HTTP Header Auth credential
   - Header Name: `xi-api-key`
   - Header Value: Your ElevenLabs API key
5. Connect to "ElevenLabs: Convert Intro Script to Voice" node

#### 2. Twilio Setup
1. Create account at [twilio.com](https://twilio.com)
2. Get your Account SID and Auth Token
3. Purchase a phone number with voice capability
4. In n8n: Create HTTP Basic Auth credential
   - Username: Your Account SID
   - Password: Your Auth Token
5. Update the Twilio node:
   - Replace `YOUR_ACCOUNT_SID` in the URL
   - Replace `+15551234567` with your Twilio phone number
6. **Important:** Set up a TwiML bin or webhook to handle DTMF responses

#### 3. Google Sheets Setup
Create a spreadsheet with two sheets:

**Sheet 1: "Leads"**
| Timestamp | Name | Phone | Email | Property | Score | Priority | Interest Level | Budget | Timeline | Recommended Action | Status |
|-----------|------|-------|-------|----------|-------|----------|----------------|--------|----------|-------------------|--------|

**Sheet 2: "LeadsSummary"**
| Timestamp | LeadName | Summary | InterestLevel | NextSteps |
|-----------|----------|---------|---------------|-----------|

#### 4. Error Notification (Optional)
Create a separate workflow to handle Google Sheets logging failures:
1. Create a simple workflow with email or Slack notification
2. Note the workflow ID
3. Update "Error Trigger: Notify Admin of Sheets Failure" node with that ID

### Webhook URL
After activation: `https://your-n8n-instance.com/webhook/new-lead`

### Expected Payload
```json
{
  "name": "Jane Smith",
  "phone": "+15551234567",
  "email": "jane@example.com",
  "propertyRef": "456 Oak Ave"
}
```

---

## 🧪 Testing Instructions

### Workflow 1 (Web Lead)
```bash
curl -X POST https://your-n8n-instance.com/webhook/incoming-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Lead",
    "email": "test@example.com",
    "phone": "555-0000",
    "userMessage": "I am interested in buying a home in the next 30 days. My budget is $400,000.",
    "propertyAddress": "123 Test St"
  }'
```

### Workflow 2 (Data Research)
1. Open the workflow in n8n
2. Click "Execute Workflow" (manual trigger)
3. Check execution logs
4. Verify Google Docs/Sheets updated

### Workflow 3 (Voice Call)
1. **Test with your own phone number first!**
2. Send a webhook request with your number
3. Answer the call and press DTMF digits
4. Check Google Sheets for logged data

---

## ⚠️ Important Notes

### Rate Limits
- **OpenAI:** Monitor your API usage
- **ElevenLabs:** Free tier has character limits
- **Twilio:** Calls have per-minute charges

### Compliance
- **TCPA (US):** Ensure compliance with telephone regulations
- **GDPR (EU):** Handle personal data appropriately
- **Do Not Call lists:** Respect opt-out requests

### Security
- Keep API keys secure
- Use HTTPS for all webhooks
- Consider adding webhook authentication

---

## 🔄 Customization Ideas

1. **Add CRM integration** - Push qualified leads to Salesforce, HubSpot, or GoHighLevel
2. **SMS follow-up** - Send text messages to interested leads
3. **Email automation** - Trigger drip campaigns based on lead score
4. **Slack notifications** - Alert sales team for hot leads
5. **Custom scoring** - Adjust scoring weights in the Code nodes

---

## 📞 Support

For issues with these workflows:
1. Check n8n execution logs for errors
2. Verify all credentials are connected
3. Test individual nodes with pinned data
4. Review API documentation for external services
