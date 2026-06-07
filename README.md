# Babud

**Self-Hosted Personal Finance Automation for Busy Families**  
*Bank + Budget = Babud. Envelope budgeting, intelligent transaction tagging, and zero-manual-entry insights — powered by real Australian bank data (Up Bank), n8n automation, and a secure family dashboard.*

## Badges
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16+-336791?logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![n8n](https://img.shields.io/badge/n8n-Automation-FF6A00?logo=n8n&logoColor=white)](https://n8n.io)
[![Up Bank API](https://img.shields.io/badge/Up%20Bank-Integrated-00A8E8)](https://developer.up.com.au)
[![TypeScript](https://img.shields.io/badge/TypeScript-Ready-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## About

Babud ("Bank + Budget") is a **privacy-first, self-hosted personal finance system** built for busy families using **Up Bank** (Australia's leading neobank). It converts raw bank transactions into clean, actionable envelope-style budgets with almost zero manual effort.

### The Problem It Solves
- Manual tagging and reconciliation is exhausting
- Native bank apps lack true envelope budgeting and shared family visibility
- Cloud tools (YNAB, PocketGuard, etc.) share your data and charge monthly subscriptions

### The Babud Solution
A lightweight, fully self-controlled stack running on a single Linux VPS:

- **n8n** orchestrates transaction sync, intelligent 3-state tagging queues, and Google Sheets → PostgreSQL budget sync
- **PostgreSQL** is the single source of truth with a normalised schema (transactions + many-to-many envelopes)
- **Metabase** delivers variance reports — budget vs. actual, YTD, overspend drill-downs

**Guiding principle:** "Less is more" — powerful automation, minimal maintenance, zero bloat.

---

## Architecture

```
Up Bank API ──webhooks──▶ n8n (BBX_Main_ReceiveWebhooks)
                                │
                                ▼
                         PostgreSQL DB ◀── n8n (BBX_Main_Daily / BBX_Main_Weekly)
                                │                        │
                                │                  Google Sheets
                                │                  (budget input)
                                ▼
                           Metabase
                           (dashboards)
```

### Data Flow
1. **Real-time:** Up Bank fires a webhook on every new transaction → n8n decrypts, classifies, and inserts into PostgreSQL, then queues auto/manual tagging
2. **Daily:** n8n polls for tag changes, validates and syncs Up Bank tags, runs auto-tagging rules, and matches internal transfers
3. **Weekly:** n8n polls the Up API for missed/unsettled transactions, resolves them, and syncs the envelope budget from Google Sheets into the database

---

## Key Features

| Feature | Detail |
|---|---|
| Real-time ingestion | Up Bank webhook + smart polling fallback |
| 3-state tagging queue | `auto_tag` → `manual_tag` → `notify` with atomic safety |
| Envelope budgeting | Edit budgets in Google Sheets → daily UPSERT to PostgreSQL |
| Metabase dashboards | Monthly variance, YTD breakdown, overspend drill-down |
| Lightweight deployment | Docker + Traefik + UFW, runs on a tiny VPS (<500 MB RAM) |
| Full data ownership | Up Bank personal-use policy compliant, no third-party cloud |

---

## Workflow Reference

Three n8n workflow files are included in `main-workflows/`:

### `BBX_Main_ReceiveWebhooks.json`
The real-time core. Triggered by Up Bank webhook calls.
- Decrypts the signed webhook payload (`BBX_Sub_WebhookDecrypt`)
- Classifies transaction type and routes to the correct sub-workflow
- Inserts/updates transactions in PostgreSQL (`BBX_Sub_UpBankToPostgres`)
- Queues auto-tagging and notification management
- Handles duplicate-safe deletion and unsettled transaction updates

### `BBX_Main_Daily.json`
Runs on a schedule (daily). Keeps the database and Up Bank tags in sync.
- Fetches recently updated transaction IDs from Up Bank
- Validates tag consistency between Up Bank and PostgreSQL
- Syncs approved tags back to Up Bank
- Runs the auto-tagger against untagged transactions
- Matches and flags internal Up Bank transfers

### `BBX_Main_Weekly.json`
Runs on a schedule (weekly). Catches anything the webhook missed.
- Polls the Up Bank API for unsettled and recently settled transactions
- Upserts any gaps into PostgreSQL
- Syncs the latest envelope/budget data from Google Sheets into the database

---

## Installation (Self-Hosting)

### Prerequisites

| Component | Version | Notes |
|---|---|---|
| Linux VPS | Ubuntu 22.04+ | Hostinger VPS 1 or equivalent works fine |
| Docker + Docker Compose | Latest stable | Traefik for reverse proxy |
| PostgreSQL | 16+ | Native on host or containerised |
| n8n | Latest | Self-hosted via Docker |
| Up Bank personal access token | — | [developer.up.com.au](https://developer.up.com.au) |
| Google Sheets API credentials | Optional | Only for budget sync workflow |
| Metabase | Latest | Community Edition, self-hosted |

### Quick Setup

1. **Clone the repo**
   ```bash
   git clone https://github.com/sevasek/babud.git
   cd babud
   ```

2. **Configure your environment**
   - Copy your Up Bank personal access token into n8n credentials
   - Set up a PostgreSQL database and note the connection string
   - (Optional) Create a Google Sheets service account for budget sync

3. **Import n8n workflows**
   - Open your n8n instance → Settings → Import from file
   - Import each JSON from `main-workflows/` in this order:
     1. Sub-workflows (if available from the companion backend repo)
     2. `BBX_Main_ReceiveWebhooks.json`
     3. `BBX_Main_Daily.json`
     4. `BBX_Main_Weekly.json`

4. **Configure Up Bank webhooks**
   - In Up Bank developer portal, register your n8n webhook URL
   - Format: `https://your-domain.com/webhook/<your-webhook-id>`

5. **Connect Metabase**
   - Point Metabase to your PostgreSQL instance
   - Import the included dashboard templates from `docs/`

6. **Activate workflows**
   - Enable all three main workflows in n8n
   - Verify with a test transaction or the Up Bank sandbox

---

## Demo / Preview

**Metabase Reports:**
- Monthly envelope variance (Actual vs Budget)
- YTD income/expense breakdown
- Overspent drill-down tables with transaction links

*(Screenshots will be added to `docs/screenshots/` — contributions welcome)*

---

## Roadmap

- [ ] React/TypeScript family dashboard with offline support
- [ ] Swipe-to-categorise UX on any transaction
- [ ] "Reward You" positive behaviour summary (e.g., under-spend notifications)
- [ ] Telegram/email weekly digest as an alternative to Metabase pull reports
- [ ] Multi-tenancy support for shared household accounts
- [ ] Local LLM-powered merchant name normalisation (Ollama integration)

---

## Known Issues / Limitations

- Currently personal-use only — no multi-family tenancy yet
- Up Bank API is read-mostly (tags/notes only); transfers still managed inside the Up app
- Sub-workflows are in the companion `babud-backend` repo (private) — public release planned

---

## License
This project is licensed under the MIT License — see LICENSE for details.

---

## Acknowledgments
- [Up Bank](https://up.com.au) — excellent developer API and webhook support
- [n8n community](https://community.n8n.io) — powerful self-hosted automation engine
- [Metabase](https://metabase.com) — beautiful BI accessible to non-engineers

## Support & Contact
- **Discussions:** Use GitHub Discussions for questions or ideas
- **Portfolio / Collaboration:** [@sevasek on X](https://x.com/sevasek)
