# Real Estate Lead Qualification

Automated n8n workflow for capturing, qualifying, and managing real estate leads from multiple sources.

## Features

- 🔗 **Multi-source intake**: Webhook, Email, Google Forms
- ⚡ **Automatic scoring**: A/B/C grades based on budget
- 📊 **Google Sheets logging**: Centralized lead tracking
- 🤖 **Agent notifications**: Automated lead assignments

## Quick Start

### 1. Import Workflow
Import the `Real Estate Lead Qualification.json` file into your n8n instance.

### 2. Configure Credentials
- Google Sheets OAuth2
- IMAP email account
- Update webhook URLs

### 3. Test Webhook
```bash
curl -X POST https://your-n8n-instance/webhook/new-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "phone": "+1234567890",
    "budget": "300000",
    "propertyType": "Apartment",
    "location": "City Center"
  }'
```

## Lead Scoring

| Grade | Budget Range | Priority |
|-------|-------------|----------|
| A     | > $300,000  | High     |
| B     | > $150,000  | Medium   |
| C     | ≤ $150,000  | Low      |

## Data Sources

- **Webhook**: Web forms, landing pages
- **Email**: Leads with "New Lead" subject
- **Google Forms**: Direct form submissions

## Output

All leads are automatically:
1. Validated and standardized
2. Scored (A/B/C grade)
3. Logged to Google Sheets
4. Sent to assigned agents

## Requirements

- n8n instance
- Google Sheets access
- IMAP email account
- Webhook endpoint

## License

MIT
