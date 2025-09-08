# Invoice Check Workflow

A small, reliable n8n workflow that scans a CSV of domains every morning, finds domains expiring in < 2 days, and posts a JSON-only summary to Telegram.
It also uses the Jalali (Shamsi) calendar for the “current date” context.

## Features

⏰ Scheduled: runs daily at 09:00 (Asia/Tehran).

📁 CSV ingest: reads domains.csv from disk.

🗂️ Markdown table: converts rows into a compact table for the LLM prompt.

🗓️ Jalali date: today’s Shamsi date in Latin digits.

🤖 LLM filter: returns only domains with < 2 days to expire as strict JSON.

📣 Telegram notify: posts the JSON to your channel/group.

🧰 Extensible: add validators or send to Slack/Email/Sheets with a couple more nodes.

## Workflow Overview

Nodes (in order)

1. Schedule Trigger – fires daily at 09:00.
2. Get CSV File – reads /home/exfiles/domains.csv.
3. Extract CSV File – parses CSV → items.
4. Make a Table (Code) – turns rows into table_md (Markdown table).
5. Current Time (Code) – produces jalaliLatin (e.g., 1404/06/09).
6. Change Date Format – (optional/redundant) formats date; you can remove it.
7. Merge – combines date & table (combine by position).
8. Check Domains (OpenAI) – returns JSON array of domains expiring in < 2 days.
9. Validate – quick sanity check: looks for "Domain 1" and "Handler Account" in output.
10. Send Result (Telegram) – posts to your chat/channel.

Tip: Replace the If node with a tiny Code validator to parse and verify the JSON strictly (see “Hardening” below).

## Requirements

- n8n (self-hosted or Cloud)
- OpenAI API credential (the workflow uses the @n8n/n8n-nodes-langchain.openAi node; model: gpt-5-mini in the sample)
- Telegram Bot token + chat ID (channel/group)
- The CSV file path inside the n8n runtime (/home/exfiles/domains.csv).
- If you run n8n in Docker, bind-mount the host folder to this path.

## Output Example

The OpenAI node is instructed to return only JSON:
```
[
  {
    "Domain 1": "example.com",
    "Expire Date": "1404/06/12",
    "Days Left to Expiration": 1,
    "Handler": "Namecheap",
    "Handler Account": "example@gmail.com"
  }
]
```
