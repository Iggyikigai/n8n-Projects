# MVNE Email Classifier - eSIM Ticket Escalation System

An AI-powered n8n workflow for automated email classification, ticket triage, and escalation for MVNE (Mobile Virtual Network Enabler) support operations, specifically designed for <u>eSIM activation</u> and related telco support issues.

## Overview

This workflow automates the entire customer support lifecycle from initial email classification to ticket escalation, using AI to intelligently categorise support requests and manage customer interactions.

## Key Features

- **AI-Powered Classification**: Automatically categorises incoming emails into 5 predefined categories:
  - eSim activation issue
  - Data roaming connectivity issues
  - Plan top-up issues
  - Usage investigation
  - Others

- **Automated Response System**: 
  - Sends first-step troubleshooting instructions based on ticket category
  - Retrieves troubleshooting steps from Google Sheets knowledge base
  - Tracks customer responses and determines resolution status

- **Smart Reply Detection**: 
  - Monitors customer replies using AI to detect if issues are "solved", "not solved", or "unclear"
  - Automatically routes tickets based on customer feedback

- **Ticket Escalation**: 
  - Compiles comprehensive escalation packages with full thread history
  - Generates structured summaries for system engineers
  - Sends escalation emails to external partners (e.g., Singtel)

- **Gmail Label Management**: 
  - Automatically applies appropriate labels (`ticket/esim`, `ticket/roaming`, etc.)
  - Tracks ticket status with labels (`ticket/awaiting-customer`, `ticket/escalated`, etc.)

- **Activity Logging**: 
  - Logs all ticket actions to Google Sheets for audit and analytics
  - Tracks ticket lifecycle from creation to resolution

## Workflow Components

### Workflow A: Intake (Email Classification)
1. **Gmail Trigger**: Monitors inbox for unread emails (excluding certain labels)
2. **Email Normalization**: Extracts and cleans email content (subject, body, headers)
3. **AI Classification**: Uses DeepSeek AI model to categorize the ticket
4. **Label Application**: Applies appropriate Gmail labels based on category
5. **First Steps Response**: Looks up troubleshooting steps and sends automated reply
6. **Logging**: Records ticket prediction and initial action

### Workflow B: Reply Listener (Customer Response Handling)
1. **Reply Detection**: Monitors for customer replies in labeled threads
2. **Response Extraction**: Extracts customer's reply content (removes quoted history)
3. **AI Solve Detection**: Determines if customer indicates issue is solved/not solved
4. **Routing**:
   - **Solved**: Logs resolution and sends confirmation
   - **Not Solved**: Escalates to system engineers with full context
   - **Unclear**: Continues monitoring

### Escalation Flow
- Fetches complete email thread
- Uses AI to generate structured escalation summary
- Creates comprehensive HTML escalation package with:
  - Issue summary and description
  - Customer information
  - Troubleshooting steps already provided
  - Full email thread history
  - Next actions for engineers
- Sends escalation email to designated recipients
- Notifies customer of escalation

## Prerequisites

### Required n8n Credentials
- **Gmail OAuth2**: Admin Gmail account with API access
- **Google Sheets OAuth2**: Access to Google Sheets for knowledge base and logging
- **DeepSeek API**: API key for AI classification and summarization

### Required Gmail Labels
Create the following labels in your Gmail account:
- `ticket/esim`
- `ticket/roaming`
- `ticket/topup`
- `ticket/usage`
- `ticket/others`
- `ticket/awaiting-customer`
- `ticket/escalated`
- `ticket/reminded`
- `ticket/closed`

### Required Google Sheets
1. **First Steps Sheet**: Contains troubleshooting steps by category
   - Columns: `category`, `steps`
   - Sheet name: "First Steps"

2. **Logs Sheet**: Tracks all ticket activities
   - Columns: `timestamp`, `threadId`, `messageId`, `subject`, `category`, `confidence`, `action`
   - Sheet name: "ESIM Demo Logs"

## Configuration

### Key Configuration Points

1. **Gmail Trigger Filter**: Modify the query in "eSim Trigger" node to adjust which emails are processed
   ```
   in:inbox is:unread -label:"ticket/awaiting-customer" -label:"ticket/reminded" -label:"ticket/closed" -label:"ticket/escalated"
   ```

2. **Escalation Recipient**: Update the "Escalate to Singtel" node with the correct recipient email

3. **Support Email**: Configure the admin support email address in relevant nodes (default: `tribalaiiptribe@gmail.com`)

4. **Reminder Timing**: Adjust wait times in reminder logic nodes (currently set to 1 minute for testing)

## Workflow Structure

```
Intake Flow:
Gmail Trigger → Normalize Email → AI Classify → Apply Labels → Lookup Steps → Send Reply → Log

Reply Flow:
Gmail Trigger → Extract Reply → AI Solve Detection → Route (Solved/Not Solved/Escalate)

Escalation Flow:
Get Thread → Prepare Context → AI Summarize → Send Escalation → Notify Customer → Log
```

## Notes

- The workflow uses structured output parsing for reliable AI responses
- All sensitive data (credentials, API keys) should be configured in n8n credentials, not in the workflow JSON
- Some reminder and auto-closing features are currently disabled (marked in workflow)
- The workflow is designed for production use but includes demo/test configurations

## License

Apache-2.0


