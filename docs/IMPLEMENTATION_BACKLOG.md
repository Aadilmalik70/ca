# GST Safe MVP Implementation Backlog

## Planning assumptions
- Team: 1 backend engineer, 1 AI engineer, 1 frontend engineer, 1 PM, 1 CA advisor
- Sprint length: 2 weeks
- Pilot target: 5 CA firms by day 90

## Epic E1 — File Ingestion and Processing Pipeline
**Outcome:** Reliable upload and asynchronous processing for purchase register and GSTR-2B files.

### Stories
- **E1-S1:** Upload endpoint for purchase register files (CSV/XLSX)
  - Acceptance criteria:
    - File validates MIME and size constraints
    - Upload metadata saved with status `uploaded`
    - S3 object key generated with tenant + period namespacing
- **E1-S2:** Upload endpoint for GSTR-2B files (JSON/XLSX)
  - Acceptance criteria:
    - JSON schema validation error details returned for malformed data
    - Upload metadata and original filename persisted
- **E1-S3:** Queue orchestration for processing jobs
  - Acceptance criteria:
    - Upload triggers a single idempotent processing job
    - Failed jobs move to retry with exponential backoff
- **E1-S4:** Processing status API
  - Acceptance criteria:
    - UI can poll status: `uploaded`, `processing`, `completed`, `failed`
    - Last error field exposed for failures

## Epic E2 — Normalization and Validation
**Outcome:** Canonical invoice schema across all supported input formats.

### Stories
- **E2-S1:** Canonical schema mapper
  - Acceptance criteria:
    - Input variants mapped into `normalized_invoices` table
    - GSTIN uppercased and whitespace removed
- **E2-S2:** Field-level validation engine
  - Acceptance criteria:
    - Required fields validated: GSTIN, invoice_no, invoice_date, taxable_value
    - Validation errors stored with row number and field
- **E2-S3:** Validation report API
  - Acceptance criteria:
    - API returns error summary counts and downloadable detail rows

## Epic E3 — Matching and Exception Engine
**Outcome:** High-precision deterministic matching with fuzzy fallback.

### Stories
- **E3-S1:** Deterministic matcher v1
  - Acceptance criteria:
    - Match key uses GSTIN + normalized invoice_no + amount thresholds
    - Produces match state: `exact_match`, `candidate`, `unmatched`
- **E3-S2:** Exception taxonomy engine
  - Acceptance criteria:
    - Exceptions emitted using fixed taxonomy constants
    - Each exception includes amount impact and period tag
- **E3-S3:** Fuzzy fallback matcher
  - Acceptance criteria:
    - Candidate scoring uses date gap and amount difference
    - Confidence value from 0 to 1 returned for each candidate

## Epic E4 — CA Review, Approvals, and Audit
**Outcome:** Exception-first review workflow with immutable actions.

### Stories
- **E4-S1:** Exception queue UI
  - Acceptance criteria:
    - Table supports filters by type, amount band, vendor
    - Sorting by highest tax-risk amount
- **E4-S2:** Approval actions API
  - Acceptance criteria:
    - Supported actions: `approve_itc`, `defer_itc`, `reject_claim`
    - Action requires reason code and optional comments
- **E4-S3:** Immutable audit trail
  - Acceptance criteria:
    - Append-only action records with actor, timestamp, and previous state
    - No update/delete endpoint for audit records
- **E4-S4:** Period lock
  - Acceptance criteria:
    - Locked month blocks any further action changes
    - Admin unlock path logged with reason

## Epic E5 — Explainability, Vendor Scoring, and Founder Signals
**Outcome:** Business-readable risk outputs and alerts.

### Stories
- **E5-S1:** Explanation template engine
  - Acceptance criteria:
    - Each exception gets a concise explanation and recommended action
    - Explanation includes specific mismatch fields
- **E5-S2:** Vendor reliability scoring
  - Acceptance criteria:
    - Score formula documented and deterministic
    - Six-month trend computed per vendor GSTIN
- **E5-S3:** Founder status computation
  - Acceptance criteria:
    - Status rules based on at-risk ITC and open exception count
    - Returns `green`, `amber`, or `red` with summary text
- **E5-S4:** Pre-filing alert dispatcher
  - Acceptance criteria:
    - Alert event triggered when status transitions to amber/red
    - Channel adapters designed for email now, WhatsApp later

## Epic E6 — Pilot Operations and Analytics
**Outcome:** Pilot-ready governance and KPI measurement.

### Stories
- **E6-S1:** KPI event logging
  - Acceptance criteria:
    - Track match rate, false mismatch adjustments, review completion time
- **E6-S2:** Pilot dashboard (internal)
  - Acceptance criteria:
    - Weekly per-client KPI view and notes
- **E6-S3:** Onboarding checklist artifact
  - Acceptance criteria:
    - Standardized checklist template and completion status for each pilot

## Sprint plan (2-week cadence)
- **Sprint 1:** E1-S1..S4, E2-S1
- **Sprint 2:** E2-S2..S3, E3-S1..S2
- **Sprint 3:** E3-S3, E4-S1..S2
- **Sprint 4:** E4-S3..S4, E5-S1
- **Sprint 5:** E5-S2..S4, E6-S1
- **Sprint 6:** E6-S2..S3, stabilization, pilot enablement

## Definition of Done (DoD)
A story is done only when:
1. Acceptance criteria are testable and verified
2. API contracts are documented
3. Logs/metrics are added where relevant
4. Security and access checks are implemented
5. Demoed to PM + CA advisor for domain sign-off
