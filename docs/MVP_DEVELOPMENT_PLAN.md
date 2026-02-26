# GST Safe MVP Development Plan

## Goal
Deliver a CA-first MVP for GST pre-filing ITC verification in 90 days with measurable pilot outcomes:
- Reduce CA reconciliation effort by 60%
- Detect at least 90% of ITC issues before filing
- Keep false mismatches under 5%

## MVP Scope
### In scope
1. File ingest: Purchase register (CSV/Excel export) and GSTR-2B (JSON/Excel)
2. Normalization and validation (GSTIN, invoice number/date, tax breakdown)
3. Matching engine (deterministic first, fuzzy fallback)
4. Exception queue with categories and priority
5. CA approval workflow with immutable audit logs
6. Vendor reliability score (rolling 6-month view)
7. Founder status signal (Green/Amber/Red) with pre-filing alert
8. Monthly summary report: ITC safe vs ITC at risk

### Out of scope (MVP)
- GST filing automation to portal
- Auto-posting to accounting systems
- Full ERP modules (payroll/inventory/forecasting)

## Development Phases

## Phase 0: Foundations (Week 0-1)
**Outcome:** Working repo standards + baseline architecture

- Finalize domain model and MVP acceptance criteria
- Define data contracts for purchase register and GSTR-2B normalized schema
- Bootstrap services:
  - `frontend/` (React + TypeScript)
  - `services/api/` (NestJS)
  - `services/matching/` (FastAPI)
- Set up CI checks (lint + build)
- Set up infra skeleton (S3 bucket naming strategy, Postgres schema migration tooling, Redis queue)

### Deliverables
- Architecture decision record (ADR)
- Initial DB schema migration
- API spec draft for upload/matching/exceptions/approvals

## Phase 1: Core Workflow (Week 2-4)
**Outcome:** End-to-end deterministic reconciliation pipeline

- Implement upload API and file persistence metadata
- Build parsing + normalization pipeline
- Implement deterministic matching logic:
  - Exact match: GSTIN + invoice number + taxable/tax values
  - Tolerance rules for minor numeric variance
- Generate exception records by type
- Build basic UI flow:
  - Upload files
  - View exception list
  - Filter and sort by impact
- Implement CA approve/defer actions with append-only audit records

### Deliverables
- E2E demo from upload to exception actions
- Baseline metrics instrumentation for match rate and exception volume

## Phase 2: Intelligence Layer (Week 5-8)
**Outcome:** Better detection quality and explainability

- Add fuzzy matching strategy (invoice/date/value proximity)
- Add prioritization score (amount, recurrence, vendor behavior)
- Implement explanation generator templates for each exception type
- Build vendor reliability scorer (6-month weighted behavior)
- Add founder status signal and threshold rules
- Set up notification hook for pre-filing alerts (email/WhatsApp placeholder)

### Deliverables
- Explainable exception cards
- Vendor reliability dashboard widget
- Green/Amber/Red summary banner

## Phase 3: Pilot Readiness (Week 9-12)
**Outcome:** Controlled pilot with operational readiness

- Hardening:
  - Performance improvements for multi-client CA firms
  - RBAC tightening and audit verification
  - Data retention and export policies
- Onboarding pack for pilot CAs
- Pilot operations dashboard for weekly KPI tracking
- Feedback loop process and issue triage

### Deliverables
- Pilot-ready release candidate
- Pilot runbook and KPI scorecard

## Workstreams and Owners

| Workstream | Owner Role | Core Responsibilities |
|---|---|---|
| Product & Compliance | PM + CA Advisor | Scope, legal checklist, acceptance criteria |
| Platform/API | Backend Engineer | Upload, orchestration, auth, audit logs |
| Matching & AI | AI Engineer | Matching, fuzzy logic, explanations, scoring |
| Frontend UX | Frontend Engineer | Exception queue, approvals, status views |
| DevOps/Security | Shared | Infra, CI/CD, observability, backup policies |

## KPIs and Operational Metrics
Track weekly:
- Clients onboarded (pilot)
- Invoice auto-match rate (%)
- False mismatch rate (%)
- CA reconciliation time saved (self-reported)
- ITC at-risk value detected pre-filing (₹)
- CA CSAT score

## Quality Gates
- Deterministic matching acceptance test passes on seeded samples
- False mismatch <5% on pilot benchmark data set
- All approval actions auditable and immutable
- Security checklist complete (no GST credentials stored)

## Risks and Mitigations
1. **Data quality variation in client exports**
   - Mitigation: robust normalization + validation feedback to user
2. **High false positives in early versions**
   - Mitigation: deterministic-first strategy and tolerance tuning
3. **Slow CA adoption due to workflow change**
   - Mitigation: exception-only UI and CA-led onboarding sessions
4. **Compliance concerns from clients**
   - Mitigation: consent templates and strict data handling policy

## Immediate Next 10 Tasks
1. Finalize normalized invoice schema and version it
2. Implement parser interfaces for CSV and JSON
3. Create matching result schema (`matched`, `exception_type`, `confidence`)
4. Add exception taxonomy constants shared across services
5. Implement upload API + async job enqueue
6. Build deterministic matcher v1
7. Create exception queue UI table with filters
8. Implement approve/defer endpoints and immutable audit trail table
9. Add KPI event logging hooks
10. Prepare pilot onboarding checklist + sample data package
