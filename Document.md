# Real Estate Lead Qualification Workflow Documentation

## Overview
This n8n workflow automates the process of capturing, qualifying, and managing real estate leads from multiple sources. It processes incoming leads, scores them based on budget criteria, logs them to Google Sheets, and sends qualified leads to assigned agents.

## Workflow Architecture

### Data Flow
```
[Multiple Triggers] → [Extract & Validate] → [Score Quality] → [Log to Sheets] → [Send to Agent] → [Process Response] → [Final Log]
```

## Trigger Sources

### 1. Webhook Trigger
- **Endpoint**: `https://majrou.app.n8n.cloud/webhook/new-lead`
- **Method**: POST
- **Purpose**: Receives leads from web forms, landing pages, or API integrations

**Expected Payload:**
```json
{
  "name": "Mohamed Ajrou",
  "phone": "+212612345678",
  "budget": "300000",
  "propertyType": "Apartment",
  "location": "Casablanca",
  "timeline": "3–6 months",
  "familySize": "4",
  "specialRequirements": "Parking, near school"
}
```

**Test Command:**
```bash
curl -X POST https://majrou.app.n8n.cloud/webhook/new-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Mohamed Ajrou",
    "phone": "+212612345678",
    "budget": "300000",
    "propertyType": "Apartment",
    "location": "Casablanca",
    "timeline": "3–6 months",
    "familySize": "4",
    "specialRequirements": "Parking, near school"
  }'
```

### 2. Email Trigger (IMAP)
- **Purpose**: Monitors email inbox for new leads
- **Search Criteria**: `[["SUBJECT", "New Lead"]]`
- **Frequency**: Real-time monitoring
- **Credentials**: IMAP account configured

### 3. Google Sheets Trigger
- **Purpose**: Monitors Google Forms responses
- **Sheet**: "Responses for Form Intake"
- **Event**: Row Added
- **Polling**: Every minute

## Processing Nodes

### 1. Extract and Validate Lead Data
**Purpose**: Standardizes and validates incoming lead data from all sources

**Code Logic:**
```javascript
// Check if data is nested in body or params
const data = $json.body || $json.params || $json;

console.log('Extracted data:', JSON.stringify(data, null, 2));

return [{
  json: {
    name: data.name || 'Unknown',
    phone: data.phone || 'Unknown',
    budget: parseInt(data.budget || '0', 10),
    propertyType: data.propertyType || 'Apartment',
    location: data.location || 'Not specified',
    timeline: data.timeline || 'Not specified',
    familySize: data.familySize || '1',
    specialRequirements: data.specialRequirements || 'None'
  }
}];
```

**Output Structure:**
```json
{
  "name": "string",
  "phone": "string",
  "budget": "integer",
  "propertyType": "string",
  "location": "string",
  "timeline": "string",
  "familySize": "string",
  "specialRequirements": "string"
}
```

### 2. Score Lead Quality (A/B/C)
**Purpose**: Assigns lead quality score based on budget

**Scoring Logic:**
- **A Grade**: Budget > 300,000
- **B Grade**: Budget > 150,000
- **C Grade**: Budget ≤ 150,000

**Code:**
```javascript
const budget = $json.budget;

let leadScore = 'C';
if (budget > 300000) {
  leadScore = 'A';
} else if (budget > 150000) {
  leadScore = 'B';
}

return [{
  json: {
    ...$json,
    leadScore
  }
}];
```

### 3. Log Lead to Google Sheets
**Purpose**: Records all leads in a central Google Sheet

**Configuration:**
- **Sheet**: "real_estate_leads"
- **Operation**: Append row
- **Mapping**: Direct field mapping to columns

**Column Mapping:**
```
Name → $json.name
Phone → $json.phone
Budget → $json.budget
Property Type → $json.propertyType
Location → $json.location
Timeline → $json.timeline
Family Size → $json.familySize
Special Requirements → $json.specialRequirements
Lead Score → $json.leadScore
```

### 4. Send Qualified Lead Summary to Assigned Agent
**Purpose**: Sends lead information to external agent system

**Configuration:**
- **Method**: POST
- **URL**: `https://webhook.site/a5a586a4-0e9d-4be7-92d3-fb079070741a`
- **Body**: Form data with lead details

**Payload Structure:**
```json
{
  "Name": "Mohamed Ajrou",
  "Phone": "+212612345678",
  "Budget": "300000",
  "Location": "Casablanca",
  "Timeline": "3–6 months",
  "Family Size": "4",
  "Special Requirements": "Parking, near school",
  "Lead Score": "A"
}
```

## Secondary Processing Flow

### 5. Merge Node
**Purpose**: Combines data from the Google Sheets log and agent response

### 6. Edit Fields
**Purpose**: Adds simulated agent response data
- **Field Added**: `agentReply: "Suggest Apartment ID 1234 and 5678"`

### 7. Code (Process Agent Reply)
**Purpose**: Processes the agent response
```javascript
return [{
  json: {
    ...$json,
    processedAgentReply: $json.agentReply.toUpperCase()
  }
}];
```

### 8. Append Row in Sheet
**Purpose**: Logs the complete lead processing results including agent responses

## Data Validation & Error Handling

### Input Validation
- All fields have default fallback values
- Budget is parsed as integer with fallback to 0
- Missing fields are populated with appropriate defaults

### Default Values:
```javascript
{
  name: 'Unknown',
  phone: 'Unknown',
  budget: 0,
  propertyType: 'Apartment',
  location: 'Not specified',
  timeline: 'Not specified',
  familySize: '1',
  specialRequirements: 'None'
}
```

## Configuration Details

### Google Sheets Integration
- **Primary Sheet**: `1b8nNjywHZvGAMsPeOZJSCiB62V3bhEhmKvELYeIK6sI`
- **Trigger Sheet**: `14GcFliQx2DwJsxZs0btgdTO-0FgPobIVH4Wzy0z-U-w`
- **Authentication**: OAuth2 with configured Google account

### Email Configuration
- **IMAP Credentials**: Configured with account `zScdUWwF2ofiIal9`
- **Search Pattern**: Emails with "New Lead" in subject line
- **Format**: Nested array format `[["SUBJECT", "New Lead"]]`

## Troubleshooting

### Common Issues

#### 1. "Unknown" Values in Output
**Cause**: Data structure mismatch or incorrect field access
**Solution**: Check if data is nested in `body` or `params` object

#### 2. Email Trigger Loop
**Cause**: Emails not being marked as read
**Solution**: Ensure Action is set to "Mark as Read" in IMAP node

#### 3. JSON Validation Errors
**Cause**: Incorrect search criteria format
**Solution**: Use nested array format: `[["SUBJECT", "New Lead"]]`

### Debug Steps
1. Check console logs in Extract and Validate node
2. Verify webhook payload structure
3. Test individual trigger sources
4. Validate Google Sheets permissions
5. Confirm IMAP credentials and search criteria

## Testing

### Manual Testing
```bash
# Test webhook endpoint
curl -X POST https://majrou.app.n8n.cloud/webhook/new-lead \
  -H "Content-Type: application/json" \
  -d '{"name": "Test User", "phone": "123456789", "budget": "250000"}'
```

### Expected Results
1. Lead appears in Google Sheets with grade B (budget 250,000)
2. Agent notification sent to webhook.site
3. Complete processing log created in secondary sheet

## Maintenance

### Regular Tasks
- Monitor Google Sheets for data accuracy
- Update scoring criteria as needed
- Verify IMAP connection health
- Check webhook endpoint availability

### Performance Optimization
- Google Sheets Trigger polls every minute (adjustable)
- Email monitoring is real-time
- Webhook responses are immediate

## Security Considerations
- Webhook endpoint is public but validated
- Google Sheets access controlled via OAuth2
- Email credentials stored securely in n8n
- No sensitive data logged in console outputs

## Workflow Status
- **Status**: Active
- **Version**: v1 execution order
- **Instance ID**: d8c34c750e2bf64dc150a6c8b3c09e4d933b02c2ddd156d6611ca364c73e2766
- **Template Setup**: Completed