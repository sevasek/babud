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

## About

Babud ("Bank + Budget") is a **complete, privacy-first, self-hosted personal finance system** built for busy families. It turns raw bank transactions from **Up Bank** (Australia’s leading neobank) into clean, actionable envelope-style budgets with almost zero manual effort.

### The Problem It Solves
- Manual tagging and reconciliation is exhausting.
- Native bank apps lack true envelope budgeting and family-shared visibility.
- Cloud tools (YNAB, PocketGuard, etc.) share your data and charge monthly fees.

### The Babud Solution
A lightweight, fully controlled stack on a single Linux VPS:
- **n8n** automates transaction sync, smart tagging queues (auto/manual/notify), and one-way Google Sheets → Postgres budget sync.
- **PostgreSQL** is the single source of truth with a normalised schema (transactions + many-to-many envelopes).
- **Metabase** delivers beautiful variance reports (budget vs actual, YTD, overspend drill-downs).

**“Less is more”** is the guiding principle — powerful automation with minimal maintenance and no bloat.

### Key Features
- Real-time(ish) Up Bank transaction ingestion via webhooks + smart polling
- Intelligent 3-state tagging queues (`ps_queue_auto_tag`, `ps_queue_manual_tag`, `ps_queue_notify`) with atomic safety
- Hybrid envelope budgeting: edit in familiar Google Sheets → daily UPSERT to DB
- Production-grade Metabase dashboards with variance calculations
- Docker + Traefik + UFW-ready deployment on a tiny VPS (<500 MB RAM)
- Full data ownership and Up Bank personal-use policy compliance

## Demo / Preview

**Metabase Reports** (examples):
- Monthly envelope variance (Actual vs Budget)
- YTD income/expense breakdown
- Overspent drill-down tables with transaction links

*(Screenshots and a short demo video will be added to the repo shortly — watch this space!)*

## Installation (Self-Hosting)

### Prerequisites
- Linux VPS (Ubuntu 22.04+ recommended, Hostinger or equivalent)
- Docker + Docker Compose
- PostgreSQL (native on host or containerised)
- Up Bank personal access token
- Google Sheets API credentials (optional for budget sync)

## Known Issues / Limitations

- Currently personal-use only — no multi-family tenancy yet
- Up Bank API is read-mostly (tags/notes only) — transfers still handled in-app

## Roadmap
- React/TypeScript frontend with offline support
- Dashboard experience: Swipe left on any transaction  → assign envelope/category
- Instant “Reward You” summary for positive behaviours

## License
This project is licensed under the MIT License — see LICENSE for details.

## Acknowledgments
- Up Bank for the excellent developer API
- n8n community for the powerful self-hosted automation engine
- Metabase for making beautiful BI accessible

## Support & Contact
- Discussions: Use GitHub Discussions for questions or ideas
- Portfolio / Collaboration: Reach out via X @sevasek