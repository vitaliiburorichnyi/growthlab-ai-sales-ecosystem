# AI Voice Sales Ecosystem — GrowthLab

Automated lead qualification system using AI voice agent, n8n workflows, and Claude AI. From first contact to CEO weekly report — fully automated.

---

## Overview

Three interconnected workflows that form a closed AI sales loop:

```
New Lead → Voice Agent Call (2 min) → BANT Score → Zoho CRM
                                            ↓
                                   Daily Dialog Analysis → Google Sheets
                                            ↓
                                   Weekly CEO Report → Telegram
```

---

## Workflows

| File | Purpose | Trigger |
|---|---|---|
| `WF1a_Call_Trigger` | Receives new lead from Zoho → triggers Happ.tools call | Zoho CRM Webhook |
| `WF1b_Postcall_Handler` | Receives call results → scores lead → updates CRM | Happ.tools Webhook |
| `WF1c_Lead_Lookup` | Returns lead name to voice agent before call starts | Happ.tools INIT |
| `WF2_Dialog_Analysis` | Analyzes all calls → writes quality report to Google Sheets | Daily 9:00 |
| `WF3_CEO_Report` | Reads CRM + Sheets → generates AI report → sends to Telegram | Friday 17:00 |

---

## Tech Stack

- **n8n** (self-hosted) — workflow automation
- **Happ.tools** — AI voice agent (GPT-5-mini, English, voice: Arabella)
- **Anthropic Claude Sonnet** — BANT scoring + dialog analysis + CEO report
- **Zoho CRM** — lead management (custom fields: AI_Score, AI_Context, Session_ID)
- **Google Sheets** — dialog quality database
- **Telegram Bot** — CEO weekly report delivery

---

## Prerequisites

- n8n self-hosted instance
- Happ.tools account with voice assistant configured
- Anthropic API key
- Zoho CRM account with custom fields created
- Google Sheets document with dialog analysis columns
- Telegram bot token

---

## Setup

### 1. Zoho CRM — Create Custom Fields

Go to `Setup → Customization → Modules → Leads → Fields` and create:

| Label | Type | API Name |
|---|---|---|
| AI Score | Integer | `AI_score` |
| AI Context | Multi-line Text | `AI_Context` |
| Session ID | Single-line Text | `Session_ID` |

### 2. Happ.tools — Configure Voice Assistant

Create assistant with:
- Type: Voice
- Model: GPT-5-mini
- Voice: Arabella (English)
- Eagerness: Normal
- Turn after silence: 6 sec

Add two tools:
- **INIT tool** (`load_lead`) — fires before call, fetches lead name from n8n
- **Postcall tool** (`save_result`) — fires after call, sends BANT data to n8n

### 3. Google Sheets — Create Analysis Sheet

Create a sheet named `Dialog Analysis` with columns:

```
id | client | phone | agent_errors | client_problem | successful_pattern | ai_score | date
```

### 4. n8n — Import Workflows

1. Import all 5 JSON files from `/workflows/` folder
2. Re-create credentials:
   - Happ Header Auth (`X-Access-Token: your_api_key`)
   - Zoho CRM OAuth2
   - Anthropic API
   - Google Sheets OAuth2
   - Telegram Bot
3. Update placeholders in each workflow (see table below)
4. Activate all workflows

### 5. Zoho CRM — Configure Webhook

Go to `Setup → Automation → Webhooks → New Webhook`:
- URL: `https://your-n8n-instance.com/webhook/new-lead`
- Module: Leads
- Parameters: Phone, Lead Name, Company, id

Create Workflow Rule:
- Module: Leads
- When: Record is Created
- Condition: Session_ID is empty
- Action: fire webhook

### 6. Happ.tools — Update Tool URLs

In your assistant tools:
- `load_lead` URL → `https://your-n8n-instance.com/webhook/lead-lookup`
- `save_result` URL → `https://your-n8n-instance.com/webhook/qualification-done`

---

## Placeholders Reference

Replace these in the JSON files before importing:

| Placeholder | Where to find it |
|---|---|
| `YOUR_HAPP_ASSISTANT_ID` | Happ.tools → assistant URL |
| `YOUR_HAPP_CREDENTIAL_ID` | n8n credential ID after creation |
| `YOUR_ZOHO_CREDENTIAL_ID` | n8n credential ID after creation |
| `YOUR_ANTHROPIC_CREDENTIAL_ID` | n8n credential ID after creation |
| `YOUR_GOOGLE_SHEETS_ID` | Google Sheets URL |
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | n8n credential ID after creation |
| `YOUR_TELEGRAM_CHAT_ID` | Telegram → getUpdates API |
| `YOUR_TELEGRAM_CREDENTIAL_ID` | n8n credential ID after creation |
| `YOUR_N8N_INSTANCE_ID` | n8n Settings → Instance |

> Webhook IDs and Workflow IDs are auto-generated on import — no action needed.

---

## BANT Scoring Rules

| Category | 25 pts | 20 pts | 15 pts | 10 pts | 0 pts |
|---|---|---|---|---|---|
| Budget | $3K+/mo | $2.5K | $1.5–2.5K | Below $1.5K | Not stated |
| Authority | Owner/CEO | CMO/Head | Influences | Gathers info | Unknown |
| Need | Urgent problem | Clear problem | Mild need | Vague desire | Not stated |
| Timeline | This week | This month | Next month | 2–3 months | Just looking |

**Total score: 0–100.** Leads above 70 are considered hot.

---

## Results

| Metric | Before | After |
|---|---|---|
| Qualification time | 20 min/lead | 2 min/lead |
| Lead response time | Hours | 2 minutes |
| AI Score per lead | None | 0–100 automatic |
| CEO report | Monthly manual | Weekly automated |
| Scalability | 100 leads/4 managers | 1000+ same team |

---

## License

MIT — free to use, modify, and distribute.
