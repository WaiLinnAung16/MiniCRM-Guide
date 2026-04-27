# Mini CRM - Complete Production Setup

## 🎯 What's Built

| Component | Status | File |
|-----------|--------|------|
| **1. Lead Capture** | ✅ Ready | Gmail + Sheets integration |
| **2. CRM Database** | ✅ Ready | Google Sheets CRM |
| **3. AI Scoring** | ✅ Ready | Automatic Hot/Warm/Cold |
| **4. Follow-Up Reminders** | ✅ Ready | Telegram alerts every 4 hours |
| **5. Daily Summary** | ✅ Ready | 9 AM Telegram report |

## 📊 Your Sheets

- **CRM Database:** `YOUR-GOOGLE-SHEET-DATABASE-ID`
- **Contact Form:** `YOUR-GOOGLE-FORM-SHEET-ID`

## 🚀 Quick Start

### 1. Set Up Google Sheets Integration

**A. Install Dependencies**
```bash
cd ~/.openclaw/skills/mini-crm/integrations/sheets
npm install googleapis fs-extra
```

**B. Authenticate**
```bash
node auth.js
```
Follow the OAuth flow, paste code to `auth_code.txt`, run again.

**C. Configure**
Edit `config.json` - already done with your Sheet IDs.

### 2. Add Apps Script to Contact Form

1. Open your Contact Form sheet
2. Extensions → Apps Script
3. Paste code from `apps_script.js`
4. Update `crmSheetId` (already set to your CRM sheet)
5. Deploy → Web App → Execute as Me → Anyone
6. Add trigger: onFormSubmit → From spreadsheet → On form submit

### 3. Test Form Submission

Add a row to your Contact Form sheet:
| Name | Email | Company | Message | Date |
|------|-------|---------|---------|------|
| Test Lead | test@company.com | Acme Inc | Looking for demo, $20K budget | today |

Check your CRM sheet - it should auto-appear with scoring!

### 4. Set Up Cron Jobs

**Daily Summary (9:00 AM):**
```bash
openclaw cron add --name "CRM Daily Summary" \
  --schedule "0 9 * * *" \
  --timezone "Asia/Rangoon" \
  --command "cd ~/.openclaw/skills/mini-crm/crm-db && node daily-summary.js"
```

**Follow-Up Reminders (every 4 hours):**
```bash
openclaw cron add --name "CRM Follow-up Check" \
  --schedule "0 */4 * * *" \
  --timezone "Asia/Rangoon" \
  --command "cd ~/.openclaw/skills/mini-crm/crm-db && node follow-up-reminders.js"
```

**Gmail Poll (every 15 minutes):**
```bash
openclaw cron add --name "Gmail Lead Poll" \
  --schedule "*/15 * * * *" \
  --command "cd ~/.openclaw/skills/mini-crm/integrations/gmail && node poll.js"
```

## 📋 Test Plan

### Send 5 Test Emails

1. **Big Company Demo (Hot)**
   - Subject: "Enterprise Demo Request - Acme Corp"
   - Body: "I'm the CTO at Acme Corp (500+ employees). We need an urgent demo of your solution. Budget is $50K. Can we meet this week?"

2. **Small Business Pricing (Warm)**
   - Subject: "Pricing question"
   - Body: "Hi, I run a small consulting business. Interested in your services - what are your rates?"

3. **Student Inquiry (Cold)**
   - Subject: "School project question"
   - Body: "I'm a student doing research on CRMs. Can you explain how your product works?"

4. **Partnership Request (Hot)**
   - Subject: "Partnership Opportunity"
   - Body: "I'm the founder of TechStart and we're looking for integration partners. Have budget for joint marketing. Let's discuss."

5. **Follow-Up (Warm/Hot)**
   - Subject: "Re: Our conversation last week"
   - Body: "Following up on our discussion. Still interested in moving forward. What's the next step?"

### Add 3 Form Entries

1. Startup CEO urgent: "Need help ASAP, $30K budget, decision maker"
2. General inquiry: "Interested in learning more about your services"
3. Budget mentioned: "Looking for solution around $10K, timeline 1 month"

## 🔄 How It Works

### Email Flow
```
Email arrives → Gmail label "CRM-Leads" → Poll script (15 min) 
→ Pending leads → Manual trigger or auto-process 
→ Lead Intake → Scoring → CRM Sheet → Telegram notification
```

### Form Flow
```
Form submitted → Apps Script → Auto-score → CRM Sheet 
→ Telegram notification (instant)
```

### Daily Summary Flow
```
9:00 AM → Read CRM data → Calculate stats 
→ Generate summary → Send Telegram
```

### Follow-Up Reminders
```
Every 4 hours → Check overdue follow-ups 
→ Check 48h+ new leads → Send Telegram alert
```

## 📱 Telegram Notifications

You'll receive:
- ✅ **Instant alerts** when new leads arrive (Hot leads highlighted)
- 📊 **Daily summary** at 9 AM with full pipeline stats
- ⏰ **Follow-up reminders** every 4 hours for overdue items

## 🛠️ Troubleshooting

| Issue | Solution |
|-------|----------|
| "Not authenticated" | Run `node auth.js` in sheets/ and gmail/ folders |
| Form not triggering | Check Apps Script deployment and trigger settings |
| No Telegram messages | Verify bot token and chat ID in `~/.openclaw/openclaw.json` |
| Leads not scoring | Check CRM sheet headers match expected columns |
| Gmail not finding emails | Verify "CRM-Leads" label exists and is applied |

## 📁 File Structure

```
~/.openclaw/skills/mini-crm/
├── crm-db/
│   ├── leads.json              # Local backup
│   ├── daily-summary.js        # 9 AM summary script
│   ├── follow-up-reminders.js  # Check overdue leads
│   └── read_db.js              # Read local DB
├── integrations/
│   ├── gmail/
│   │   ├── poll.js             # Gmail polling
│   │   ├── auth.js             # OAuth setup
│   │   ├── config.json         # Gmail config
│   │   └── pending_leads.json  # Temp storage
│   └── sheets/
│       ├── auth.js             # OAuth setup
│       ├── process.js          # Form processing
│       ├── apps_script.js      # Copy to Apps Script
│       └── config.json         # Sheet IDs
├── agents/
│   ├── lead-intake/SKILL.md    # Agent prompt
│   ├── scoring/SKILL.md        # Agent prompt
│   └── crm-manager/SKILL.md    # Agent prompt
└── README.md                   # This file
```

## 🎉 Next Steps

1. ✅ Run auth.js for Sheets
2. ✅ Add Apps Script to Contact Form sheet
3. ✅ Test with 1 form submission
4. ✅ Set up cron jobs
5. ✅ Send test emails
6. ✅ Verify Telegram notifications

**Your complete AI-powered CRM is ready!** 🚀
