# Product Requirements Document (PRD)

## Product
GST Safe — AI Compliance Agent for pre-filing ITC assurance

## Background
CA teams lose significant time reconciling books against GSTR-2B every month. Issues such as missing vendor filings, invoice mismatches, and period differences create filing-time surprises and ITC risk.

## Problem Statement
Current accounting workflows identify many GST reconciliation issues too late. CAs need an explainable, workflow-compatible system that surfaces high-risk exceptions early and supports controlled approval before filing.

## Target Users
### Primary
- Chartered Accountant firms managing multiple SMB clients
- Outsourced accounting providers

### Secondary
- SMB founders/finance owners requiring filing readiness visibility

## Objectives
1. Cut monthly CA reconciliation time by at least 60%
2. Detect at least 90% of ITC issues pre-filing
3. Maintain mismatch false-positive rate below 5%
4. Enable human-approved ITC decisions with audit history

## Functional Requirements

### FR1 — Data Ingestion
- Upload purchase register files (CSV/Excel)
- Upload GSTR-2B files (JSON/Excel)
- Persist file metadata and processing status

### FR2 — Normalization and Validation
- Normalize GSTIN format and invoice fields
- Validate required values: GSTIN, invoice number/date, taxable value, tax amounts
- Produce row-level validation errors

### FR3 — Matching Engine
- Deterministic matching by GSTIN + invoice number + amount checks
- Fuzzy matching fallback for near-match candidates
- Assign confidence and reasoning tags

### FR4 — Exception Management
- Classify exceptions:
  - Missing in GSTR-2B
  - Missing in books
  - Value mismatch
  - Period mismatch
  - Credit note mismatch
- Prioritize by monetary risk and recurrence

### FR5 — Explainability
- Generate concise, CA-grade explanation text per exception
- Show recommended next action (review vendor, defer ITC, request revised invoice)

### FR6 — Approval Workflow
- CA can approve/defer/reject exceptions
- Month lock supported after review completion
- All actions written to immutable audit trail

### FR7 — Vendor Reliability
- Calculate vendor reliability score over a rolling 6 months
- Show trend and top-risk vendors

### FR8 — Founder Status and Alerts
- Show Green/Amber/Red status for filing readiness
- Trigger pre-filing alert if at-risk ITC crosses configured threshold

### FR9 — Reporting
- Monthly summary: ITC safe vs ITC at risk
- Export exception and decision report

## Non-Functional Requirements
- Security: encryption in transit and at rest
- Privacy: no GST credential storage
- Reliability: resumable processing jobs and clear failure states
- Performance: support CA workflows across multiple clients/monthly batches
- Auditability: append-only action history

## User Flows
1. CA uploads purchase register and GSTR-2B
2. System validates and normalizes records
3. Matcher produces matched/unmatched/exception output
4. CA reviews prioritized exceptions and explanations
5. CA approves/defer decisions and locks period
6. Founder sees status signal and summary

## Data Entities (MVP)
- `clients`
- `filing_periods`
- `uploads`
- `normalized_invoices`
- `match_results`
- `exceptions`
- `approval_actions`
- `vendor_scores`
- `status_snapshots`

## Acceptance Criteria
1. Given valid files, system produces reconciliation output with <5% false mismatches on benchmark dataset
2. CA can perform approve/defer action on any exception and view immutable action history
3. System generates monthly ITC summary and filing status signal
4. Vendor score calculation runs successfully for each filing period with prior-month data

## Success Metrics
- Reconciliation time reduction (%)
- Auto-match rate (%)
- False mismatch rate (%)
- Pre-filing issue detection coverage (%)
- Pilot CA CSAT

## Release Plan
### Milestone 1 (30 days)
Upload + normalization + deterministic matching + basic exception UI + approvals

### Milestone 2 (60 days)
Fuzzy matching + explainability + vendor scoring + founder status alerts

### Milestone 3 (90 days)
Pilot hardening + analytics dashboard + onboarding assets

## Dependencies
- Pilot CA data availability with consent
- Baseline sample datasets for acceptance testing
- Cloud infra (S3-compatible storage, Postgres, Redis)

## Open Questions
1. Required tolerance bands for amount/date fuzzy matching?
2. Should month lock be irreversible or admin-unlockable?
3. Preferred founder alert channel at MVP (email vs WhatsApp)?
4. Do pilot CAs need white-label branding in first release?
