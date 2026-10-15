
# AI Customer Support Ticket Triage

An AI-powered customer support automation workflow built with **n8n, OpenRouter, and Google Sheets**.

The workflow receives support tickets through a webhook, uses an LLM to classify them, generates a suggested response, stores the results in Google Sheets, and returns a JSON response.

## Features

- AI-powered ticket categorization and sentiment analysis
- Priority classification: Low, Normal, High, or Urgent
- Automatic human-review flagging
- AI-generated response drafts
- Automatic ticket storage in Google Sheets
- Automatic status assignment
- Webhook integration for HTTP POST requests
- Structured LLM output using an output parser

> **Note:** The workflow drafts replies but does not send emails, issue refunds, or perform account changes.

## Tech Stack

| Technology | Purpose |
|---|---|
| n8n | Workflow automation and orchestration |
| OpenRouter | LLM access |
| NVIDIA Nemotron | Configured language model |
| Structured Output Parser | Produces structured AI output |
| Google Sheets | Ticket storage and tracking |
| Postman | API testing |

## Workflow Architecture

```text
Webhook
   ↓
Edit Fields
   ↓
Basic LLM Chain
   ├── OpenRouter Chat Model
   └── Structured Output Parser
   ↓
If (human review required?)
   ├── True ──┐
   └── False ─┤
              ↓
     Append Row in Google Sheets
              ↓
       Respond to Webhook
```

### Workflow Steps

1. **Webhook:** Receives a ticket through an HTTP POST request.
2. **Edit Fields:** Extracts the customer's name, email, subject, and message.
3. **Basic LLM Chain:** Classifies the ticket and generates a response draft.
4. **Structured Output Parser:** Structures the model's output.
5. **If:** Checks whether human review is required.
6. **Google Sheets:** Appends the ticket and its classification to a spreadsheet.
7. **Respond to Webhook:** Returns a JSON confirmation to the requester.

Both branches of the IF node currently connect to the same Google Sheets node. The ticket's status is determined by the `requires_human_review` value.

## AI Classification

| Field | Description | Possible Values |
|---|---|---|
| Category | Type of support issue | Billing, Technical Support, Account Access, Order/Shipping, Refund/Return, General Inquiry, Other |
| Sentiment | Customer's sentiment | Positive, Neutral, Negative |
| Priority | Urgency of the issue | Low, Normal, High, Urgent |
| Reason | Explanation for the priority | Text |
| Response_Draft | Suggested reply for a support agent | Generated text |
| Human_Review | Whether human review is required | `true` / `false` |

### Priority Guidelines

- **Urgent:** Immediate serious risk or critical service disruption.
- **High:** Significant unresolved issue requiring prompt attention.
- **Normal:** Ordinary support request.
- **Low:** Minor or informational request.

High- and Urgent-priority tickets are flagged for human review. The model may also request review when a human decision is necessary.

## Requirements

- A running n8n instance
- An OpenRouter account and configured API credential
- A Google account with access to the destination spreadsheet
- Google Sheets OAuth2 credentials configured in n8n
- Postman or another HTTP client

## Setup

### 1. Import the Workflow

1. Export or save the n8n workflow JSON file.
2. Open n8n and import the JSON file.
3. Open the imported workflow and configure its credentials.

### 2. Configure OpenRouter

1. Open the **OpenRouter Chat Model** node.
2. Select your OpenRouter credential.
3. Confirm that the configured model is available.

The workflow currently uses `nvidia/nemotron-3-super-120b-a12b:free`. Model availability and provider limits may change.

### 3. Configure Google Sheets

1. Open the **Append row in sheet** node.
2. Select your Google Sheets credential.
3. Choose your spreadsheet and worksheet.
4. Ensure the first row contains these exact headers:

```text
Ticket_ID
Created_At
Customer_Name
Customer_Email
Subject
Message
Category
Sentiment
Priority
Reason
Response_Draft
Human_Review
Status
```

Update the spreadsheet selection if you want to use your own Google Sheet.

## Testing with Postman

### 1. Start the Webhook Test

Open the Webhook node and click **Listen for Test Event**.

Copy the test URL displayed by n8n. It commonly resembles:

```text
http://localhost:5678/webhook-test/support-ticket-triage
```

Use the exact URL displayed in your instance.

### 2. Configure the Request

- **Method:** `POST`
- **Body:** `raw`
- **Format:** `JSON`

Example request:

```json
{
  "customer_name": "Jane Doe",
  "customer_email": "jane@example.com",
  "subject": "I was charged twice",
  "message": "I placed one order but my card was charged twice. Please help me get the duplicate payment resolved."
}
```

Click **Send** while the webhook is listening.

### 3. Expected Response

```json
{
  "success": true,
  "message": "Ticket classified and saved successfully.",
  "ticket_id": "76",
  "category": "Billing",
  "sentiment": "Negative",
  "priority": "High",
  "human_review": true
}
```

The ticket ID and AI-generated results may vary. Confirm that a new row appears in Google Sheets.

## Request Format

The current workflow accepts **one ticket per request**.

| Field | Type | Description |
|---|---|---|
| `customer_name` | String | Customer's name |
| `customer_email` | String | Customer's email |
| `subject` | String | Ticket subject |
| `message` | String | Description of the issue |

Batch processing requires additional workflow logic to split an incoming JSON array into individual tickets.

## Google Sheets Output

Each processed ticket is stored with:

- Ticket ID and creation timestamp
- Customer information and original message
- AI-generated category, sentiment, and priority
- Reason and suggested response
- Human-review flag and status

The status is assigned as follows:

- `Pending Review` when human review is required
- `New` otherwise

## Deployment

The exported workflow is currently inactive.

For production use:

1. Configure and test your credentials.
2. Activate the workflow.
3. Use the production webhook URL shown by n8n.
4. Configure authentication and validate incoming requests before public deployment.

A public webpage cannot access `localhost` on your computer. You'll need a publicly reachable n8n instance or a securely configured tunnel.

## Security Considerations

- Use fictional customer data for testing.
- Never commit API keys or credentials to a public repository.
- Add authentication and request validation before exposing the webhook.
- Review AI-generated replies before sending them to customers.
- Do not treat AI-generated text as proof that a refund or other action occurred.

## Future Improvements

- Batch ticket processing
- Webhook authentication and input validation
- Error handling and failure notifications
- A public support-ticket submission interface
- A dedicated human-review queue
- Human approval before sending suggested replies
