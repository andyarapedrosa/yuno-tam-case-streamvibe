# Section 2: Technical Action Plan

**Merchant:** StreamVibe
**Date:** November 25, 2024
**Incident:** Brazil card authorization rate drop — EBANX 3DS2 regression
**Prepared by:** Yuno TAM Team
**Audience:** Internal + StreamVibe (Claudia Mendez, Rafael Santos)

---

## Overview

This action plan addresses two parallel workstreams:

- **Workstream A — Active incident (Brazil):** Stabilize authorization rates while EBANX resolves the root cause, then restore optimal routing configuration with safeguards.
- **Workstream B — Structural hardening:** Implement changes that prevent this class of failure from recurring, regardless of which acquirer is involved.

Argentina is excluded from this plan as it requires a separate strategic discussion — not a technical incident response.

---

## Priority Matrix

| Priority | Action | Owner | Timeline | Expected Impact |
|----------|--------|-------|----------|-----------------|
| P0 | Keep dLocal as Brazil primary | Yuno | ✅ Done | Stabilizes AR at ~82.7% |
| P0 | Formal EBANX escalation with full evidence package | Yuno | 24h | Unblocks resolution timeline |
| P1 | Define EBANX reactivation criteria | Yuno + StreamVibe | 48h | Prevents premature rollback |
| P1 | EBANX: formal RCA + fix ETA | EBANX | 48–72h | Closes root cause loop |
| P2 | Soft decline retry logic for code 91 | Yuno | 1 week | Recovers retryable declines |
| P2 | Brazil routing split post-resolution | Yuno | Post-fix | Eliminates single-acquirer exposure |
| P3 | Acquirer health monitoring + auto-failover | Yuno | 2–3 weeks | Reduces detection time from days to hours |
| P3 | 3DS threshold review for Brazil | Yuno + StreamVibe | 2–3 weeks | Reduces unnecessary friction on annual plans |

---

## Workstream A: Active Incident Response

### A1 — Maintain dLocal as Brazil Primary [P0 — DONE]
**Owner:** Yuno
**Status:** ✅ Active
**Action:** dLocal is now handling 100% of Brazil card traffic. No StreamVibe engineering changes required.
**Expected impact:** Brazil card AR stabilizes at ~82.7% — equivalent to dLocal's current performance, recovering approximately **3,744 lost approvals/month** vs. current EBANX state.
**Note:** Do not revert to EBANX until reactivation criteria (A3) are formally met and validated with data.

---

### A2 — Formal EBANX Escalation [P0 — In Progress]
**Owner:** Yuno
**Timeline:** 24 hours
**Action:** Submit a formal P1 escalation to EBANX TAM/support with the following evidence package:

Evidence to include:
- Brazil card AR by acquirer: EBANX 54.6% vs. dLocal 82.7% (same period, same BINs)
- Decline code 91 trend: 412 → 3,825 (+829%), representing 62.8% of all EBANX declines
- Issuer BIN breakdown: all 5 major issuers at ~54-55% simultaneously
- Card network breakdown: Visa, Mastercard, Amex all equally affected
- Incident timeline anchored to Nov 15 DS v2.2 changelog entry

Specific asks from EBANX:
1. Formal acknowledgment of the regression correlated with the Nov 15 DS v2.2 update
2. Root cause analysis with technical detail on the timeout mechanism
3. Fix ETA with milestone checkpoints
4. Confirmation of whether the issue affects only 3DS-authenticated transactions or all card authorizations
5. Rollback feasibility assessment (can DS v2.2 be reverted while fix is developed?)
6. Post-incident protocol: what change notification will EBANX commit to for future infrastructure updates?

**Expected output:** EBANX P1 ticket acknowledged within 24h, RCA + fix timeline within 72h.

---

### A3 — Define EBANX Reactivation Criteria [P1]
**Owner:** Yuno (propose) + StreamVibe (approve)
**Timeline:** Agree within 48 hours
**Action:** Establish measurable, objective criteria that must be met before EBANX is restored as Brazil primary. These criteria protect StreamVibe from a premature rollback that re-exposes the problem.

**Proposed criteria (to confirm with StreamVibe):**
- EBANX Brazil card AR ≥ 80% sustained for **5 consecutive days** in shadow testing
- Decline code 91 at or below baseline (<5% of EBANX declines)
- Written confirmation from EBANX that the DS v2.2 regression is resolved in production
- Parallel processing period: run EBANX and dLocal simultaneously at 20%/80% split for 48h before full reactivation

**Why this matters:** Without explicit criteria, there's pressure to reactivate EBANX as soon as they say "it's fixed." These criteria ensure we validate with data, not promises.

---

### A4 — EBANX: Root Cause Analysis + Fix [P1]
**Owner:** EBANX (Yuno to track and escalate)
**Timeline:** RCA within 72h, fix ETA within 1 week
**Actions required from EBANX:**
- Identify exact mechanism: which component in DS v2.2 is generating the timeout (connection pool configuration, endpoint routing, TLS handshake parameters, or ACS URL mapping)
- Determine if a rollback to DS v2.1 is feasible as a stopgap
- Confirm scope: 3DS-only or broader authorization path
- Provide fix ETA and deployment window with change control notice to Yuno

**Escalation path if EBANX does not respond within 72h:** Escalate to Yuno's Partner Integrations team to engage EBANX at the partnership level, not just support tier.

---

### A5 — Soft Decline Retry Logic for Code 91 [P2]
**Owner:** Yuno
**Timeline:** Within 1 week
**Action:** Code 91 (issuer timeout) is a **soft decline** — it is retryable. Currently, when EBANX returns code 91, Yuno's routing logic may not be immediately re-routing to dLocal for that specific transaction. Validate and tighten the retry behavior:

- Confirm that code 91 responses from any acquirer trigger an immediate single retry via the available fallback acquirer
- Confirm retry is happening within the same checkout session (not requiring the customer to re-initiate)
- Validate the retry does not cause double-charge risk (idempotency key behavior)

**Expected impact:** For retryable code 91 declines that are currently falling through, a same-session retry to dLocal could recover an estimated **15-25% of soft declines**, translating to ~200-400 additional approvals/month.
**StreamVibe engineering requirement:** None — this is a Yuno routing configuration change.

---

## Workstream B: Structural Hardening (Post-Incident)

### B1 — Brazil Routing Split Post-Resolution [P2]
**Owner:** Yuno
**Timeline:** Implement within 1 week of EBANX resolution
**Action:** Once EBANX passes reactivation criteria, do not restore them to 100% of Brazil card traffic. Implement a permanent routing split:

**Proposed configuration:**
- dLocal: 50% of Brazil card traffic (primary)
- EBANX: 50% of Brazil card traffic (co-primary)
- Fallback: dLocal handles EBANX declines, EBANX handles dLocal declines

**Why 50/50 and not the previous 70/30:**
The prior configuration (70% EBANX, 30% dLocal fallback) created concentration risk — when EBANX degraded, 70% of card volume was immediately at risk. A balanced split means a single-acquirer failure degrades by at most 50%, not 70%+.

**Expected impact:** Reduces blast radius of any future single-acquirer incident by ~40%. Also enables ongoing A/B performance comparison between acquirers.
**StreamVibe engineering requirement:** None.

---

### B2 — Acquirer Health Monitoring + Auto-Failover [P3]
**Owner:** Yuno
**Timeline:** 2–3 weeks
**Action:** Implement real-time acquirer health monitoring with automated failover triggers:

- Alert threshold: if any acquirer's 1-hour rolling auth rate drops >5pp below their 7-day baseline → trigger alert to Yuno TAM + ops
- Hard failover threshold: if any acquirer's 1-hour rolling auth rate drops >15pp below baseline → auto-route 100% of that acquirer's traffic to the fallback
- Code-specific monitoring: spike in code 91 or code 05 beyond 2× baseline → alert, regardless of overall AR

**Context:** In this incident, EBANX's auth rate degraded from Nov 15 to Nov 21 before detection. This tooling would have flagged the anomaly within 24-48 hours of the Nov 15 update, not 6 days later.

**Expected impact:** Reduces detection-to-mitigation time from ~6 days to <24 hours for future acquirer-side incidents.
**StreamVibe engineering requirement:** None — dashboard/alerting configuration on Yuno's side.

---

### B3 — 3DS Threshold Review for Brazil [P3]
**Owner:** Yuno + StreamVibe
**Timeline:** 2–3 weeks (post-incident stabilization)
**Action:** The current 3DS trigger in Brazil is set at **>$50 USD**. StreamVibe's pricing is:
- Monthly plan: $9.99 → **below threshold** → no 3DS required
- Annual plan: $89.99 → **above threshold** → 3DS triggered

This means 3DS friction is currently concentrated on StreamVibe's highest-value transactions (annual subscribers). Two optimizations to evaluate:

**Option 1 — Threshold adjustment:**
Lower the threshold to ensure 3DS is only applied where required by issuer mandates, not on all transactions above $50. Some Brazilian issuers mandate 3DS for card-not-present transactions above a certain amount (varies by issuer). Review the actual mandate per BIN range vs. the blanket rule currently applied.

**Option 2 — 3DS exemptions for recurring transactions:**
Brazilian regulation allows 3DS exemptions for subsequent recurring payments once the initial transaction has been authenticated (MIT — Merchant Initiated Transaction). If StreamVibe is triggering 3DS on every monthly renewal for the same subscriber, this is unnecessary friction and a fixable auth rate leak.

**Expected impact:** Reducing unnecessary 3DS challenges on annual renewals could improve EBANX auth rate by an estimated 2-4pp once the regression is resolved. Also reduces abandonment from 3DS friction.
**StreamVibe engineering requirement:** Minor — may require updating the payment initiation flag for recurring transactions (MIT vs. CIT). Yuno to provide integration guidance.

---

## Impact Summary

| Action | Auth Rate Impact | Revenue Impact | Timeline |
|--------|-----------------|----------------|----------|
| A1 (dLocal as primary) | Brazil cards: ~82.7% (vs current 63%) | Recovers ~$37K–$130K/month | ✅ Active now |
| A5 (Code 91 retry logic) | +15-25% of soft declines recovered | +~$2K–$5K/month incremental | 1 week |
| B1 (Routing split) | Reduces blast radius future incidents by ~40% | Prevents next incident impact | Post-fix |
| B3 (3DS threshold) | +2-4pp on EBANX annual renewals post-fix | +~$3K–$8K/month incremental | 2-3 weeks |

---

## RACI Summary

| Action | Yuno | StreamVibe | EBANX |
|--------|------|------------|-------|
| dLocal as primary (A1) | **R** | I | — |
| EBANX escalation (A2) | **R** | I | A |
| Reactivation criteria (A3) | **R** | **A** | I |
| Root cause + fix (A4) | I/escalate | I | **R** |
| Code 91 retry logic (A5) | **R** | I | — |
| Routing split (B1) | **R** | I | — |
| Health monitoring (B2) | **R** | I | — |
| 3DS threshold review (B3) | **R** | **A** | — |

*R = Responsible, A = Approves, I = Informed*

---

## What StreamVibe Engineering Needs to Do

**Immediate:** Nothing. All mitigation actions are Yuno-side routing changes.

**Within 2-3 weeks (low effort):**
- Review and approve EBANX reactivation criteria (business decision, not engineering)
- Evaluate MIT flag implementation for recurring transactions (B3) — minor integration change, Yuno provides guidance

**Optional but recommended:**
- Monitor subscriber churn data for the period Nov 21–present. Customers who had a failed card renewal and did not retry represent a recoverable churn cohort — a targeted win-back campaign (retry prompt, discount, alternative payment method) can recover a portion of them. This is a product/marketing decision, not a payments one.

---

*Next update: November 27, 2024 — or sooner if EBANX provides RCA response.*
