# efood Partner Support Agent

Multi-agent conversational system that lets efood restaurant and
store partners manage their storefront through natural language
chat, without logging into a backend dashboard.

Live demo: https://giamarigkos-del.github.io/efood-demo-v2/

## What It Does

Partners authenticate with a 7-digit Vendor ID, matched against a
live Google Sheets database to pull store name, status, health
check score, active coupons, and average review score.

Once authenticated, partners can:

- Toggle store status (open/closed) in real time
- Update item prices, item names, opening hours, and product
  availability
- Run bulk operations across multiple stores under the same account
  (Master Flow)
- Trigger a callback request that automatically creates a task in
  Asana, tagged with store name, vendor ID, and issue summary

Every conversation is logged for QA and reporting purposes. Cross-store
data isolation is enforced so a partner only ever sees their own
store's data, even when the same phone number or session is reused
across stores.

## Architecture

Three-agent orchestration pattern, not a single monolithic prompt:

- **Orchestrator Agent** — handles intent routing and Vendor ID
  authentication, decides which downstream agent should handle the
  request
- **Support Agent** — handles account, billing, technical, and
  operational queries; owns the callback/escalation flow into Asana
- **Catalog Agent** — handles all product-level changes (price,
  name, hours, availability), including cross-store bulk edits

Agents transition silently between each other. No "transferring you
to another agent" messages are shown to the partner, so the
multi-agent structure is invisible from the user side.

## Data Layer

- **Database:** Google Sheets, two tabs — Stores (VendorID,
  StoreName, Status, HealthCheck, Coupons, AvgScore) and Reports
  (append-only conversation log)
- **Session bridging:** a Google Apps Script endpoint writes the
  authenticated Vendor ID into a Session sheet cell before the
  Botpress webchat loads, since Botpress variables can't reliably
  persist Vendor ID across node transitions on their own
- **Task escalation:** Asana API, tasks created programmatically
  with store context pre-filled

## Tech Stack

| Layer | Tool |
|---|---|
| Conversational engine | Botpress v3.6 (multi-agent flow builder) |
| LLM | Claude Sonnet 4.6 |
| Database | Google Sheets + Google Apps Script |
| Task management | Asana API |
| Frontend | Static HTML/CSS/JS, GitHub Pages |
| Live updates | Client-side polling every 4s, cache-busting to bypass GitHub Pages CDN caching |

## Security

Prompt-level anti-injection handling and a blocklist of internal
system terms (backend tool names, internal platform names) that the
agent is instructed never to surface, regardless of how the user
phrases a request.

## Status

Functional demo, tested against 5 mock partner accounts spanning 3
store categories, including one multi-store account to validate the
bulk-edit Master Flow. Presented internally; not in production.