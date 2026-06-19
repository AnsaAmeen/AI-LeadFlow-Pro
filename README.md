# 🤖 AI-LeadFlow-Pro
### Intelligent Lead Nurturing & Follow-up Automation

> A fully automated, AI-powered lead nurturing system built on n8n that captures leads from multiple sources, scores them with GPT-4, sends personalized outreach via WhatsApp & Email, and manages follow-ups — all without any human involvement.

---

## 📌 Overview

**AI-LeadFlow-Pro** is a 26-node n8n workflow designed for real estate agencies, marketing teams, and businesses that want to automate their entire lead response and nurturing pipeline. It captures leads from Facebook, WhatsApp, website forms, and Google Ads — then uses GPT-4 to score, enrich, and communicate with each lead automatically.

---

## 🎯 What This Workflow Does

| Stage | Action |
|-------|--------|
| **Capture** | Collects leads from 4 sources simultaneously |
| **Deduplicate** | Checks HubSpot CRM to avoid duplicate contacts |
| **Score** | GPT-4 scores each lead as Hot / Warm / Cold |
| **Outreach** | Sends personalized WhatsApp + Email instantly |
| **Notify** | Alerts your team on Slack with full lead profile |
| **Follow-up** | Auto follow-ups if no reply after 2 days |
| **Route** | Routes interested leads to Google Calendar booking |
| **Archive** | Archives unresponsive leads after 3 follow-ups |

---

## 🏗️ Workflow Architecture

```
Lead Sources (4)
│
├── Facebook Lead Ads
├── Website Form Webhook
├── WhatsApp Inbound (Twilio)
└── Google Ads Lead Form
        │
        ▼
   Merge All Sources
        │
        ▼
   Normalize Fields
        │
        ▼
   HubSpot — Check Duplicate
        │
        ▼
   IF — New Lead?
   ├── NO  → Stop (duplicate)
   └── YES →
            │
            ▼
       OpenAI — Score & Enrich (GPT-4)
            │
            ▼
       Parse AI Score
            │
            ▼
       HubSpot — Create Contact
            │
            ▼
       OpenAI — Write First Message
            │
        ┌───┴───┐
        ▼       ▼
  Send WhatsApp  Send Email (Gmail)
        │
        ▼
  Extract Message Text
        │
    ┌───┴───┐
    ▼       ▼
Slack Notify  Wait 2 Days
              │
              ▼
        HubSpot — Check Reply Status
              │
              ▼
        IF — Lead Replied?
        ├── YES →
        │        OpenAI — Analyze Reply
        │        Parse Reply Analysis
        │        Switch — Route by Intent
        │        ├── Interested → Google Calendar — Create Viewing
        │        │               Send Reply WhatsApp
        │        │               HubSpot — Mark as Replied
        │        └── Not Ready  → HubSpot — Mark as Replied
        │
        └── NO →
                 IF — Under 3 Follow-ups?
                 ├── YES →
                 │        OpenAI — Write Follow-up
                 │        Send Follow-up WhatsApp
                 │        HubSpot — Increment Follow-up Count
                 │        [loops back to Wait 2 Days]
                 │
                 └── NO →
                          HubSpot — Archive No Response
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **n8n** | Workflow orchestration |
| **OpenAI GPT-4** | Lead scoring, message writing, reply analysis |
| **HubSpot CRM** | Contact management, deduplication, status tracking |
| **Twilio** | WhatsApp inbound & outbound messaging |
| **Gmail API** | Email outreach |
| **Slack API** | Team notifications |
| **Google Calendar** | Appointment/viewing booking |
| **Facebook Lead Ads** | Lead capture from Facebook campaigns |
| **Webhook** | Website form & Google Ads lead capture |

---

## ⚙️ Prerequisites

Before importing this workflow, make sure you have:

- [ ] n8n instance running (cloud or self-hosted)
- [ ] OpenAI API key (GPT-4 access)
- [ ] HubSpot account with Private App created
- [ ] Twilio account with WhatsApp sandbox enabled
- [ ] Gmail account with Google OAuth2 configured
- [ ] Slack workspace with a bot token
- [ ] Google Calendar API enabled
- [ ] Facebook Developer App (for Lead Ads)

---

## 🚀 Setup Guide

### Step 1 — Import Workflow

1. Download `AI-LeadFlow-Pro.json` from this repo
2. Open your n8n instance
3. Click **"Import from file"**
4. Select the downloaded JSON file

### Step 2 — Configure Credentials

Go to **Settings → Credentials** in n8n and add the following:

#### OpenAI
```
API Key: sk-xxxxxxxxxxxxxxxxxxxx
```

#### HubSpot
1. Go to HubSpot → Settings → Integrations → Private Apps
2. Create new app with these scopes:
   - `crm.objects.contacts.read`
   - `crm.objects.contacts.write`
   - `crm.objects.deals.read`
   - `crm.objects.deals.write`
   - `timeline`
3. Copy the Access Token and paste in n8n

#### Twilio (WhatsApp)
```
Account SID: ACxxxxxxxxxxxxxxxx
Auth Token:  xxxxxxxxxxxxxxxx
WhatsApp Number: whatsapp:+14155238886
```

#### Gmail
1. Go to Google Cloud Console
2. Enable Gmail API
3. Create OAuth2 credentials
4. Add redirect URI: `https://oauth.n8n.cloud/oauth2/callback`

#### Slack
```
Bot Token: xoxb-xxxxxxxxxxxx
Channel ID: #leads (or your channel)
```

#### Google Calendar
```
Use same OAuth2 credentials as Gmail
Calendar ID: your-email@gmail.com
```

### Step 3 — Configure Webhook

1. Click on **Webhook1** node
2. Copy the **Production URL**
3. Add this URL to your website form action
4. For testing, use the **Test URL** with Postman

### Step 4 — Test with Postman

Send a POST request to your webhook Test URL:

```json
{
  "name": "Test Lead",
  "email": "testlead@example.com",
  "phone": "+1234567890",
  "source": "Website Form",
  "message": "I am interested in your services"
}
```

**Expected Response:**
```json
{
  "message": "Workflow was started"
}
```

### Step 5 — Activate Workflow

1. Fix any credential warnings (orange triangles)
2. Click **"Activate"** toggle in top right
3. Workflow is now live ✅

---

## 📊 Workflow Nodes — Complete List

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Facebook Lead Ads | Trigger | Captures Facebook leads |
| 2 | Website Form Webhook | Trigger | Captures website form leads |
| 3 | WhatsApp Inbound | Trigger | Captures WhatsApp messages |
| 4 | Webhook1 | Trigger | Google Ads / manual testing |
| 5 | Merge All Sources | Core | Combines all lead streams |
| 6 | Normalize Fields | Core | Standardizes field names |
| 7 | HubSpot — Check Duplicate | HubSpot | Searches existing contacts |
| 8 | IF — New Lead? | Logic | Stops duplicates |
| 9 | OpenAI — Score & Enrich | AI | GPT-4 lead scoring |
| 10 | Parse AI Score | Core | Extracts JSON score data |
| 11 | HubSpot — Create Contact | HubSpot | Adds new contact to CRM |
| 12 | OpenAI — Write First Message | AI | GPT-4 personalized outreach |
| 13 | Send WhatsApp | Twilio | WhatsApp message delivery |
| 14 | Send Email (Gmail) | Gmail | Email message delivery |
| 15 | Extract Message Text | Core | Saves message for records |
| 16 | Slack — Notify Agent | Slack | Team alert with lead profile |
| 17 | Wait 2 Days | Core | Pause before follow-up check |
| 18 | HubSpot — Check Reply Status | HubSpot | Checks if lead responded |
| 19 | IF — Lead Replied? | Logic | Branches replied vs silent |
| 20 | OpenAI — Analyze Reply | AI | GPT-4 intent detection |
| 21 | Parse Reply Analysis | Core | Extracts intent from JSON |
| 22 | Switch — Route by Intent | Logic | Routes by interest level |
| 23 | Google Calendar — Create Viewing | Calendar | Books appointment slot |
| 24 | Send Reply WhatsApp | Twilio | Sends booking confirmation |
| 25 | HubSpot — Mark as Replied | HubSpot | Updates contact status |
| 26 | IF — Under 3 Follow-ups? | Logic | Controls follow-up loop |
| 27 | OpenAI — Write Follow-up | AI | GPT-4 follow-up message |
| 28 | Send Follow-up WhatsApp | Twilio | Sends follow-up message |
| 29 | HubSpot — Increment Follow-up Count | HubSpot | Updates follow-up counter |
| 30 | HubSpot — Archive No Response | HubSpot | Archives cold leads |

---

## 🔁 Follow-up Logic

```
Day 0  → First outreach (WhatsApp + Email)
Day 2  → Check reply → No reply → Follow-up #1
Day 4  → Check reply → No reply → Follow-up #2
Day 6  → Check reply → No reply → Follow-up #3
Day 8  → No reply → Archive as Unqualified
```

---

## 📈 Results You Can Expect

- ⚡ **< 3 seconds** response time to every new lead
- 📞 **24/7** lead capture and outreach — no missed leads
- 🤖 **60%** reduction in manual follow-up work
- 📊 **100%** of leads logged and scored in HubSpot CRM
- 🔄 **3x** follow-up attempts before archiving

---

## 🧪 Testing Without Real Integrations

For portfolio demo purposes, you can test with just the Webhook node active:

1. Deactivate Facebook Lead Ads, WhatsApp, and Google Ads nodes
2. Use **Webhook1** as your only trigger
3. Send test data via Postman
4. Comment out or bypass WhatsApp/Gmail nodes temporarily
5. Check HubSpot and OpenAI nodes output in n8n

---

## 📁 Repository Structure

```
AI-LeadFlow-Pro/
│
├── README.md                  ← This file
├── AI-LeadFlow-Pro.json       ← n8n workflow import file
├── screenshots/
│   ├── workflow-overview.png  ← Full workflow screenshot
│   ├── lead-capture.png       ← Stage 1 nodes
│   └── follow-up-logic.png    ← Follow-up flow
└── docs/
    └── setup-guide.md         ← Detailed setup instructions
```

---

## 👩‍💻 Built By

**Ansa Ameen**
AI Developer & Automation Specialist
[LinkedIn](https://linkedin.com/in/ansa-ameen) | Softsynic Technologies

Built with n8n · OpenAI GPT-4 · HubSpot · Twilio · Slack · Google APIs
