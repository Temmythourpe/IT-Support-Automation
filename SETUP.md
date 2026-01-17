# Setup Guide - IT Support Automation

## Prerequisites

- **n8n** installed locally or n8n cloud account
- **Groq API** account (free at https://console.groq.com)
- **Google Account** for Sheets integration

## Step 1: Install n8n

### Option A: Local Installation (Docker)
```bash
docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
```

### Option B: n8n Cloud
Sign up at https://n8n.io and create a free account.

Access n8n at `http://localhost:5678` (local) or your cloud URL.

## Step 2: Get API Keys

### Groq API Key
1. Go to https://console.groq.com
2. Sign up for a free account
3. Navigate to **API Keys** section
4. Click **Create API Key**
5. Copy and save your key (starts with `gsk_`)

### Google OAuth Credentials
1. Go to https://console.cloud.google.com
2. Create a new project or select existing
3. Enable **Google Sheets API**
4. Go to **Credentials** → **Create Credentials** → **OAuth Client ID**
5. Choose **Desktop app**
6. Save Client ID and Client Secret

## Step 3: Import Workflow

1. In n8n, click **"+ Create new workflow"**
2. Click the three dots **(⋮)** menu → **Import from File**
3. Select `workflows/it-support-automation.json`
4. Workflow will load on the canvas

## Step 4: Configure Credentials

### A. Groq API (HTTP Request Node)
1. Click on **HTTP Request** node
2. Under Authentication → **Create New Credential**
3. Select **Header Auth**
4. **Name**: `Authorization`
5. **Value**: `Bearer YOUR_GROQ_API_KEY` (replace with your actual key)
6. Save credential

### B. Google Sheets
1. Click on **Google Sheets** node
2. Click **Create New Credential**
3. Enter your **Client ID** and **Client Secret**
4. Click **Connect my account**
5. Sign in with Google and grant permissions
6. Add yourself as a test user in Google Cloud Console if needed

## Step 5: Create Google Sheet

1. Create a new Google Sheet named **"IT Support Tickets"**
2. Add these headers in Row 1:
```
   Ticket ID | User | Category | Priority | Status | Sentiment | Timestamp
```
3. Copy the Sheet ID from the URL (the long string between `/d/` and `/edit`)
4. In n8n Google Sheets node, select this spreadsheet

## Step 6: Activate Workflow

1. Click **Save** (top right)
2. Click **Publish** to activate
3. Copy the **Production Webhook URL** from the Webhook node

## Step 7: Test the Workflow

### Test 1: Password Reset (Auto-Resolve)
```bash
curl -X POST YOUR_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Forgot my password",
    "description": "I need to reset my password urgently",
    "user": "test@company.com"
  }'
```

### Test 2: Hardware Issue (Human Review)
```bash
curl -X POST YOUR_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Laptop screen flickering",
    "description": "My screen flickers when opening applications",
    "user": "test@company.com"
  }'
```

**Expected Results:**
- Password reset → Status: `auto_resolved`
- Hardware issue → Status: `pending_human_review`
- Both logged in Google Sheets

## Monitoring

- Check **Executions** tab in n8n for execution history
- Review Google Sheet for all logged tickets
- Monitor for errors in the workflow

## Troubleshooting

### API Key Errors
- Verify Groq API key is correct and active
- Ensure `Bearer ` prefix is included with one space

### Google Sheets Not Updating
- Check OAuth connection is active
- Verify sheet name and ID are correct
- Ensure columns are mapped properly

### Webhook 404 Errors
- Ensure workflow is **Published/Active**
- Use production URL, not test URL
- Check webhook path matches your configuration

## Security Best Practices

1. **Never commit credentials** to version control
2. Use **environment variables** in production
3. **Rotate API keys** regularly
4. **Restrict webhook access** using authentication
5. **Monitor logs** for suspicious activity

## Support

For issues or questions:
- Open an issue on GitHub
- Check n8n community forums
- Review Groq API documentation

---