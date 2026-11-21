# Devin Bot Brief — First Sprint

This is the working brief for the Devin Bot assigned to the NDNT Accounting
Automation roadmap. It exists so the Bot can start on day one: mission,
ground rules, and the first sprint's tasks with acceptance criteria.

## Mission

Own roadmap workstreams 1 and 2 for a production accounting-automation
system used by real clients in Vietnam:

1. Harden the document intake pipeline and build its test suite.
2. Add bank-statement reconciliation for all document streams.

## Ground rules

- Production code stays in a private repository. This public repo holds
  briefs and sanitized design notes only — never client data, credentials,
  or infrastructure details.
- Every change ships as a small pull request with tests. A human reviews
  and approves before anything deploys.
- The system is human-in-the-loop by design: AI output is always a
  proposal, never a filing. No task may weaken that rule.
- Deterministic code first: fixed rules are handled by code; AI is used
  only where judgment is genuinely needed.

## Sprint 1 — Intake hardening and parser tests

| # | Task | Acceptance criteria |
|---|------|---------------------|
| 1 | Test suite for the XML invoice parser | Covers valid invoices, malformed XML, missing tax codes, rounding edge cases; all tests pass in CI |
| 2 | Test suite for the text-PDF parser | Fixtures for multi-page invoices and amount-in-words cross-checks; mismatches raise, never silently pass |
| 3 | Confidence-score calibration for the vision reader | Low-confidence fields are forced to null; calibration report compares reader output against human-reviewed samples |
| 4 | Exception-queue hardening | Conflicting zone classifications and unknown senders land in the exception queue with a reason code; zero silent guesses |
| 5 | Intake observability | Structured logs and a daily health summary for the intake pipeline; failed documents are queued for rescan, never lost |

## Sprint 2 — Bank-statement reconciliation (preview)

- Match non-VAT receipts and transfer-bill-only expenses 1-to-1 against
  bank statement lines by amount, date and beneficiary.
- Unmatched lines surface in the review sheet with a reason, not a guess.
- Reconciliation report per company per month, ready for accountant sign-off.

## Definition of done

A task is done when its tests pass, the pull request is human-approved,
and the change is live in production without breaking the review flow.
