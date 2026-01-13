# 🏠 Daily Real Estate Deal Finder - Setup Guide

## Overview

This n8n automation sends you daily real estate investment deals via email every morning at 9AM. It:

1. **Searches** Zillow for properties matching your criteria
2. **Calculates** key investment metrics (Cash on Cash ROI, Monthly Cash Flow, Down Payment, etc.)
3. **Filters** to only positive cash flow deals
4. **Saves** results to Google Sheets for tracking
5. **Emails** you a beautiful HTML digest with the top deals

---

## Prerequisites

Before importing this workflow, you'll need:

- ✅ **n8n account** (cloud or self-hosted)
- ✅ **Google account** with OAuth2 setup
- ✅ **Google Sheets API** enabled
- ✅ **RapidAPI account** with Zillow API subscription
- ✅ **Blank Google Sheet** created

---

## Step-by-Step Setup

### Step 1: Import the Workflow

1. Open n8n and go to **Workflows**
2. Click **Import from File**
3. Select `daily-real-estate-deal-finder.json`
4. The workflow will appear with all nodes configured

---

### Step 2: Set Up Google OAuth2 Credentials

📺 **Video Tutorial**: https://youtu.be/LTuy83t_Rt4

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project (or use existing)
3. Enable these APIs:
   - Google Sheets API
   - Gmail API
4. Configure OAuth consent screen
5. Create OAuth2 credentials
6. In n8n, create a new **Google Sheets OAuth2** credential
7. Authorize with your Google account

---

### Step 3: Set Up RapidAPI Zillow Credentials

1. Go to [RapidAPI](https://rapidapi.com)
2. Search for **"Zillow API by apimaker"** or similar
3. Subscribe to a plan (there are free tiers available)
4. Copy your **X-RapidAPI-Key**

5. In n8n, create a new **HTTP Header Auth** credential:
   - **Name**: `RapidAPI Zillow`
   - Add these headers:
     - `X-RapidAPI-Key`: `your-api-key-here`
     - `X-RapidAPI-Host`: `zillow-com1.p.rapidapi.com`

6. Connect this credential to both HTTP Request nodes in the workflow

---

### Step 4: Create Your Google Sheet

Create a new Google Sheet with these exact column headers in Row 1:

| A | B | C | D | E | F | G | H | I | J |
|---|---|---|---|---|---|---|---|---|---|
| Address | Price | Rent Zestimate | Cash on Cash ROI | Monthly Cash Flow | Down Payment | Monthly Maintenance | Vacancy Loss | Property URL | Date Added |

Then in n8n:
1. Open the **"Save to Google Sheets"** node
2. Select your Google Sheets credential
3. Choose your spreadsheet from the dropdown
4. Select the worksheet (usually "Sheet1")

---

### Step 5: Configure Search Parameters

Open the **"Set Search Parameters"** node and customize:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `location` | Target market | `"Austin, TX"` or `"Phoenix, AZ"` |
| `min_beds` | Minimum bedrooms | `2` |
| `min_baths` | Minimum bathrooms | `2` |
| `min_price` | Minimum price | `250000` |
| `max_price` | Maximum price | `400000` |
| `property_type` | Type of property | `"SingleFamily"` |
| `down_payment_pct` | Down payment % | `0.2` (20%) |
| `interest_rate` | Current mortgage rate | `0.07` (7%) |
| `loan_term_years` | Loan term | `30` |
| `vacancy_rate` | Vacancy allowance | `0.08` (8%) |
| `maintenance_rate` | Maintenance reserve | `0.1` (10%) |
| `recipient_email` | Your email | `"your-email@gmail.com"` |

---

### Step 6: Connect Gmail Credential

1. Open the **"Send Email Digest"** node
2. Select your Gmail OAuth2 credential (same as Google Sheets)
3. The recipient email comes from parameters, but verify it looks correct

---

### Step 7: Test the Workflow

1. Click **"Test Workflow"** (or execute manually)
2. Watch each node execute
3. Check for any errors in the API calls
4. Verify data appears in your Google Sheet
5. Check your email for the digest

---

## Investment Metrics Explained

The workflow calculates these key metrics:

### Down Payment
```
Down Payment = Purchase Price × 20%
```

### Monthly Mortgage (P&I)
Uses standard amortization formula for 30-year fixed rate.

### Vacancy Loss
```
Vacancy Loss = Rent Estimate × 8%
```
Industry standard for single-family rentals.

### Maintenance Reserve
```
Maintenance = Rent Estimate × 10%
```
Covers repairs, CapEx, and general upkeep.

### Monthly Cash Flow
```
Cash Flow = Rent - Mortgage - Vacancy - Maintenance - Tax/Insurance
```

### Cash on Cash ROI
```
ROI = (Annual Cash Flow / Total Cash Invested) × 100
```
Where Total Cash Invested = Down Payment + Closing Costs (~3%)

---

## Troubleshooting

### Zillow API Fails
- Verify your RapidAPI key is correct
- Check your subscription tier and rate limits
- Some free tiers have daily limits

### Calculation Errors
- Open the Code node and check the JavaScript
- Verify property data is being passed correctly
- Use n8n's built-in debugger to inspect data

### Google Sheets Errors
- Ensure column headers match exactly
- Re-authorize your Google credential if expired
- Check that the spreadsheet ID is correct

### No Results Found
- Broaden your search criteria
- Check if the location format is correct (City, State)
- Verify price range is realistic for the market

### Email Not Sending
- Check Gmail credential authorization
- Verify the recipient email is valid
- Check spam folder

---

## Customization Ideas

1. **Multiple Markets**: Duplicate the workflow for different cities
2. **Slack Notifications**: Add a Slack node alongside email
3. **Database Storage**: Use Airtable or Supabase instead of Sheets
4. **Better Filtering**: Add min ROI threshold or cap rate calculations
5. **Property Photos**: Fetch and include listing photos in email
6. **Competitor Analysis**: Add Redfin or Realtor.com data

---

## Support

If you have questions or issues:
1. Check the sticky notes in the workflow for inline help
2. Review the n8n documentation at docs.n8n.io
3. Join the n8n community forum

**Happy Investing! 🏡💰**
