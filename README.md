<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Customer Support Ticket Triage</title>
  <style>
    :root {
      color-scheme: light;
      --primary: #2563eb;
      --heading: #172554;
      --text: #334155;
      --muted: #64748b;
      --border: #e2e8f0;
      --background: #f8fafc;
      --code-bg: #eef2f7;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 40px 20px;
      background: var(--background);
      color: var(--text);
      font-family: Arial, Helvetica, sans-serif;
      line-height: 1.7;
    }

    main {
      max-width: 950px;
      margin: auto;
      padding: 40px;
      background: white;
      border: 1px solid var(--border);
      border-radius: 12px;
    }

    h1, h2, h3 {
      color: var(--heading);
      line-height: 1.3;
    }

    h1 {
      font-size: 2.2rem;
      margin-top: 0;
    }

    h2 {
      margin-top: 38px;
      padding-bottom: 8px;
      border-bottom: 1px solid var(--border);
    }

    a {
      color: var(--primary);
    }

    code {
      padding: 3px 6px;
      background: var(--code-bg);
      border-radius: 4px;
      font-family: Consolas, monospace;
      overflow-wrap: anywhere;
    }

    pre {
      padding: 18px;
      overflow-x: auto;
      background: #f1f5f9;
      border: 1px solid var(--border);
      border-radius: 8px;
      line-height: 1.5;
    }

    pre code {
      padding: 0;
      background: transparent;
      overflow-wrap: normal;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin: 18px 0;
    }

    th, td {
      padding: 12px;
      border: 1px solid var(--border);
      text-align: left;
      vertical-align: top;
    }

    th {
      background: #eff6ff;
      color: var(--heading);
    }

    .subtitle {
      color: var(--muted);
      font-size: 1.1rem;
    }

    .note {
      padding: 14px 18px;
      background: #eff6ff;
      border-left: 4px solid var(--primary);
      border-radius: 4px;
    }

    .warning {
      padding: 14px 18px;
      background: #fff7ed;
      border-left: 4px solid #ea580c;
      border-radius: 4px;
    }

    .workflow {
      padding: 18px;
      background: #f8fafc;
      border: 1px solid var(--border);
      border-radius: 8px;
      font-family: Consolas, monospace;
      white-space: pre-wrap;
    }

    footer {
      margin-top: 40px;
      padding-top: 20px;
      border-top: 1px solid var(--border);
      color: var(--muted);
      font-size: 0.9rem;
    }

    @media (max-width: 600px) {
      body {
        padding: 12px;
      }

      main {
        padding: 20px;
      }

      h1 {
        font-size: 1.7rem;
      }

      table {
        display: block;
        overflow-x: auto;
      }
    }
  </style>
</head>

<body>
<main>
  <header>
    <h1>AI Customer Support Ticket Triage</h1>
    <p class="subtitle">
      An AI-powered support ticket classification and tracking
      workflow built with n8n, OpenRouter, and Google Sheets.
    </p>
  </header>

  <section>
    <h2>Overview</h2>
    <p>
      This project automates the initial triage of customer support
      tickets. It receives a ticket through a webhook, uses a large
      language model (LLM) to analyze and classify the issue, saves
      the results to Google Sheets, and returns a JSON response to
      the requesting application.
    </p>
    <p>
      The workflow helps support teams organize incoming requests,
      identify high-priority issues, and prepare suggested replies
      for support agents.
    </p>
    <div class="note">
      <strong>Important:</strong> The system generates response drafts.
      It does not send customer emails, issue refunds, or perform
      account changes.
    </div>
  </section>

  <section>
    <h2>Features</h2>
    <ul>
      <li>Webhook-based ticket submission using HTTP POST.</li>
      <li>AI-powered ticket categorization and sentiment analysis.</li>
      <li>Priority classification: Low, Normal, High, or Urgent.</li>
      <li>Automatic human-review flagging.</li>
      <li>AI-generated, empathetic response drafts.</li>
      <li>Automatic ticket storage in Google Sheets.</li>
      <li>Automatic ticket status assignment.</li>
      <li>JSON response containing the ticket ID and classification.</li>
    </ul>
  </section>

  <section>
    <h2>Technologies</h2>
    <ul>
      <li><strong>n8n:</strong> Workflow automation and orchestration.</li>
      <li><strong>OpenRouter:</strong> LLM access.</li>
      <li><strong>NVIDIA Nemotron:</strong> Configured model,
        <code>nvidia/nemotron-3-super-120b-a12b:free</code>.</li>
      <li><strong>Structured Output Parser:</strong> Structures the
        model's response into defined fields.</li>
      <li><strong>Google Sheets:</strong> Ticket storage and tracking.</li>
      <li><strong>Postman:</strong> Webhook testing.</li>
    </ul>
    <p>
      Model availability and free-tier limits may change. Select an
      available model in OpenRouter if the configured model is no
      longer accessible.
    </p>
  </section>

  <section>
    <h2>Workflow Architecture</h2>
    <div class="workflow">Webhook
   ↓
Edit Fields
   ↓
Basic LLM Chain
   ├── OpenRouter Chat Model
   └── Structured Output Parser
   ↓
If (human review required?)
   ├── True  ──┐
   └── False ──┤
               ↓
      Append Row in Google Sheets
               ↓
       Respond to Webhook</div>

    <h3>Workflow steps</h3>
    <ol>
      <li>
        <strong>Webhook:</strong> Receives a ticket as a JSON POST
        request.
      </li>
      <li>
        <strong>Edit Fields:</strong> Extracts the customer's name,
        email, subject, and message.
      </li>
      <li>
        <strong>Basic LLM Chain:</strong> Analyzes the ticket and
        generates classification fields and a response draft.
      </li>
      <li>
        <strong>Structured Output Parser:</strong> Enforces the
        expected output structure.
      </li>
      <li>
        <strong>If:</strong> Checks whether human review is required.
        Both branches currently continue to the same Google Sheets node.
      </li>
      <li>
        <strong>Google Sheets:</strong> Appends a new row containing
        the original ticket and AI results.
      </li>
      <li>
        <strong>Respond to Webhook:</strong> Returns a JSON confirmation
        after the Google Sheets node executes.
      </li>
    </ol>
  </section>

  <section>
    <h2>AI Classification Fields</h2>
    <table>
      <thead>
        <tr>
          <th>Field</th>
          <th>Purpose</th>
          <th>Possible values</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Category</td>
          <td>Identifies the type of issue.</td>
          <td>Billing, Technical Support, Account Access,
            Order/Shipping, Refund/Return, General Inquiry, Other</td>
        </tr>
        <tr>
          <td>Sentiment</td>
          <td>Estimates the customer's sentiment.</td>
          <td>Positive, Neutral, Negative</td>
        </tr>
        <tr>
          <td>Priority</td>
          <td>Estimates the urgency of the request.</td>
          <td>Low, Normal, High, Urgent</td>
        </tr>
        <tr>
          <td>Reason</td>
          <td>Explains the selected priority.</td>
          <td>Short text explanation</td>
        </tr>
        <tr>
          <td>Response_Draft</td>
          <td>Suggests a professional response for an agent.</td>
          <td>Generated text</td>
        </tr>
        <tr>
          <td>Human_Review</td>
          <td>Indicates whether a human should review the ticket.</td>
          <td>true / false</td>
        </tr>
      </tbody>
    </table>

    <h3>Priority guidelines</h3>
    <ul>
      <li><strong>Urgent:</strong> Immediate serious risk or critical service disruption.</li>
      <li><strong>High:</strong> Significant unresolved issue requiring prompt attention.</li>
      <li><strong>Normal:</strong> Ordinary support request.</li>
      <li><strong>Low:</strong> Minor or informational request.</li>
    </ul>
    <p>
      High- and Urgent-priority tickets are flagged for human review.
      The model may also request review when a human decision is needed.
      AI classifications are recommendations and should be validated.
    </p>
  </section>

  <section>
    <h2>Requirements</h2>
    <ul>
      <li>A running n8n instance. This workflow was exported from
        n8n 2.43.5 Self Hosted.</li>
      <li>An OpenRouter account and configured n8n credential.</li>
      <li>A Google account with access to the destination spreadsheet.</li>
      <li>A Google Sheets OAuth2 credential configured in n8n.</li>
      <li>Postman or another HTTP client.</li>
    </ul>
  </section>

  <section>
    <h2>Setup</h2>

    <h3>1. Import the workflow</h3>
    <ol>
      <li>Save the exported workflow JSON file.</li>
      <li>Open n8n and import the JSON from the workflow menu.</li>
      <li>Open the imported workflow and review its nodes and connections.</li>
    </ol>

    <h3>2. Configure OpenRouter</h3>
    <ol>
      <li>Open the <strong>OpenRouter Chat Model</strong> node.</li>
      <li>Select or create an OpenRouter credential.</li>
      <li>Confirm that the configured model is available to your account.</li>
      <li>Test the workflow and inspect the Basic LLM Chain output.</li>
    </ol>

    <h3>3. Configure Google Sheets</h3>
    <ol>
      <li>Open the <strong>Append row in sheet</strong> node.</li>
      <li>Select your Google Sheets credential.</li>
      <li>Select your spreadsheet and worksheet.</li>
      <li>Ensure the first row contains the following exact headers:</li>
    </ol>

    <pre><code>Ticket_ID
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
Status</code></pre>

    <p>
      The exported workflow references an existing spreadsheet.
      Replace the spreadsheet and worksheet selections if you want
      to use your own destination.
    </p>

    <h3>4. Check the webhook</h3>
    <ul>
      <li>HTTP method: <code>POST</code></li>
      <li>Path: <code>support-ticket-triage</code></li>
      <li>Response mode: Using Respond to Webhook Node</li>
    </ul>

    <div class="warning">
      <strong>Security warning:</strong> The supplied workflow has no
      webhook authentication. Configure appropriate authentication
      and input validation before exposing it to the public internet.
    </div>
  </section>

  <section>
    <h2>Testing with Postman</h2>

    <h3>1. Start the test listener</h3>
    <ol>
      <li>Open the Webhook node.</li>
      <li>Click <strong>Listen for Test Event</strong>.</li>
      <li>Copy the Test URL displayed by n8n.</li>
    </ol>

    <p>A local test URL commonly resembles:</p>
    <pre><code>http://localhost:5678/webhook-test/support-ticket-triage</code></pre>
    <p>Use the exact URL shown in your Webhook node.</p>

    <h3>2. Configure the Postman request</h3>
    <ul>
      <li>Method: <code>POST</code></li>
      <li>Body: <strong>raw → JSON</strong></li>
    </ul>

    <p>Example request body:</p>
    <pre><code>{
  "customer_name": "Jane Doe",
  "customer_email": "jane@example.com",
  "subject": "I was charged twice",
  "message": "I placed one order but my card was charged twice. Please help me get the duplicate payment resolved."
}</code></pre>

    <p>Click <strong>Send</strong> while the webhook is listening.</p>

    <h3>3. Expected response</h3>
    <pre><code>{
  "success": true,
  "message": "Ticket classified and saved successfully.",
  "ticket_id": "76",
  "category": "Billing",
  "sentiment": "Negative",
  "priority": "High",
  "human_review": true
}</code></pre>

    <p>
      The ticket ID and AI-generated values may differ. Verify that
      the ticket is saved as a new row in Google Sheets.
    </p>
  </section>

  <section>
    <h2>Request Schema</h2>
    <p>The workflow currently accepts <strong>one ticket per request</strong>.</p>

    <table>
      <thead>
        <tr><th>Field</th><th>Type</th><th>Description</th></tr>
      </thead>
      <tbody>
        <tr><td><code>customer_name</code></td><td>String</td><td>Customer's name</td></tr>
        <tr><td><code>customer_email</code></td><td>String</td><td>Customer's email address</td></tr>
        <tr><td><code>subject</code></td><td>String</td><td>Short description of the issue</td></tr>
        <tr><td><code>message</code></td><td>String</td><td>Details of the customer's request</td></tr>
      </tbody>
    </table>

    <p>
      Arrays of tickets are not supported by the current workflow.
      Send tickets individually or add a node to split the incoming
      array into separate items before the AI node.
    </p>
  </section>

  <section>
    <h2>Google Sheets Output</h2>
    <p>Each processed ticket is appended with these fields:</p>
    <ul>
      <li><strong>Ticket_ID:</strong> n8n execution ID.</li>
      <li><strong>Created_At:</strong> Timestamp formatted for Africa/Cairo.</li>
      <li><strong>Customer fields:</strong> Name, email, subject, and message.</li>
      <li><strong>AI fields:</strong> Category, sentiment, priority, reason, and response draft.</li>
      <li><strong>Human_Review:</strong> Whether a human review is required.</li>
      <li><strong>Status:</strong> <code>Pending Review</code> when human review is required; otherwise <code>New</code>.</li>
    </ul>
  </section>

  <section>
    <h2>Production Deployment</h2>
    <p>
      The workflow export is inactive. For production use, activate it
      in n8n and use the production webhook URL displayed by the Webhook node.
    </p>
    <ul>
      <li>The test URL works while the workflow is listening for a test event.</li>
      <li>The production URL requires the workflow to be active.</li>
      <li>A public webpage cannot reach <code>localhost</code> on your computer.</li>
      <li>For a public demo, host n8n on a reachable server or use a securely configured tunnel.</li>
    </ul>
  </section>

  <section>
    <h2>Troubleshooting</h2>

    <h3>The webhook returns the original request</h3>
    <p>
      Confirm that the Webhook node uses the Respond to Webhook node
      for its response and that the Google Sheets node connects to it.
    </p>

    <h3>Google Sheets does not save a row</h3>
    <p>
      Check credentials, the spreadsheet and worksheet selections,
      column headers, and the Google Sheets node execution output.
    </p>

    <h3>OpenRouter returns an error</h3>
    <p>
      Verify your credential, model availability, and provider usage limits.
    </p>

    <h3>The ticket status is incorrect</h3>
    <p>
      Inspect <code>requires_human_review</code> in the LLM output and
      verify the Status mapping in the Google Sheets node.
    </p>
  </section>

  <section>
    <h2>Limitations and Future Improvements</h2>
    <ul>
      <li>Add batch ticket processing.</li>
      <li>Add webhook authentication and request validation.</li>
      <li>Add error handling and failure notifications.</li>
      <li>Build a public support-ticket submission interface.</li>
      <li>Route human-review tickets to a dedicated queue or notification.</li>
      <li>Require human approval before sending AI-generated replies.</li>
    </ul>
  </section>

  <section>
    <h2>Privacy and Responsible Use</h2>
    <ul>
      <li>Use fictional customer information for testing.</li>
      <li>Store only the personal information needed for support.</li>
      <li>Do not expose API keys or credentials in public repositories.</li>
      <li>Review AI-generated responses before sending them to customers.</li>
      <li>Do not treat AI-generated text as evidence that a refund or other action occurred.</li>
    </ul>
  </section>

  <footer>
    <p>
      AI Customer Support Ticket Triage · Built with n8n, OpenRouter,
      and Google Sheets.
    </p>
    <p>Add a license file if you plan to distribute this project.</p>
  </footer>
</main>
</body>
</html>
