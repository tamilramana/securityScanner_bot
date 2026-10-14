# securityScanner

An n8n workflow that passively scans a website's JavaScript files for patterns that look like exposed secrets, classifies the findings with AI, and sends a report to Telegram.

> **Authorized use only.** Scan only websites you own or have written permission to test. Do not use this on third-party targets.

## What it does

1. **Manual Trigger** starts a run.
2. **Target Config** holds the target URL.
3. **Fetch Homepage** downloads the page HTML.
4. **Extract JavaScript URLs** finds `<script src>` links and keeps only same-host files.
5. **Fetch JavaScript Files** downloads each approved JS file.
6. **Detect Candidate Secrets** matches the content against secret patterns, such as API keys and tokens.
7. **AI Agent (Google Gemini)** classifies each finding.
8. **Telegram - Report** sends three messages: a summary, the findings, and any failed requests.

It is passive. It only makes normal GET requests, it doesn't test or use any credential it finds, and it doesn't follow redirects.

## Requirements

- An n8n instance (cloud or self-hosted)
- A Telegram bot (create one with @BotFather) and your chat ID
- A Google Gemini API key
- A target you are allowed to scan, for example a local lab page

## Setup

1. In n8n, go to **Workflows -> Import from file** and select `workflow/ngp-059-passive-js-secret-scanner.json`.
2. Add credentials:
   - **Telegram API**: your bot token
   - **Google Gemini (PaLM) API**: your API key
3. Open **Target Config** and set `targetUrl` to your target.
4. Open **Telegram - Report** and set your chat ID.
5. Click **Execute workflow**.

## Example report

```
Passive JS Secret Scanner
Tag: NGP-059
Target: example.com

JS files approved: 4
Fetched OK: 3
Failed requests: 1

Candidate secrets detected: 8
```

## Local demo lab

To demo it safely, host a small page with fake credentials (for example "NGP-059 Local JS Lab") and expose it with a temporary tunnel:

```
cloudflared tunnel --url http://localhost:<port>
```

Quick-tunnel URLs change on every restart. Update `targetUrl` each time.

## Troubleshooting

| Message | Meaning |
|---|---|
| `ENOTFOUND` | The domain doesn't exist. The tunnel is stopped or the URL is wrong. |
| `HTTP 530` | The tunnel hostname exists, but it can't reach your local server. |
| `host not allowed` | The JS file is on a different host, so it is skipped on purpose. |
| `ECONNRESET` on Telegram | A temporary network error. The node retries automatically. |

## Limitations

- Pattern matching can produce false positives. The AI step only helps prioritize them.
- It scans only the homepage and the JS files linked from it.

## License

MIT
