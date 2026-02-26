# GST Safe Pilot Execution Runbook

## Objective
Run a controlled 90-day pilot with CA firms and validate:
- 60% reduction in reconciliation effort
- >=90% pre-filing issue detection
- <5% false mismatch rate

## Pilot design
- **Participants:** 5 CA firms, 5-10 clients each in phase 1
- **Pilot length:** 1 full filing cycle minimum (expand to 3 cycles)
- **Data input:** sanitized purchase register + GSTR-2B extracts with consent

## Onboarding steps
1. Execute consent checklist and DPA acknowledgement
2. Collect sample files for one historical month
3. Configure client profile and filing period
4. Run baseline reconciliation and record manual benchmark time
5. Conduct CA training (60 minutes) on exception workflow

## Weekly operating cadence
- **Monday:** process latest data and share KPI snapshot
- **Wednesday:** CA feedback call (30 minutes)
- **Friday:** triage and release fixes; publish change log

## KPI scorecard template
- Client count onboarded
- Invoice volume processed
- Auto-match rate
- False mismatch rate
- CA review completion time
- ITC at-risk value identified pre-filing
- Open defects by severity

## Escalation policy
- **P1:** incorrect reconciliation causing wrong ITC recommendation
  - Response target: 4 hours
  - Mitigation: disable automated recommendation, force manual review mode
- **P2:** processing delays > SLO
  - Response target: same business day
- **P3:** reporting/UI defects without compliance impact
  - Response target: next sprint

## Exit criteria for paid rollout
1. At least 3 pilot CAs confirm measurable time savings
2. False mismatch rate remains below 5% for 2 consecutive periods
3. No unresolved P1 issues at pilot close
4. Founders report status signals as actionable in monthly filing prep
