# Instant lead reply

An n8n automation that answers every new website enquiry with a personal AI-drafted reply in under a minute. It logs the lead in Google Sheets and sends the draft to the business owner to check and send.

## Problem

Small businesses often take a day or two to answer new enquiries. By then many leads have moved on. Replies are also inconsistent, and some enquiries are never logged at all.

## Solution

```
Website form ──> Gemini drafts a reply ──> Row in "Leads" sheet ──> Draft emailed to the owner
                        │
                        └─ if the AI call fails ──> Row in "Leads" sheet with status = error
```

1. **Form** (n8n Form Trigger): name, email, company and message.
2. **Gemini** writes a short, warm first reply. It uses the lead's language and first name, answers only what they asked, and suggests a short call. It never invents prices, dates or claims.
3. **Google Sheets** logs the time, lead details, drafted reply and `status = drafted`.
4. **Gmail** sends the lead details and the suggested reply to the **owner's own inbox**. The lead never gets an automatic email, so a human always checks the reply first.
5. **Error branch:** if the AI call fails (rate limit, outage), the lead is still logged with `status = error` and the error message. No lead is lost silently.

## Tools

n8n (self-hosted), Google Gemini (`gemini-3-flash-preview`), Google Sheets, Gmail.

## Results on the sample data

Tested with the 6 leads in `sample-data.csv` (all fictional):

| Lead | What it tests | Result |
|---|---|---|
| Sarah Mitchell, James Okafor | Normal enquiries | Reply addresses their actual request |
| Priya Sharma | Empty company and message | Thanks her and asks what she needs |
| Lucía Fernández | Spanish message | Full reply in Spanish |
| Tom Becker | 1,268-character message | Short summary of his needs; no invented timeline or price; suggests a call |
| Aisha Khan | Vague ("need help") | Asks one simple question |
| Forced failure (invalid model name) | Error branch | Row logged with `status = error` and the error text; no email sent |

All 6 leads were logged and emailed within about a minute of submitting the form.

## Safety

- The AI only **drafts**. A person reviews every reply before it reaches a customer.
- Sheet cells are written as raw text (Cell Format: "Use Format From N8N"), so a lead cannot inject spreadsheet formulas.
- `workflow.json` holds no secrets. Credentials appear only as named references; each user connects their own Google and Gemini accounts.
- Demo data is fictional sample data only.

## Run it yourself

1. Import `workflow.json` into n8n (Workflows → Import from file).
2. Create your own credentials: Google Gemini (PaLM) API, Google Sheets OAuth2 and Gmail OAuth2.
3. Make a Google Sheet with a `Leads` tab and these headers in row 1: `time, name, email, company, message, reply, status, error`.
4. In the two Sheets nodes, select your sheet. In the Gmail node, set **To** to your own address.
5. Click **Execute workflow** and submit the test form.

## Known limits

- If Google Sheets or Gmail fails (for example an expired sign-in), the run stops. Only AI failures are caught by the error branch.
- No spam filtering or duplicate detection on the form yet.
