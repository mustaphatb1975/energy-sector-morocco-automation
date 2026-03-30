# 💧 Water & Energy Pump Automation — Morocco 🇲🇦

<div align="center">

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-Data_Layer-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![WhatsApp](https://img.shields.io/badge/WhatsApp-Real--time_Alerts-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)
![Status](https://img.shields.io/badge/Status-Production-brightgreen?style=for-the-badge)
![Sector](https://img.shields.io/badge/Sector-Energy_%26_Water-0055A4?style=for-the-badge)

**Production automation system monitoring 7 industrial pumps in Morocco's energy sector.**  
Manual hours → **Zero** · Human errors → **Near-Zero** · Monitoring → **Real-Time**

</div>

---

## 📋 Overview

This n8n workflow automates the full data lifecycle for 7 industrial water and energy pumps — from daily index collection to anomaly detection and instant supervisor alerts via WhatsApp.

The system was built for an energy sector facility in Morocco 🇲🇦 and replaced a fully manual reporting process, eliminating transcription errors and enabling real-time operational visibility.

> *"This system completely transformed how we track pump performance. What used to take hours of manual work now happens automatically every day."*  
> — **Operations Supervisor**, Energy Sector — Morocco 🇲🇦

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    TRIGGER LAYER                                  │
│  ┌─────────────────────┐     ┌──────────────────────────────┐    │
│  │  Schedule Trigger   │     │  WhatsApp Webhook (POST)     │    │
│  │  Daily @ 09:50 AM   │     │  Receives daily pump indexes │    │
│  └────────┬────────────┘     └──────────────┬───────────────┘    │
│           │                                 │                     │
│           ▼                                 ▼                     │
│  ┌─────────────────────┐     ┌──────────────────────────────┐    │
│  │  Morning Reminder   │     │  Month Router (Switch)       │    │
│  │  via WhatsApp API   │     │  Jan / Feb / Mar / Apr…      │    │
│  └─────────────────────┘     └──────────────┬───────────────┘    │
└─────────────────────────────────────────────┼────────────────────┘
                                              │
┌─────────────────────────────────────────────▼────────────────────┐
│                  PROCESSING LAYER                                 │
│                                                                   │
│  ┌──────────────────┐    ┌───────────────────┐                   │
│  │ Parse WhatsApp   │    │ Fetch Yesterday's │                   │
│  │ Message → JSON   │    │ Row (Google Sheet)│                   │
│  │ 4 index values   │    │ Pompe 1 & 2 data  │                   │
│  └────────┬─────────┘    └─────────┬─────────┘                   │
│           └──────────┬─────────────┘                             │
│                      ▼                                            │
│           ┌──────────────────────┐                               │
│           │   Merge & Calculate  │                               │
│           │  • difference(h)     │                               │
│           │  • difference(h)_2   │                               │
│           │  • difference(m3)    │                               │
│           │  • difference(kW)    │                               │
│           │  • Total d'heure     │                               │
│           └──────────┬───────────┘                               │
└──────────────────────┼────────────────────────────────────────────┘
                       │
┌──────────────────────▼────────────────────────────────────────────┐
│                  VALIDATION LAYER                                  │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  Anomaly Detection Rules (JavaScript)                       │  │
│  │                                                             │  │
│  │  Metric          Min     Max     Action on Violation        │  │
│  │  ─────────────   ─────   ─────   ──────────────────────     │  │
│  │  difference(m3)  400     560     → Flag as "wrong"          │  │
│  │  difference(h)   2.0     2.8     → Flag as "wrong"          │  │
│  │  difference(h)_2 1.5     2.3     → Flag as "wrong"          │  │
│  │  différence(kW)  50      70      → Flag as "wrong"          │  │
│  │  Total d'heure   3.5     5.1     → Flag as "wrong"          │  │
│  └─────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────┘
                       │
        ┌──────────────┴────────────────┐
        ▼                               ▼
┌───────────────────┐        ┌──────────────────────┐
│   ✅ ALL OK        │        │  ⚠️ ANOMALY DETECTED   │
│                   │        │                       │
│  Append/Update    │        │  WhatsApp Alert →     │
│  Google Sheet     │        │  Operations           │
│  (monthly tab)    │        │  Supervisor           │
└───────────────────┘        └──────────────────────┘
```

---

## ⚙️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Automation Engine** | n8n (self-hosted) | Core workflow orchestration |
| **Messaging API** | Evolution API (WhatsApp) | Daily reminders + anomaly alerts |
| **Data Storage** | Google Sheets | Monthly operational data log |
| **Scripting** | JavaScript (n8n Code nodes) | Calculation, parsing, validation |
| **Trigger** | Schedule + Webhook | Dual-trigger architecture |

---

## 📊 Key Metrics Tracked

```
Pompe 1    →  Index(h)    — Running hours counter (Pump 1)
Pompe 2    →  Index(h)_2  — Running hours counter (Pump 2)
Débitmètre →  Index(m3)   — Water volume pumped (cubic meters)
Centrale   →  Index(kW)   — Energy consumed (kilowatts)
```

**Calculated Daily:**
- `difference(h)` — Hours delta Pump 1
- `difference(h)_2` — Hours delta Pump 2
- `difference(m3)` — Volume delta (water pumped today)
- `difference(kW)` — Energy delta
- `Total d'heure` — Combined pump operation hours

---

## 🔁 Workflow Nodes (30 Total)

| # | Node | Type | Function |
|---|---|---|---|
| 1 | Schedule Trigger | Trigger | Fires daily at 09:50 AM |
| 2 | Webhook | Trigger | Receives WhatsApp pump data |
| 3 | select the exact mounth | Code | Extracts month from timestamp |
| 4 | Switch2 | Router | Routes to correct monthly sheet |
| 5 | output 4 itmes1 | Code | Parses 4 pump index values from message |
| 6 | Get row(s) in sheet1 | Google Sheets | Fetches yesterday's baseline row |
| 7 | If1 | Condition | Validates row has existing data |
| 8 | Limit1 | Limiter | Takes last item for merge |
| 9 | Merge | Merger | Combines today + yesterday data |
| 10 | calcule the numbers | Code | Computes all daily differences |
| 11 | calcule the total hours | Code | Sums total pump hours |
| 12 | If3 | Condition | Checks for negative anomalies |
| 13 | Get row(s) in sheet5 | Google Sheets | Fetches current day's row |
| 14 | If6 | Condition | Row existence check |
| 15 | Append or update row | Google Sheets | Saves computed data to sheet |
| 16 | Update row in sheet | Google Sheets | Updates total hours |
| 17 | Get row(s) in sheet | Google Sheets | Re-reads updated row |
| 18 | If5 | Condition | Non-empty check before verify |
| 19 | Limit | Limiter | Takes last item |
| 20 | verefy the rows1 | Code | Runs all 5 anomaly threshold checks |
| 21 | wrong output1 | Code | Collects flagged field names |
| 22 | Enviar texto | Evolution API | Sends WhatsApp alert (path 1) |
| 23 | Enviar texto3 | Evolution API | Sends WhatsApp alert (path 2) |
| 24 | Enviar texto2 | Evolution API | Sends morning reminder |
| 25 | If2 | Condition | Detects month-end transition |
| 26 | Edit Fields | Set | Updates day field for new month |
| 27 | Limit5 | Limiter | Pipeline control |
| 28 | Sticky Note | Note | "send message every morning" |
| 29 | Sticky Note1 | Note | "account the today input" |
| 30 | Sticky Note2 | Note | "append first month data" |

---

## 🚀 Setup & Deployment

### Prerequisites

- n8n instance (self-hosted or cloud)
- Google Sheets API credentials (OAuth2)
- Evolution API instance (WhatsApp Business)

### Step 1 — Configure Credentials

In n8n, create the following credentials:

```
1. Google Sheets OAuth2 API
   Name: Google Sheets account
   Scope: https://www.googleapis.com/auth/spreadsheets

2. Evolution API
   Name: Evolution account
   Base URL: YOUR_EVOLUTION_API_URL
   API Key: YOUR_EVOLUTION_API_KEY
```

### Step 2 — Prepare Google Sheet

Create a Google Sheet with the following structure:

```
Row 1: Title row (optional)
Row 2: Headers → Jours | index(h) | difference(h) | Index(h)_2 | difference(h)_2 | Total d'heure | Index(m3) | difference(m3) | Index(kW) | différence(kW)
Row 3+: Daily data (one row per day)
```

### Step 3 — Import Workflow

1. Open n8n → **Workflows** → **Import from File**
2. Select `workflow.json`
3. Update all `YOUR_*` placeholders:

| Placeholder | Replace With |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | Your Google Sheet ID from the URL |
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | ID from n8n credentials page |
| `YOUR_EVOLUTION_API_CREDENTIAL_ID` | ID from n8n credentials page |
| `YOUR_WHATSAPP_INSTANCE` | Your Evolution API instance name |
| `YOUR_WHATSAPP_NUMBER@s.whatsapp.net` | Target WhatsApp number |
| `YOUR_WEBHOOK_PATH` | Custom path for your webhook endpoint |
| `YOUR_WEBHOOK_ID` | Auto-generated by n8n after import |

### Step 4 — Activate

```
n8n → Workflow → Toggle Active → ✅ Active
```

---

## 📁 Repository Structure

```
energy-sector-morocco-automation/
├── README.md              ← This file
├── workflow.json          ← Clean n8n workflow (no secrets)
└── .gitignore             ← Protects sensitive files
```

---

## 🔒 Security Notes

All sensitive data has been removed from `workflow.json`:

- ✅ Google Sheet IDs → `YOUR_GOOGLE_SHEET_ID`
- ✅ API Credential IDs → `YOUR_*_CREDENTIAL_ID`
- ✅ WhatsApp Instance Name → `YOUR_WHATSAPP_INSTANCE`
- ✅ Webhook Path & ID → `YOUR_WEBHOOK_PATH` / `YOUR_WEBHOOK_ID`
- ✅ Phone Numbers → `YOUR_WHATSAPP_NUMBER@s.whatsapp.net`
- ✅ Real Names → Generic role identifiers

---

## 📈 Results

| Metric | Before | After |
|---|---|---|
| Daily manual data entry | ~2 hours | **0 minutes** |
| Human transcription errors | Frequent | **Near-zero** |
| Anomaly detection time | Hours (if noticed) | **< 1 minute** |
| Supervisor notification | Phone call | **Automated WhatsApp** |
| Monthly reporting | Manual Excel | **Auto-updated Sheet** |

---

## 👤 Author

**Mustapha Taleb** — AI Automation Engineer  
📍 Agadir, Morocco 🇲🇦  
✉️ [mustaphatb1975@gmail.com](mailto:mustaphatb1975@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/talebmustapha/)  
🐙 [GitHub](https://github.com/mustaphatb1975)

---

<div align="center">

*Built with precision for Morocco's energy sector — where automation meets operational excellence.*

</div>
