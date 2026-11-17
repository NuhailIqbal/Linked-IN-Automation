# LinkedIn DM Automation (n8n)

A human-in-the-loop LinkedIn outreach workflow in n8n: upload a leads CSV, research each lead's latest post, draft a personalised DM with Gemini, get it approved by email, and only then send it — with daily limits, duplicate guards, and reply tracking.

Built by [Nuhail Iqbal](https://www.linkedin.com/in/nuhail-iqbal-a93b1935b/) — AI Agent & Automation Engineer (n8n · VAPI · RAG · WhatsApp Chatbots · Python).

## How it works

```
Lead Upload Form → Parse + Validate Leads CSV → Loop Over Leads
  → Launch PhantomBuster → Wait → Get Result → Extract Latest Post
  → Get Unipile Profile → Check Connection (first-degree?)
  → Generate DM with Gemini → Parse Draft → Draft Valid?
  → Email Review + Wait (approve / reject)
  → Daily Limit + Duplicate Guard → Recheck Profile → Human Delay 15–30s
  → Send LinkedIn DM → Record Sent + Prevent Duplicates
  ↘ Unipile Reply Webhook → Verify + Record Reply → Email Reply Notification
```

### The safeguards (the point of this workflow)

- **Nothing sends without a human saying yes.** Every draft goes to your email first. You approve or reject it, and can edit the final message before it goes out.
- **Draft validation.** A draft must be 250–400 characters, end with a question, and contain no links — otherwise it is rejected before review.
- **Daily limit + duplicate guard.** Sends are counted per day in workflow static data, and a lead who already received a message is never messaged again.
- **Connection re-check.** The profile is re-checked right before sending, so a DM only goes to someone still connected.
- **Human delay.** A random 15–30 second wait sits between the final check and the send.
- **Reply webhook.** Incoming replies arrive on a secret-validated webhook and trigger an email notification.

### The pipeline in brief

1. **Lead Upload Form** — paste a CSV with `linkedin_url, first_name, company` (max 25 leads per run).
2. **PhantomBuster** — the LinkedIn Activity Extractor pulls the lead's latest post.
3. **Unipile** — checks the profile and confirms a first-degree connection.
4. **Gemini** — drafts a short, personal DM based on the lead's latest post.
5. **Email review** — you approve, reject, or edit the draft.
6. **Send** — via Unipile, after the limit, duplicate, and connection checks.
7. **Replies** — tracked on the reply webhook and emailed to you.

## Try it

### Prerequisites

- n8n (self-hosted or cloud)
- A PhantomBuster account with a LinkedIn Activity Extractor agent
- A Unipile account connected to LinkedIn
- A Google Gemini API key
- An SMTP account for the review and notification emails

### Setup

1. Import `workflows/LinkedIn_DM_Automation.json` into n8n (Workflows → Import from file).
2. Create credentials in n8n and connect them to the matching nodes:
   - Header Auth → PhantomBuster nodes, Unipile nodes
   - Header Auth with header `x-goog-api-key` → **Generate DM with Gemini**
   - SMTP → both email nodes
3. Open **CONFIG — EDIT ME** and fill in the four values (the node throws an error if any is still a placeholder):
   - PhantomBuster agent ID
   - Unipile account ID
   - Gemini model (default `gemini-3.1-flash-lite`)
   - SMTP sender email and webhook secret
4. In Unipile, add the production reply webhook URL, subscribe to `message.new`, and set the header `X-Webhook-Secret` to your secret.
5. Activate the workflow, open the Lead Upload Form URL, paste a CSV, and submit. Start with two test leads.

## A note on using this responsibly

Automating LinkedIn outreach can violate LinkedIn's terms of service if misused. This workflow is built with approval gates, validation, and limits precisely so a human stays in control — you are responsible for using it within LinkedIn's rules, Unipile's and PhantomBuster's terms, and applicable law. The sample data in this repo is fictional.

## Files

| File | What it is |
| --- | --- |
| `workflows/LinkedIn_DM_Automation.json` | The importable n8n workflow (37 nodes) |

## Notes

- The workflow ships with `active: false` and no credentials — add your own in n8n after import.
- All account IDs, secrets, emails, and sample lead data from the original deployment have been replaced with placeholders.

## License

MIT — see [LICENSE](LICENSE).
