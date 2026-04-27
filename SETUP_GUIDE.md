# Mini CRM Setup Guide

## Quick Start

### 1. Install the Skill

```bash
# Copy the skill to your OpenClaw skills directory
cp -r ~/.openclaw/skills/mini-crm ~/skills/mini-crm

# Or create symlink
ln -s ~/.openclaw/skills/mini-crm ~/skills/mini-crm
```

### 2. Configure Telegram (for Notifications)

To receive daily summaries and alerts:

1. Open Telegram and message @BotFather
2. Create a new bot: `/newbot`
3. Save the bot token
4. Get your chat ID by messaging @userinfobot
5. Configure OpenClaw:

```bash
openclaw config set telegram.bot.token YOUR_BOT_TOKEN
openclaw config set telegram.chat.id YOUR_CHAT_ID
```

### 3. Set Up Cron Jobs

**Daily Summary (9:00 AM):**
```bash
openclaw cron add --name "CRM Daily Summary" \
  --schedule "0 9 * * *" \
  --agent crm-manager \
  --message "Generate daily CRM summary and send to Telegram"
```

**Follow-up Reminders (every 4 hours during business hours):**
```bash
openclaw cron add --name "CRM Follow-up Reminders" \
  --schedule "0 */4 * * *" \
  --agent crm-manager \
  --message "Check for overdue follow-ups and send reminders"
```

### 4. Connect Email (Optional)

For automatic lead capture from email:

1. Set up email forwarding or IMAP polling
2. Create webhook endpoint
3. Forward emails to OpenClaw webhook

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CRM MANAGER AGENT (Lexi)                 │
│         ┌──────────────┬──────────────┬──────────────┐     │
│         │  Lead Intake │   Scoring    │  Reminders   │     │
│         │    Agent     │    Agent     │   & Reports  │     │
│         └──────┬───────┘ └──────┬─────┘ └──────┬──────┘     │
│                │              │              │              │
└────────────────┼──────────────┼──────────────┼──────────────┘
                 │              │              │
              ┌──▼──────────────▼──────────────▼──┐
              │         CRM DATABASE              │
              │  ~/.openclaw/skills/mini-crm/     │
              │        crm-db/leads.json          │
              └───────────────────────────────────┘
```

## How It Works

### Adding a Lead (Manual)

1. User provides raw lead data
2. Lexi spawns **Lead Intake Agent**
3. Lead Intake cleans and standardizes the data
4. Lexi spawns **Scoring Agent** with standardized lead
5. Scoring Agent classifies as Hot/Warm/Cold
6. Lexi saves to database and confirms

Example:
```
You: "Add this lead: John from john@acme.com said they need help with onboarding, budget is $15K"

Lexi: 
1. Spawns Lead Intake with the raw data
2. Receives: { name: "John", email: "john@acme.com", ... }
3. Spawns Scoring Agent
4. Receives: { status: "Hot", value_estimate: 15000, ... }
5. Saves to database
6. Replies: "✅ Lead saved! Acme Inc - HOT ($15K) - Recommended follow-up: within 24 hours"
```

### Daily Summary

Every morning at 9 AM:
1. CRM Manager reads all leads
2. Calculates statistics
3. Identifies urgent follow-ups
4. Sends formatted summary to Telegram

### Pipeline Queries

Ask Lexi questions like:
- "Who should I talk to first?" → Hot leads, sorted by value
- "What deals are we about to lose?" → Hot leads with no contact 48h+
- "Show me all warm leads" → Filtered list
- "What's my pipeline value?" → Sum of all open deals

## File Structure

```
mini-crm/
├── SKILL.md                           # Main skill documentation
├── agents/
│   ├── lead-intake/
│   │   └── SKILL.md                   # Lead Intake Agent
│   ├── scoring/
│   │   └── SKILL.md                   # Scoring Agent
│   └── crm-manager/
│       └── SKILL.md                   # CRM Manager Agent (Lexi)
├── crm-db/
│   ├── leads.json                     # Database file
│   ├── read_db.js                     # Read database script
│   ├── write_db.js                    # Write database script
│   └── query_db.js                    # Query/filter script
└── references/
    └── setup-guide.md                 # This file
```

## Integration Examples

### Google Sheets Webhook

Create a Google Apps Script:

```javascript
function onFormSubmit(e) {
  const formData = e.namedValues;
  
  const payload = {
    name: formData['Name'][0],
    email: formData['Email'][0],
    message: formData['Message'][0],
    source: 'webform',
    timestamp: new Date().toISOString()
  };
  
  UrlFetchApp.fetch('YOUR_OPENCLAW_WEBHOOK_URL', {
    method: 'POST',
    payload: JSON.stringify(payload),
    headers: { 'Content-Type': 'application/json' }
  });
}
```

### Google Sheets Integration

#### Option A: Inbound (Form submissions → OpenClaw)

For capturing leads from Google Forms into the Mini CRM.

```javascript
function onFormSubmit(e) {
  const formData = e.namedValues;
  
  const payload = {
    name: formData['Name'][0],
    email: formData['Email'][0],
    message: formData['Message'][0],
    source: 'webform',
    timestamp: new Date().toISOString()
  };
  
  UrlFetchApp.fetch('YOUR_OPENCLAW_WEBHOOK_URL', {
    method: 'POST',
    payload: JSON.stringify(payload),
    headers: { 'Content-Type': 'application/json' }
  });
}
```

#### Option B: Outbound (Mini CRM → Google Sheet)

For syncing your leads database to a Google Sheet for easy viewing and sharing.

**Step 1: Set up Google Authentication**

Choose one of these methods:

**Method 1: Service Account (Recommended for automation)**
1. Go to https://console.cloud.google.com/
2. Create a new project or select existing
3. Enable the Google Sheets API
4. Go to Credentials → Create Credentials → Service Account
5. Create key → JSON → Download the file
6. Run: `node crm-db/setup-google-auth.js`
7. When prompted, enter the path to your downloaded JSON key file
8. Share your Google Sheet with the service account email (found in the JSON file)

**Method 2: OAuth (For personal use)**
1. Run: `node crm-db/setup-google-auth.js`
2. Choose OAuth option
3. Follow the prompts to authorize

**Step 2: Configure Spreadsheet**

Edit `crm-db/google-sheets-config.json`:

```json
{
  "spreadsheetId": "YOUR_SPREADSHEET_ID",
  "sheetName": "Leads",
  "autoSync": true,
  "syncIntervalMinutes": 60
}
```

To find your Spreadsheet ID, look at the URL:
`https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`

**Step 3: Test Sync**

```bash
cd ~/.openclaw/skills/mini-crm
node crm-db/sync_to_sheets.js
```

You should see: `✅ Successfully synced X leads to Google Sheets`

The sync will:
- Create a "Leads" sheet if it doesn't exist
- Format the header row with gray background and bold text
- Sync all leads with their status, scores, and values
- Provide a direct link to view your spreadsheet

### Email Forwarding

Set up email rule to forward to webhook:

```javascript
// Webhook handler
app.post('/webhook/email', (req, res) => {
  const { from, subject, body } = req.body;
  
  // Spawn Lead Intake Agent
  sessionsSpawn({
    task: `Process email lead: From: ${from}, Subject: ${subject}, Body: ${body}`,
    agentId: 'lead-intake'
  });
  
  res.status(200).send('OK');
});
```

## Database Backup

The database auto-backs up when file > 1MB. Manual backup:

```bash
# Create timestamped backup
cp leads.json "leads.json.backup.$(date +%s)"
```

## Troubleshooting

### Database Locked
If you get "file is locked" errors, check no other process is reading/writing.

### Empty Results
Make sure you're querying the correct path. Database is at:
`~/.openclaw/skills/mini-crm/crm-db/leads.json`

### Telegram Not Sending
Verify bot token and chat ID are configured correctly.

## Next Steps

1. Add your first lead manually to test the flow
2. Set up Telegram notifications
3. Configure cron jobs for automated summaries
4. Connect email/Google Sheets when ready
5. Train your team on asking Lexi pipeline questions

## Customization

### Adjust Scoring Criteria
Edit `agents/scoring/SKILL.md` to change Hot/Warm/Cold rules.

### Change Summary Time
Edit the cron job schedule to your preferred time.

### Add Custom Fields
Update the schema in all SKILL.md files and the database scripts.

## Support

This is your CRM - customize it as needed! The agents are just prompts and can be refined based on your specific business needs.
