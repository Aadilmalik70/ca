# GST Safe MVP Technical Specification

## 1. Service boundaries
- **Frontend (React/TypeScript):** upload flow, exception queue, actioning UI, status dashboard
- **API Service (NestJS):** auth, upload orchestration, reporting endpoints, audit APIs
- **Matching Service (FastAPI):** normalization pipeline, match engine, explanation and scoring jobs
- **Data stores:** PostgreSQL (transactional), S3-compatible storage (raw uploads), Redis/BullMQ (jobs)

## 2. Core API endpoints (v1)

### Upload and processing
- `POST /api/v1/uploads/purchase-register`
- `POST /api/v1/uploads/gstr2b`
- `GET /api/v1/uploads/{uploadId}/status`

### Reconciliation
- `POST /api/v1/reconciliation/{clientId}/{period}/run`
- `GET /api/v1/reconciliation/{clientId}/{period}/summary`
- `GET /api/v1/reconciliation/{clientId}/{period}/exceptions`

### Workflow and audit
- `POST /api/v1/exceptions/{exceptionId}/actions`
- `GET /api/v1/exceptions/{exceptionId}/audit`
- `POST /api/v1/periods/{clientId}/{period}/lock`

### Insights
- `GET /api/v1/vendors/{clientId}/scores`
- `GET /api/v1/status/{clientId}/{period}`
- `GET /api/v1/reports/{clientId}/{period}/itc-summary`

## 3. Canonical invoice schema (normalized)

| Field | Type | Required | Notes |
|---|---|---:|---|
| source_type | enum | yes | `books` or `gstr2b` |
| client_id | uuid | yes | tenant-scoped |
| period | text | yes | format `YYYY-MM` |
| vendor_gstin | text | yes | normalized uppercase |
| vendor_name | text | no | optional in 2B |
| invoice_no | text | yes | stripped and uppercased |
| invoice_date | date | yes | ISO date |
| taxable_value | numeric(14,2) | yes | non-negative |
| cgst | numeric(14,2) | no | default 0 |
| sgst | numeric(14,2) | no | default 0 |
| igst | numeric(14,2) | no | default 0 |
| cess | numeric(14,2) | no | default 0 |
| total_tax | numeric(14,2) | yes | computed |
| raw_row_ref | text | yes | line index for traceability |

## 4. Matching rules

### Deterministic rules (priority order)
1. Exact key match: `vendor_gstin + invoice_no + taxable_value + total_tax`
2. Amount tolerance match: same key with amount delta <= ₹1
3. Period drift candidate: same key found in adjacent period (N-1/N+1)

### Fuzzy rules (executed on unmatched)
- Candidate window by same GSTIN and similar invoice number (Levenshtein threshold)
- Date gap score (0-30 day tolerance)
- Amount delta score (% difference)
- Composite confidence score = weighted sum(date, amount, invoice similarity)

## 5. Exception taxonomy
- `missing_in_2b`
- `missing_in_books`
- `value_mismatch`
- `period_mismatch`
- `credit_note_mismatch`
- `validation_error`

Each exception stores:
- `risk_amount`
- `confidence`
- `explanation_template_key`
- `recommended_action`

## 6. Data model (table list)
- `clients`
- `users`
- `filing_periods`
- `uploads`
- `normalized_invoices`
- `match_results`
- `exceptions`
- `exception_actions` (append-only)
- `vendor_scores`
- `status_snapshots`
- `kpi_events`

## 7. Security model
- JWT auth with tenant-scoped RBAC roles: `ca_admin`, `ca_reviewer`, `founder_viewer`, `platform_admin`
- Raw files encrypted in storage (SSE)
- DB encryption-at-rest enabled
- No GST portal credentials persisted
- Audit logging for lock/unlock and approval actions

## 8. Observability and SLOs
- Processing latency SLO: 95% runs complete within 10 minutes for 25k invoices
- API availability target: 99.5% during business hours
- Error budget tracking per service
- Required metrics:
  - upload success/failure rate
  - normalization validation error rate
  - auto-match rate and mismatch corrections
  - action throughput and lock completion

## 9. Release readiness checklist
- Schema migrations reversible and tested
- Seed dataset acceptance test run and stored artifact
- Security checklist completed
- Pilot CA UAT signed-off
- Incident rollback procedure documented
