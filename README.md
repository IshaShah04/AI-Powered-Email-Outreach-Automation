# AI-Powered Email Outreach Automation

An automated pipeline built in n8n that reads lead data from Google Sheets, generates personalized outreach emails using AI, and sends them via Gmail with a built-in confidence-based review gate.

## Overview

This project automates a common but time-consuming task: writing and sending personalized outreach emails to a list of leads. Instead of manually drafting each email, the workflow reads lead information, generates a tailored email using an AI model, checks how confident the AI is in the result, and either sends it automatically or flags it for manual review.

## Architecture

```
Google Sheets (read leads)
        |
        v
Google Gemini (generate personalized email as structured JSON)
        |
        v
Code Node (parse AI response into subject/body/confidence fields)
        |
        v
IF Node (confidence >= 0.7 ?)
        |
   ---------------------
   |                   |
 True                False
   |                   |
Gmail (send email)   Google Sheets (mark "Needs Review")
   |
Google Sheets (mark "Sent")
```

## How It Works

1. **Lead source**: Leads live in a Google Sheet with columns: Name, Email, Company, Purpose, Status.
2. **Trigger**: The workflow is started manually. Only rows with Status set to "Pending" are processed.
3. **Email generation**: Each lead's data is sent to Google Gemini with a system prompt instructing it to write a short, professional, personalized email and return the result as structured JSON containing a subject, body, and a confidence score.
4. **Parsing**: A Code node parses the AI's JSON response and merges it back with the original lead data, since the AI's response does not automatically carry forward the original spreadsheet fields.
5. **Confidence check**: An IF node checks the AI's self-reported confidence score.
   - If confidence is 0.7 or higher, the email is sent automatically via Gmail, and the row's Status is updated to "Sent."
   - If confidence is below 0.7, no email is sent. Instead, the row's Status is updated to "Needs Review" so it can be checked manually.
6. **Status tracking**: The Google Sheet always reflects the current state of each lead, preventing duplicate sends on future runs.

## Tech Stack

- n8n (workflow orchestration)
- Google Sheets API (lead storage and status tracking)
- Google Gemini API (email generation, free tier)
- Gmail API (sending emails)
- JavaScript (used in the Code node for parsing AI output)

## Setup Instructions

To run this workflow yourself, you will need:

1. An n8n instance (cloud or self-hosted).
2. A Google account with:
   - A Google Sheet containing the columns: Name, Email, Company, Purpose, Status.
   - Google Sheets and Gmail credentials configured in n8n via OAuth.
3. A free Google Gemini API key, available at aistudio.google.com/apikey.
4. Import the workflow JSON file from this repository into n8n.
5. Connect your own credentials to the Google Sheets, Gemini, and Gmail nodes (credentials are not included in the exported workflow file).
6. Update the Document and Sheet fields in the Google Sheets nodes to point to your own spreadsheet.

## Design Decisions

- **AI is used only where reasoning is required.** Reading spreadsheet data, parsing JSON, checking a confidence threshold, and sending an email are all handled with deterministic n8n logic rather than additional AI calls. This keeps the system cheap, predictable, and easy to debug.
- **A single AI call handles both personalization and drafting.** A separate "personalization agent" was considered but judged unnecessary at this scale, since one well-structured prompt can handle both tasks reliably.
- **Confidence-based routing was chosen over always requiring manual approval.** This balances automation with safety: emails the AI is confident about go out immediately, while uncertain ones are held back for a human to check.

## Known Limitations

- This is a personal/learning project built at a small, non-production scale, not a deployed business system.
- The workflow currently uses a manual trigger rather than a schedule, by design, so nothing is sent without the operator explicitly starting a run.
- Follow-up emails, reply handling, and analytics are not implemented in this version.

## Possible Future Improvements

- Add a scheduled trigger for fully automated daily runs.
- Add a human-approval step (via n8n Form or Slack) for emails below the confidence threshold, rather than only flagging them.
- Add follow-up email logic for leads that do not reply within a set number of days.
- Add reply classification to detect interest, opt-outs, and other response types automatically.
