# NDNT Accounting Automation

An automated accounting department for Vietnamese small and medium companies.
It receives accounting documents, reads them, validates them, routes them to a
human review layer, and generates import files for MISA (Vietnam's widely used
accounting software).

**Status: live in production since November 2026**, running 24/7 and serving
real clients, including a hospitality group with five operational zones.

> This public repository is a showcase: architecture, design principles and
> roadmap only. The production code is private because the system processes
> client accounting data. No client data, credentials, or infrastructure
> details are published here.

## What it does

1. **Intake** — Supplier invoices and receipts arrive by email and through a
   Telegram bot (photos, PDFs, albums). An album is treated as one multi-page
   document; separate photos are separate documents.
2. **Reading** — XML invoices and text-based PDFs are parsed by deterministic
   code only. Scans and handwritten documents go to an AI-vision router
   (primary provider plus free-tier failovers) that returns one fixed JSON
   schema with per-field confidence scores.
3. **Validation** — Totals are reconciled, amounts are cross-checked against
   the amount in words, tax codes are verified, and each document is
   classified by company and business zone. Conflicts or unknown senders go
   to an exception queue — the system never guesses.
4. **Review** — Extracted data lands in a structured review sheet. An AI
   accountant re-checks flagged rows and classifies warehouse/item codes;
   humans approve before anything moves on.
5. **Output** — Deterministic code generates per-module import files in
   MISA's exact templates.
6. **Learning loop** — Human corrections feed a correction dictionary
   (seller-scoped first, promoted to global only after repeat confirmations),
   so the same reading mistake is not made twice.

## Safety design (human in the loop)

- The AI never files anything. It reads and proposes; accountants decide.
- Handwritten documents get 100% human review.
- Low-confidence fields are forced to null instead of guessed.
- Non-VAT receipts and transfer-bill-only expenses are handled as separate
  streams with their own reconciliation against bank outflows.
- Daily backups with checksum verification and a rehearsed restore procedure.

## Architecture

- **Runtime:** Linux VPS, Docker containers, workflow automation (n8n)
  driven by Python services.
- **Intake workflows:** webhook-based email and Telegram pipelines.
- **Messaging bridge:** a companion system forwards Zalo group messages to
  Telegram topics (and replies back), with a Python gateway owning the
  group-to-topic map, message deduplication, media download/re-upload, and
  a self-healing watchdog.
- **Data layer:** structured staging files and a review spreadsheet; the
  bridge keeps a local archive of every forwarded message.

## Repository structure (production, private — shown redacted)

```
intake-system/
  readers/        XML + text-PDF parsers, AI-vision router
  validators/     totals, tax-code and zone classification rules
  staging/        review-sheet shaped output
  corrections/    correction dictionary and promotion rules
bridge/
  gateway/        Python control service (mapping, dedupe, media)
  workflows/      message forwarding and command panels
infra/
  backup/         daily backup, checksum verify, restore kit
  monitoring/     health checks and watchdog
```

## Roadmap — where a Devin Bot would work

1. Harden the intake pipeline and build a full test suite for the parsers.
2. Bank-statement reconciliation for all three document streams.
3. Refactor the messaging bridge gateway for multi-account scale.
4. Monitoring and backup automation for an upcoming server migration.

Each workstream ships into the live codebase above, reviewed by a human
before deployment.
