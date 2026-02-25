# StreamVibe LatAm Authorization Crisis — Full Technical Package

**Merchant:** StreamVibe
**Period:** Last 30 days (as of November 25, 2024)
**Prepared by:** Yuno TAM Team
**Incident status:** Active — Brazil cards, EBANX-exclusive

---

## Executive Introduction

StreamVibe, a video streaming platform operating across five Latin American markets (~87,000 active subscribers, ~$2.3M/month in processing), experienced a significant drop in Brazil's payment approval rate over the past 30 days: from 81.2% to 68.0% overall, and from 82.5% to 63.0% for card transactions specifically.

This document consolidates the complete technical analysis and response produced by the Yuno TAM team. The root cause has been identified with high confidence: a regression introduced by EBANX's November 15 infrastructure update to its 3DS2 Directory Server degraded authentication routing for Brazilian issuers, producing a systemic 829% spike in issuer timeout errors (decline code 91) across all card networks and banks. Yuno's automatic failover to dLocal has stabilized card approval rates at ~82.7% while EBANX resolves the issue. No engineering work is required from StreamVibe's side.

Argentina's low authorization rate (54%) is a separate, unrelated issue with macroeconomic causes. It is not a technical incident and is treated as such throughout this document.

**Estimated revenue at risk:** $37K–$130K/month (depending on plan mix), recoverable with the actions described herein.

---

## Table of Contents

1. [Root Cause Analysis — Technical (Internal)](#section-1-root-cause-analysis--technical-internal)
   - 1.1 Brazil: Isolating the Affected Segment
   - 1.2 Brazil Cards: Acquirer-Level Isolation
   - 1.3 EBANX: Breakdown by Network and Issuer BIN
   - 1.4 Error Code Forensic Analysis
   - 1.5 Root Cause Hypothesis
   - 1.6 Events that Are NOT the Root Cause
   - 1.7 Impact Quantification
   - 1.8 Argentina: Technical Assessment
   - 1.9 Confirmed vs. Needs Confirmation

2. [Technical Action Plan](#section-2-technical-action-plan)
   - 2.1 Priority Matrix
   - 2.2 Workstream A: Active Incident Response
   - 2.3 Workstream B: Structural Hardening
   - 2.4 RACI Summary
   - 2.5 What StreamVibe Engineering Needs to Do

3. [Merchant Communication Package](#section-3-merchant-communication-package)
   - 3.1 RCA Delivery Email
   - 3.2 Merchant-Facing Incident Report
   - 3.3 Pre-Call Speech Guide

4. [Proactive Optimization Recommendations](#section-4-proactive-optimization-recommendations)
   - 4.1 Recommendation 1: Make PIX the Primary Payment Experience in Brazil
   - 4.2 Recommendation 2: Address Argentina by Solving the Right Problem
   - 4.3 Recommendation 3: Tokenization + MIT for Recurring Billing Across All Markets
   - 4.4 Prioritized Summary Table

---

---

# Section 1: Root Cause Analysis — Technical (Internal)

**Audience:** Internal / Technical

---

## Overview

StreamVibe's Brazil authorization rate dropped from **81.2% to 68.0% (-13.2pp)** over the last 30 days. The drop is entirely attributable to a single acquirer — EBANX — and is driven by a systemic spike in issuer timeout errors (decline code 91), consistent with a defect introduced by EBANX's November 15 3DS2 infrastructure update. Argentina's low authorization rate (54%) is a separate, unrelated issue rooted in macroeconomic conditions, not a technical failure.

---

## 1.1 Brazil: Isolating the Affected Segment

| Payment Method | Attempts | Current AR | Prior AR | Change |
|----------------|----------|-----------|------------|--------|
| Cards | 19,200 | 63.0% | 82.5% | **-19.5pp** |
| PIX | 6,850 | 93.0% | 92.8% | +0.2pp |
| Boleto | 2,400 | 78.3% | 77.9% | +0.4pp |

PIX and Boleto are performing at or above prior-period levels. The crisis is **cards only**. This rules out any platform-level integration issue, network-level outage, or merchant misconfiguration affecting all payment methods.

---

## 1.2 Brazil Cards: Acquirer-Level Isolation

| Acquirer | Attempts | Current AR | Prior AR | Change |
|---------|----------|-----------|------------|--------|
| EBANX (primary) | 13,440 | 54.6% | 83.1% | **-28.5pp** |
| dLocal (fallback) | 5,760 | 82.7% | 81.0% | +1.7pp |

dLocal is processing payments for **the same Brazilian cardholders, at the same issuing banks, on the same card networks** — and reaching 82.7% approval rate. The 28.1pp gap between EBANX and dLocal eliminates Brazil's banking infrastructure, macroeconomic conditions, and issuer-side policy changes as contributing factors. **The failure is inside EBANX.**

---

## 1.3 EBANX: Breakdown by Network and Issuer BIN

**By Network:**

| Network | Attempts | Current AR | Prior AR | Change |
|---------|----------|-----------|------------|--------|
| Visa | 7,392 | 54.0% | 84.2% | -30.2pp |
| Mastercard | 4,838 | 55.0% | 81.8% | -26.8pp |
| Amex | 1,210 | 56.4% | 82.5% | -26.1pp |

**By Issuer BIN:**

| Issuer | Attempts | Current AR | Prior AR | Change |
|--------|----------|-----------|------------|--------|
| Banco do Brasil (4011xx) | 2,688 | 55.0% | 85.1% | -30.1pp |
| Itaú (5211xx) | 3,360 | 54.0% | 83.8% | -29.8pp |
| Bradesco (4389xx) | 2,150 | 54.0% | 82.9% | -28.9pp |
| Santander (5543xx) | 1,478 | 54.1% | 83.5% | -29.4pp |
| Nubank (5201xx) | 1,881 | 55.0% | 84.0% | -29.0pp |
| Other issuers | 1,883 | 55.7% | 82.4% | -26.7pp |

The authorization rate converged in a narrow band of **54-56% across every major issuer and every card network**. This mathematical convergence is only explainable by a failure in the **shared intermediary** between EBANX and all these issuers: EBANX's own 3DS Directory Server layer.

---

## 1.4 Error Code Forensic Analysis

| Code | Description | Current Count | Prior Count | Change | % of Declines |
|------|-------------|--------------|------------|--------|---------------|
| **91** | **Issuer timeout/unavailable** | **3,825** | **412** | **+829%** | **62.8%** |
| 51 | Insufficient funds | 1,104 | 1,089 | +1.4% | 18.1% |
| 05 | Do not honor (generic) | 518 | 502 | +3.2% | 8.5% |
| 14 | Invalid card number | 245 | 238 | +2.9% | 4.0% |
| 54 | Expired card | 184 | 179 | +2.8% | 3.0% |
| 62 | Restricted card | 137 | 133 | +3.0% | 2.2% |

Code 91 is the only anomaly. All other decline codes increased 1-3% — consistent with normal volume variation. Code 91 increased **829%**. This is a single-mode failure event.

Code 91 in the context of 3DS processing indicates the issuer's ACS (Authentication Control Server) is not responding within the expected timeout window during the 3DS challenge. EBANX's Directory Server is responsible for routing these challenge requests. A defect in that routing layer would produce exactly this pattern: timeouts across all issuer ACS endpoints simultaneously, with no individual issuer being uniquely worse.

---

## 1.5 Root Cause Hypothesis

**Hypothesis: EBANX's Directory Server v2.2 upgrade on November 15 introduced a regression in 3DS authentication routing for Brazilian issuers, causing systemic ACS request timeouts (code 91) across all card networks and issuing banks.**

**Confidence level: High.**

Supporting evidence chain:
1. **Temporal correlation:** EBANX deployed DS v2.2 on Nov 15. Rates were stable on Nov 10 (82.8%) and Nov 16 (81.9%). Collapse on Nov 21.
2. **Failure mode:** Code 91 spiked 829% at EBANX. dLocal exhibits no code 91 spikes.
3. **Impact uniformity:** All five major issuers and all three card networks dropped 28-30pp in parallel.
4. **Vendor self-acknowledgment:** EBANX support confirmed "intermittent latency on 3DS challenges from Brazilian issuers since Nov 20" (in a parallel conversation with another Yuno merchant — internal use only).

**Remaining uncertainty:**
- The 5-day lag between upgrade (Nov 15) and collapse (Nov 21) is not fully explained. Possible mechanisms: gradual rollout, connection pool saturation, or the BdB maintenance window acting as a stress trigger.
- Amex is not in the DS v2.2 scope (Visa/MC only), yet also dropped ~26pp — raises the question of whether the issue extends beyond 3DS into EBANX's general authorization API.
- A breakdown by transaction amount (above/below the $50 USD 3DS threshold) would confirm whether $9.99 monthly non-3DS transactions are also affected.

---

## 1.6 Events that Are NOT the Root Cause

**StreamVibe frontend checkout update (Oct 31):** Deployed 3+ weeks before the drop. Rates stable at 82.8% through Nov 10. Hard decline codes 14 and 54 increased only 2.9% and 2.8%. Not a contributing factor.

**PIX launch in Brazil (Nov 8):** Operates on a separate payment rail. PIX AR is 93.0%, stable. No interaction with card authorization infrastructure. Unrelated.

**Banco do Brasil scheduled maintenance (Nov 20-21):** Marked complete by Nov 21 evening; the rate continued deteriorating to 54.6% through Nov 24. BdB BINs dropped to exactly the same level as Itaú, Bradesco, Santander, and Nubank. dLocal maintained 82.7% on BdB BINs during the same period. Not the root cause — may have acted as the stress trigger that surfaced EBANX's pre-existing latency regression.

---

## 1.7 Impact Quantification — Brazil Cards

| | Prior Period | Current Period | Delta |
|-|-------------|----------------|-------|
| EBANX card approvals | 11,169 (13,440 × 83.1%) | 7,338 (13,440 × 54.6%) | **-3,831** |
| dLocal card approvals | 4,666 (5,760 × 81.0%) | 4,764 (5,760 × 82.7%) | +98 (partial recovery) |
| **Net Brazil card approvals lost** | **15,840** | **12,096** | **-3,744/month** |

**Revenue at risk:**

| Scenario | Monthly Impact |
|----------|---------------|
| Conservative (100% monthly plan, avg $9.99) | **~$37,402/month** |
| Blended (70% monthly / 30% annual, $89.99) | **~$127,000/month** |

These figures represent only failed charge attempts. Downstream subscriber churn from customers who did not retry is unquantified but typically multiplies 1.5×–2.5× the immediate revenue impact for subscription businesses.

---

## 1.8 Argentina: Technical Assessment

**Diagnosis: Performing at market baseline. Not a technical incident.**

| Metric | StreamVibe | LatAm Subscription Benchmark |
|--------|-----------|-------------------------------|
| Cards via Mercado Pago | 51.0% | 52-58% |
| Mercado Pago wallet | 66.1% | 64-70% |

Argentina's card auth rate is within the established benchmark range. The dominant decline codes are 51 (insufficient funds) and 62 (restricted card) — issuer responses to Argentina's macroeconomic environment (>100% annual inflation, capital controls). Non-retryable and not resolved by routing changes.

Alternatives (Stripe, dLocal) perform at **48-50%** on Argentina cards — materially worse than Mercado Pago's 51%. Disabling Mercado Pago would reduce Argentina's auth rate by an estimated **3-5pp**.

The Mercado Pago wallet (66.1%, above benchmark) is the highest-converting instrument available in Argentina. Strategic opportunity: shift the subscriber base toward the wallet.

---

## 1.9 Confirmed vs. Needs Confirmation

**Confirmed by data:**
- The Brazil crisis is cards only. PIX and Boleto unaffected.
- The crisis is EBANX only. dLocal operating at 82.7%.
- Decline code 91 increased 829% exclusively at EBANX.
- All issuers and networks uniformly affected → failure in EBANX's DS layer.
- EBANX deployed DS v2.2 five days before the collapse.
- EBANX support acknowledged 3DS challenge latency since Nov 20.
- Argentina is at market benchmark. No alternative acquirer performs better.

**Requires confirmation:**
- Breakdown by transaction amount (above/below $50 USD 3DS threshold): are $9.99 monthly transactions also declining?
- EBANX formal root cause analysis and fix/rollback timeline.
- Whether Amex flows through the updated DS v2.2 or a separate path.
- Exact DS v2.2 rollout schedule (gradual vs. full cutover on Nov 15).

---

---

# Section 2: Technical Action Plan

**Audience:** Internal + StreamVibe (Claudia Mendez, Rafael Santos)

---

## Overview

This action plan addresses two parallel workstreams:

- **Workstream A — Active incident (Brazil):** Stabilize authorization rates while EBANX resolves the root cause, then restore the optimal routing configuration with safeguards.
- **Workstream B — Structural hardening:** Implement changes that prevent this class of failure from recurring, regardless of which acquirer is involved.

Argentina is excluded from this plan — it requires a separate strategic discussion and is not a technical incident response.

---

## 2.1 Priority Matrix

| Priority | Action | Owner | Timeline | Expected Impact |
|----------|--------|-------|----------|----------------|
| P0 | Maintain dLocal as Brazil primary | Yuno | ✅ Done | Stabilizes AR at ~82.7% |
| P0 | Formal escalation to EBANX with full evidence package | Yuno | 24h | Unblocks resolution timeline |
| P1 | Define EBANX reactivation criteria | Yuno + StreamVibe | 48h | Prevents premature rollback |
| P1 | EBANX: formal RCA + fix ETA | EBANX | 48–72h | Closes root cause loop |
| P2 | Retry logic for soft decline code 91 | Yuno | 1 week | Recovers retryable declines |
| P2 | Brazil routing split post-resolution | Yuno | Post-fix | Eliminates single-acquirer exposure |
| P3 | Acquirer health monitoring + auto-failover | Yuno | 2–3 weeks | Reduces detection time from days to hours |
| P3 | 3DS threshold review for Brazil | Yuno + StreamVibe | 2–3 weeks | Reduces unnecessary friction on annual plans |

---

## 2.2 Workstream A: Active Incident Response

### A1 — Maintain dLocal as Brazil Primary [P0 — DONE]
**Owner:** Yuno | **Status:** ✅ Active

dLocal now handles 100% of Brazil card traffic. No engineering changes required from StreamVibe. Expected impact: Brazil card AR stabilizes at ~82.7%, recovering approximately **3,744 lost approvals/month** vs. the current EBANX degraded state.

**Note:** Do not revert to EBANX until reactivation criteria (A3) are formally met and validated with data.

---

### A2 — Formal Escalation to EBANX [P0 — In Progress]
**Owner:** Yuno | **Timeline:** 24 hours

Submit a formal P1 escalation to EBANX TAM/support with the following evidence package:
- Brazil card AR by acquirer: EBANX 54.6% vs. dLocal 82.7% (same period, same BINs)
- Decline code 91 trend: 412 → 3,825 (+829%), representing 62.8% of all EBANX declines
- Issuer BIN breakdown: top 5 issuers all at ~54-55% simultaneously
- Card network breakdown: Visa, Mastercard, Amex equally affected
- Incident timeline anchored to the Nov 15 DS v2.2 changelog entry

Specific requests to EBANX:
1. Formal acknowledgment of regression correlated with Nov 15 DS v2.2 upgrade
2. Root cause analysis with technical detail on the timeout mechanism
3. Fix ETA with intermediate milestones
4. Confirmation of scope: 3DS-authenticated transactions only or all card authorizations
5. Rollback feasibility assessment (can DS v2.2 be reverted while fix is developed?)
6. Post-incident protocol: what change control notification does EBANX commit to for future infrastructure updates?

**Expected output:** EBANX P1 ticket acknowledged in 24h, RCA + fix timeline in 72h.

---

### A3 — Define EBANX Reactivation Criteria [P1]
**Owner:** Yuno (proposes) + StreamVibe (approves) | **Timeline:** Agree within 48 hours

Establish measurable, objective criteria that must be met before restoring EBANX as Brazil primary. These protect StreamVibe from a premature rollback.

**Proposed criteria:**
- Brazil card AR on EBANX ≥ 80% sustained for **5 consecutive days** in shadow testing
- Decline code 91 at or below baseline (<5% of EBANX declines)
- Written confirmation from EBANX that the DS v2.2 regression is resolved in production
- Parallel processing period: run EBANX and dLocal at 20%/80% for 48h before full reactivation

---

### A4 — EBANX: Root Cause Analysis + Fix [P1]
**Owner:** EBANX (Yuno tracks and escalates) | **Timeline:** RCA in 72h, fix ETA in 1 week

Required actions from EBANX:
- Identify the exact mechanism: which DS v2.2 component is generating the timeout (connection pool config, endpoint routing, TLS handshake parameters, or ACS URL mapping)
- Determine whether a rollback to DS v2.1 is feasible as an interim measure
- Confirm scope: 3DS flow only or also general authorization path
- Provide fix ETA and deployment window with change control notice to Yuno

**Escalation path if EBANX does not respond in 72h:** Escalate to Yuno's Partner Integrations team to engage EBANX at the partnership level, not just support.

---

### A5 — Retry Logic for Soft Decline Code 91 [P2]
**Owner:** Yuno | **Timeline:** Within 1 week

Code 91 (issuer timeout) is a **soft decline** — it is retryable. Validate and adjust retry behavior:
- Confirm code 91 responses from any acquirer trigger an immediate single retry via the available fallback acquirer
- Confirm the retry occurs within the same checkout session (no customer restart required)
- Validate the retry does not generate double-charge risk (idempotency key behavior)

**Expected impact:** For retryable code 91 declines not currently being recovered, a same-session retry to dLocal could recover an estimated **15-25% of soft declines**, equivalent to ~200-400 additional approvals/month.
**StreamVibe engineering requirement:** None — this is a routing configuration change on Yuno's side.

---

## 2.3 Workstream B: Structural Hardening (Post-Incident)

### B1 — Brazil Routing Split Post-Resolution [P2]
**Owner:** Yuno | **Timeline:** Implement within 1 week of EBANX resolution

Once EBANX meets the reactivation criteria, do not restore 100% of Brazil card traffic to EBANX. Implement a permanent split:

**Proposed configuration:**
- dLocal: 50% of Brazil card traffic (co-primary)
- EBANX: 50% of Brazil card traffic (co-primary)
- Fallback: dLocal handles EBANX declines, EBANX handles dLocal declines

The prior 70/30 configuration created concentration risk — when EBANX degraded, 70% of card volume was immediately exposed. An even split means a single-acquirer failure degrades at most 50%, not 70%+.

**Expected impact:** Reduces blast radius of any future single-acquirer incident by ~40%. Also enables continuous A/B performance comparison between acquirers.
**StreamVibe engineering requirement:** None.

---

### B2 — Acquirer Health Monitoring + Auto-Failover [P3]
**Owner:** Yuno | **Timeline:** 2–3 weeks

Implement real-time acquirer health monitoring with automated failover triggers:
- **Alert threshold:** if any acquirer's hourly AR drops >5pp below its 7-day baseline → alert to Yuno TAM + ops
- **Hard failover threshold:** if any acquirer's hourly AR drops >15pp below baseline → automatically route 100% of that acquirer's traffic to fallback
- **Code-specific monitoring:** spike in code 91 or code 05 above 2× baseline → alert, regardless of overall AR

**Context:** In this incident, EBANX's rate degraded from Nov 15 to Nov 21 before detection. This system would have flagged the anomaly within 24-48 hours of the Nov 15 upgrade, not 6 days later.

**Expected impact:** Reduces detection-to-mitigation time from ~6 days to <24 hours for future acquirer-side incidents.
**StreamVibe engineering requirement:** None.

---

### B3 — 3DS Threshold Review for Brazil [P3]
**Owner:** Yuno + StreamVibe | **Timeline:** 2–3 weeks (post-incident stabilization)

Current 3DS trigger in Brazil is set at **>$50 USD**. StreamVibe's pricing:
- Monthly plan: $9.99 → **below threshold** → no 3DS required
- Annual plan: $89.99 → **above threshold** → 3DS activated

3DS friction is currently concentrated on StreamVibe's highest-value transactions (annual subscribers). Two optimizations to evaluate:

**Option 1 — Threshold adjustment:** Review whether the $50 threshold reflects actual issuer mandates by BIN, vs. a generic rule. Some Brazilian issuers require 3DS for card-not-present transactions above a certain amount that varies by issuer. Review actual mandates by BIN range vs. the current blanket rule.

**Option 2 — 3DS exemptions for recurring transactions:** Brazilian regulation allows 3DS exemptions for subsequent recurring payments once the initial transaction has been authenticated (MIT — Merchant Initiated Transaction). If StreamVibe is triggering 3DS on every monthly renewal from established subscribers, this is unnecessary friction and a solvable approval rate leak.

**Expected impact:** Reducing unnecessary 3DS challenges on annual renewals could improve EBANX's rate by an estimated 2-4pp once the regression is resolved.
**StreamVibe engineering requirement:** Minor — may require updating the payment initiation flag for recurring transactions (MIT vs. CIT). Yuno provides integration guidance.

---

## 2.4 RACI Summary

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

## 2.5 What StreamVibe Engineering Needs to Do

**Immediate:** Nothing. All mitigation actions are routing changes on Yuno's side.

**In 2-3 weeks (low effort):**
- Review and approve EBANX reactivation criteria (business decision, not engineering)
- Evaluate MIT flag implementation for recurring transactions (B3) — minor integration change, Yuno provides guidance

**Optional but recommended:**
- Monitor subscriber churn data for the Nov 21–present period. Customers who had a failed card renewal and did not retry represent a recoverable churn cohort — a targeted win-back campaign (retry prompt, discount, alternative payment method) can recover a portion. This is a product/marketing decision, not a payments one.

---

## Impact Summary

| Action | AR Impact | Revenue Impact | Timeline |
|--------|-----------|----------------|----------|
| A1 (dLocal as primary) | Brazil cards: ~82.7% (vs 63% current) | Recovers ~$37K–$130K/month | ✅ Active now |
| A5 (code 91 retry logic) | +15-25% of soft declines recovered | +~$2K–$5K/month incremental | 1 week |
| B1 (routing split) | Reduces blast radius ~40% for future incidents | Prevents impact of next incident | Post-fix |
| B3 (3DS threshold) | +2-4pp on annual renewals post-fix | +~$3K–$8K/month incremental | 2-3 weeks |

---

---

# Section 3: Merchant Communication Package

**Audience:** Claudia Mendez (Head of Payments) · Rafael Santos (CTO)

---

## 3.1 RCA Delivery Email

**To:** claudia.mendez@streamvibe.com
**CC:** rafael.santos@streamvibe.com
**Subject:** [StreamVibe] Root cause analysis — Brazil approval rate drop
**Date:** November 25, 2024

---

Claudia, Rafael,

Sharing the formal analysis of what is happening with Brazil's approval rates.

**The key points in three bullets:**

1. **The problem has been identified and localized.** The card decline (from 82.5% to 63.0%) is strongly correlated with an infrastructure update EBANX performed on November 15. Our analysis points to a regression in their authentication layer as the probable cause — which we are formally confirming with them. The rest of the ecosystem — PIX, Boleto, and dLocal — is operating normally.

2. **Mitigation is already active.** We activated dLocal as the primary processor in Brazil on Yuno's side. This required no technical changes from StreamVibe. Transactions are being processed normally by dLocal (82.7% approval rate) while we work with EBANX on the definitive resolution.

3. **Argentina is a separate issue.** The 54% rate in Argentina is not a technical failure — it is within local market benchmark and has macroeconomic causes. Disabling Mercado Pago would worsen that number. I explain it in detail in the attached report.

---

The attached document covers in detail: incident timeline, evidence confirming the root cause, what we ruled out, estimated revenue impact, and actions in progress with owners and timelines.

We'll be in touch to coordinate the follow-up call. Any questions before then, I'll respond immediately.

Best,
[TAM Name]
Technical Account Manager — Yuno
[email] | [phone]

---
> 📎 Attachment: `streamvibe-brazil-auth-rate-rca-nov2024.pdf`

---

## 3.2 Merchant-Facing Incident Report

**Incident status:** Active — under resolution
**Audience:** Claudia Mendez (Head of Payments) · Rafael Santos (CTO)

### Executive Summary

Over the past week, StreamVibe experienced a significant drop in card payment approval rates in Brazil: from **81.2% to 68.0%** overall, and from **82.5% to 63.0%** for cards specifically. The problem is localized to a single payment processor — EBANX — and is strongly correlated with an infrastructure update performed on November 15. Our analysis points to a regression in their authentication layer as the probable cause, which we are formally confirming with EBANX.

Alternative payment methods (PIX and Boleto) are functioning normally. The fallback processor, dLocal, is maintaining an 82.7% approval rate for the same affected cards, confirming the problem is exclusive to EBANX and does not reflect the behavior of the Brazilian market or issuing banks. The Argentina situation is independent with a different cause, detailed at the end of this report.

---

### What Is Happening?

When a customer attempts to pay by card in Brazil, the transaction goes through an identity verification process called **3DS authentication** (where the bank asks the customer to confirm the purchase via SMS, app, or password). EBANX acts as the intermediary between StreamVibe and the issuing banks to manage this process.

On November 15, EBANX performed a technical update to the server that manages these authentications. Starting November 20, that server began generating excessive response times when communicating with Brazilian issuing banks. As a result, banks interpret the lack of timely response and decline the transaction with error code **91 ("issuer timeout")**.

This error went from representing **3% of declines** to causing **62.8% of all EBANX declines**, with an increase of **829%** in absolute volume. All other decline types (insufficient funds, expired card, etc.) remain at normal levels.

---

### Incident Timeline

| Date | Event |
|------|--------|
| Nov 2 | Yuno updates Brazil routing: EBANX becomes primary processor |
| Nov 10 | Approval rate via EBANX: **82.8%** — normal operation |
| Nov 15 | EBANX implements update to its 3DS authentication infrastructure |
| Nov 16 | Approval rate via EBANX: **81.9%** — still stable |
| Nov 20-21 | Banco do Brasil scheduled maintenance (3DS) — marked complete Nov 21 evening |
| Nov 21 | EBANX approval rate drops to **68.2%**. Timeout error rises from ~3% to 61% of declines |
| Nov 24 | EBANX approval rate: **54.6%** — continues deteriorating |
| Nov 25 | Yuno has activated mitigation via dLocal. Active escalation with EBANX |

---

### Evidence Confirming the Root Cause

**1. dLocal processes the same cards at 82.7%**

The fallback processor dLocal is approving cards from the exact same Brazilian banks (Banco do Brasil, Itaú, Bradesco, Santander, Nubank) at **82.7%** — nearly identical to the prior period. If the problem were with the banks or the Brazilian market, dLocal would also be affected. It is not.

| Processor | Current approval rate | Change vs prior period |
|-----------|----------------------|------------------------|
| EBANX (primary) | 54.6% | **-28.5 percentage points** |
| dLocal (fallback) | 82.7% | +1.7 percentage points |

**2. All banks and networks dropped simultaneously to the same level**

Brazil's five major issuing banks (Banco do Brasil, Itaú, Bradesco, Santander, Nubank) dropped 28-30 percentage points simultaneously, all converging at ~54-55%. Visa, Mastercard, and Amex dropped almost identically. This mathematical uniformity cannot be the result of changes at individual banks — it can only be explained by a failure in the system mediating between EBANX and all banks: their authentication server.

**3. The only error that exploded was the timeout**

| Error code | Description | Before | Now | Change |
|-----------|-------------|--------|-----|--------|
| **91** | **Bank not responding (timeout)** | **412** | **3,825** | **+829%** |
| 51 | Insufficient funds | 1,089 | 1,104 | +1.4% |
| 05 | Generic decline | 502 | 518 | +3.2% |
| 14 | Invalid card number | 238 | 245 | +2.9% |
| 54 | Expired card | 179 | 184 | +2.8% |

All other declines grew only 1-3%, consistent with normal volume growth. Code 91 grew 829%. It is a single-type failure with an identifiable origin.

---

### What Did NOT Cause the Problem

**The checkout update (Oct 31):** Implemented nearly four weeks before the drop. Approval rates remained stable through November 16. Declines from card data errors (codes 14 and 54) show no abnormal change.

**PIX launch (Nov 8):** PIX operates on a completely separate payment network. Its approval rate is **93.0%**, stable and unchanged. It has no interaction with the card authorization system.

**Banco do Brasil maintenance (Nov 20-21):** This maintenance window coincided with the start of the drop, but is not the root cause. The maintenance was marked complete and the rate continued falling for days afterward. Additionally, all other banks without maintenance dropped at exactly the same level as Banco do Brasil, and dLocal maintains 82.7% approval on Banco do Brasil cards during the same period.

---

### Revenue Impact

| | Prior period | Current period | Difference |
|-|-------------|----------------|------------|
| Approvals via EBANX | ~11,169/month | ~7,338/month | **-3,831 approvals** |
| Recovery via dLocal | — | +98 additional | Partial mitigation |
| **Net approval loss** | | | **~3,744/month** |

**Estimated revenue impact:**
- Conservative scenario (monthly subscriptions at $9.99): **~$37,400/month at risk**
- Blended scenario (monthly + annual): **up to ~$127,000/month at risk**

These figures reflect only payments that failed and were not recovered. They do not include subscription cancellations from customers who faced a decline and did not retry — which historically multiplies the impact 1.5×–2.5× in subscription businesses.

---

### Argentina: A Different Situation

Argentina's approval rate (54%) has shown no significant improvement for 8 weeks and is unrelated to the Brazil incident. Its cause is structural and macroeconomic.

Argentina has annual inflation above 100% and strict capital controls that limit international card use. The primary decline types are insufficient funds and cards with bank-restricted limits — both reflecting the country's economic conditions, not integration failures.

**The 51-54% rate is within market reference range** (52-58%) for subscription businesses in Argentina. Alternative processors available in Argentina show lower card approval rates, as Mercado Pago has an advantage through its direct relationships with major local banks. Switching processors in this market would worsen results, not improve them.

**Recommendation:** Disabling Mercado Pago as a card processor is not recommended — it would reduce the approval rate by an estimated additional 3-5 percentage points.

---

### Actions in Progress

| Action | Owner | Status | Timeline |
|--------|-------|--------|----------|
| Activate dLocal as primary processor in Brazil (temporary) | Yuno | ✅ In progress | Immediate |
| Escalate incident to EBANX with full code 91 evidence | Yuno | 🔄 In progress | 24 hours |
| Request formal root cause analysis and rollback plan from EBANX | Yuno + EBANX | 🕐 Awaiting response | 48 hours |
| Obtain full scope analysis by plan segment | Yuno + EBANX | 🕐 Awaiting data | 48 hours |
| Review Brazil routing strategy post-resolution | Yuno | 🕐 Pending | Next week |

---

### Confirmed vs. Still Pending

**Confirmed:**
- Problem is cards only. PIX and Boleto operating normally.
- Problem is EBANX only. dLocal functioning at 82.7% under identical conditions.
- The dominant error (code 91 — timeout) grew 829% at EBANX exclusively.
- Uniformity of impact across all banks and networks points to EBANX's authentication server as origin.
- The Argentina incident is independent and not a technical failure.

**Pending confirmation:**
- Formal root cause analysis and resolution plan from EBANX.
- Fix timeline and stabilization criteria for reactivating EBANX as primary processor.
- Full scope analysis by plan segment.

---

*Next update: November 27, 2024 — or sooner if there are developments from EBANX.*
*Direct contact for this incident: [TAM Name] — [email]*

---

## 3.3 Pre-Call Speech Guide

**Participants:** Claudia Mendez (Head of Payments), Rafael Santos (CTO)
**Led by:** Senior TAM, Yuno
**Estimated duration:** 30–40 minutes

---

### Before the Call — Checklist

- [ ] Confirm the RCA report was sent in writing before the call
- [ ] Have data ready: current auth rate with dLocal (82.7%), EBANX history, code 91 decline curve
- [ ] Prepare shared screen with Yuno dashboard if needed
- [ ] Confirm dLocal is already active as primary processor in Brazil

---

### 1. OPENING — Tone, agenda, and acknowledging urgency

**Goal:** Establish trust from the first minute. Claudia will be impatient. Rafael will be scanning whether this will generate extra work. Get to the point fast.

**Tone:** Direct, calm, owning the situation. Not defensive.

**Suggested opening:**
> "Thanks for the time, Claudia, Rafael. I'll go straight to the point because I know you have limited time and this deserves real attention."

> "We sent you the written report in parallel with this call — the goal of this session is not to read it together, but for you to leave with full clarity on what happened, what is already resolved, and what is still pending."

**Explicit agenda (say it out loud):**
> "In the next 30 minutes we'll cover four things: one, what caused the Brazil drop and why the problem is isolated; two, what we already activated from Yuno without you having to do anything; three, what we need from EBANX to close the loop; and four, the Argentina topic which I understand is generating noise with the finance team."

---

### 2. DIAGNOSIS — What happened and why

**Goal:** Explain the root cause clearly. No unnecessary technical jargon. Claudia needs the "what" and "why." Rafael can handle more technical depth if he wants it.

**Core message:** The problem is EBANX's, not the Brazilian market's, not StreamVibe's cards, not Yuno's.

**Suggested narrative:**
> "On November 15, EBANX made an infrastructure update to their authentication layer. That update introduced a regression — in simple terms, something that was working stopped working correctly."

> "The decline code that exploded was 91 — it means the bank didn't receive a response in time. That error went up 829% compared to the prior period. That is not normal market variation. It is a very specific technical signal."

> "The number that confirms everything: dLocal, which has the same card portfolio and the same issuers in Brazil, is authorizing at 82.7% in that same period. If the problem were the market, dLocal would also drop. It didn't. The problem is EBANX."

**Key data to have visible:**

| Metric | Before | Now |
|--------|--------|-----|
| Brazil card auth rate | 82.5% | 63.0% |
| EBANX (primary) | 83.1% | 54.6% |
| dLocal (same portfolio) | 81.0% | 82.7% |
| Code 91 declines at EBANX | baseline | +829% |

**For Rafael if he goes deeper technically:**
> "The regression is in EBANX's Directory Server — the intermediary between them and each bank's authentication systems. When that server fails in the handshake with the bank, the transaction times out. That explains why all banks dropped to the same level simultaneously — it's not a problem with each bank, it's a problem with the intermediary."

---

### 3. WHAT WE ALREADY DID — Active mitigation

**Goal:** Show that Yuno already acted. We didn't come to explain a problem — we came to confirm we already mitigated it operationally while closing the underlying solution.

**Core message:** You don't need to do anything technical. It's already mitigated.

**Suggested lines:**
> "From Yuno we already activated dLocal as the primary processor in Brazil. This required no integration change from StreamVibe's side — we handled it at the routing layer within the platform."

> "Claudia, the short answer to 'what are you doing?' is: we're already doing it. Transactions are being processed by dLocal while EBANX resolves the problem."

> "Rafael, this doesn't generate any new tickets or integration work for you. The change was in Yuno's orchestration layer — that's exactly what you're using the platform for."

**To close this point with impact:**
> "The advantage of having orchestration is this: when a processor fails, you don't depend on us sending you an email — we can act directly. That's what we did."

---

### 4. WHAT WE NEED FROM EBANX — Pending items and timeline

**Goal:** Be transparent about what is not in our hands. Don't promise dates we can't guarantee.

**Core message:** Mitigation is active. The definitive resolution depends on EBANX, but we're on top of them.

**Suggested lines:**
> "We have the case escalated with EBANX's technical team. I won't give you a date I can't guarantee — what I will say is this is a documented bug with clear evidence and we're pushing for resolution in the next 7 to 14 days."

> "Until we have that confirmation with data, EBANX does not return as primary in Brazil. Period."

**Open questions with EBANX (share with merchant):**
1. When was the regression detected internally and why was there no proactive notification?
2. What is the fix ETA for production?
3. Will there be a formal post-mortem from their side?
4. What controls will they implement so an infrastructure update doesn't generate this impact without alerts?

> "These questions are open with EBANX and we'll share the answers with you as soon as we have them. We will not accept a 'it's fixed' without data to prove it."

---

### 5. ARGENTINA — Neutralizing the panic

**Goal:** Brief, direct, and definitive. Finance wants to disable Mercado Pago. That would be a mistake. Say it clearly, without generating additional alarm.

**Core message:** Argentina is at market baseline. Disabling Mercado Pago would do more damage.

**Suggested lines:**
> "I understand finance is looking at the 54% in Argentina with concern. I want to be direct: that is not a technical problem. It's the current market baseline in Argentina — inflation, currency restrictions, local bank behavior. There is no bug to fix."

> "Disabling Mercado Pago in Argentina does not improve that number — it makes it worse. It is the processor with the highest coverage and best approval rates available in that market. Removing it eliminates processing capacity, it does not fix the low rate."

**If they push with "so what do we do about Argentina?":**
> "What we do is monitor and evaluate whether there are routing tactics that can move a few points. But expectations have to be realistic: in Argentina's current macroeconomic context, 54% is what much of the market is seeing. It's not a StreamVibe anomaly."

> "We can schedule a separate session specifically for Argentina if you want to go deeper — but mixing it with the Brazil problem would create noise in this call."

---

### 6. CLOSE — Next steps and follow-up cadence

**Next steps — say them out loud:**

| Action | Owner | Timeline |
|--------|-------|----------|
| EBANX response on fix ETA | Yuno (active escalation) | 7–10 business days |
| Weekly Brazil performance report | Yuno | Every Monday |
| EBANX reactivation criteria | Agree today | This call |
| Optional Argentina session | StreamVibe (if decided) | Next week |

**EBANX reactivation criteria — propose in the call:**
> "I propose we agree on the criteria to re-activate EBANX as primary: auth rate sustained above 80% for at least 5 consecutive days, no code 91 spikes, and written confirmation of the fix from EBANX. Do you agree with those parameters?"

**Closing tone:**
> "I know this kind of situation generates noise. What I want you to take from this call is this: the problem is identified, it's operationally mitigated, and we're on top of the definitive close with EBANX. I won't leave you waiting for updates — you'll know what I know, when I know it."

---

### Objection Handling

**"Why did this happen? Shouldn't you have detected it earlier?"**
> "That's a fair question. EBANX made an update on November 15 without proactive notification to the partner ecosystem. From Yuno we detected it through decline code monitoring when the code 91 spike became statistically significant. What has changed: we're establishing more aggressive alerts on acquirer AR degradation to detect these patterns in 24-48 hours, not days."

**"Should we remove EBANX permanently?"**
> "It's a question worth asking, and I want to give you an honest answer. Right now, removing EBANX permanently would be a reactive decision based on a single incident. What I recommend: keep dLocal as primary for the next 30-60 days, evaluate comparative performance with data, and make a processor strategy decision with information, not with the noise of the incident."

**"What does StreamVibe engineering need to do?"**
> "Nothing. The switch to dLocal was done in Yuno's orchestration layer. No new tickets, no API changes, no integration work required. That's exactly what you're using the platform for."

**If they mention revenue loss or compensation:**
> "That's a conversation we can have. Let's first resolve the technical problem and have the complete data on the actual impact — that gives us a solid foundation for any conversation of that kind."

---

### Call Success Signal

- Claudia says: *"Ok, so we're covered for now."*
- Rafael says: *"Good, I don't need to do anything on my side."*

That is the ideal close.

---

---

# Section 4: Proactive Optimization Recommendations

**Audience:** Claudia Mendez (Head of Payments) · Rafael Santos (CTO)

> The following recommendations are independent of the active Brazil incident. They identify structural opportunities in StreamVibe's payment configuration that are currently leaving revenue on the table — regardless of EBANX's resolution timeline.

---

## 4.1 Recommendation 1: Make PIX the Primary Payment Experience in Brazil

### The Opportunity

PIX is already StreamVibe's highest-approval payment method in Brazil: **93.0%** — 30 points above cards even in a normal period (82.5%), and 30 points above where cards are today (63.0%). In just 6 weeks since launch, PIX represents 24% of Brazil's volume, confirming organic adoption is strong.

The opportunity is to stop treating PIX as a secondary option and structurally position it as the first checkout choice.

### The Data

| Payment Method | Approval Rate | Attempts | % of Volume |
|----------------|---------------|----------|-------------|
| Cards | 63.0% (crisis) / ~82.5% (normal) | 19,200 | 67.5% |
| **PIX** | **93.0%** | **6,850** | **24.1%** |
| Boleto | 78.3% | 2,400 | 8.4% |

If PIX's share grows from 24% to 40% of Brazil volume — a realistic 6-month target given its adoption pace — StreamVibe gains ~2,300 additional monthly transactions processed at 93% approval instead of ~83% for cards. At a $9.99 average ticket, this represents approximately **+$190 in monthly approvals per 1,000 Brazilian subscribers** migrated to PIX, with no processor change.

PIX also settles **instantly** (T+0 vs. T+1 or T+2 for cards), improving StreamVibe's cash flow in Brazil.

### Recommended Actions

**1. Checkout repositioning (product change — low engineering effort):**
Show PIX as the first visible option for Brazilian users in checkout — above cards, not below. Behavioral data from checkout flows indicates that payment method presentation order significantly influences selection, especially for users without a pre-established preference.

**2. PIX Automático for recurring billing (medium engineering effort):**
Brazil launched **PIX Automático** in 2024 — a scheme enabling merchants to initiate recurring PIX debits with prior customer authorization, similar to a direct debit mandate. It is the first payment method in Brazil combining PIX's 93%+ approval rate with fully automated recurring billing.

For a subscription business, this is the highest-priority integration opportunity in Brazil: it eliminates both card decline risk and 3DS authentication friction on renewals. Yuno supports PIX Automático — implementation requires StreamVibe to collect PIX mandates from new subscribers during onboarding.

**3. PIX incentive for annual plan (commercial decision — zero engineering):**
Offer a moderate incentive (e.g., 1 month free or a small discount) to annual subscribers who pay via PIX. This shifts highest-value transactions to the highest-approval channel, improves cash flow (instant settlement of the full year), and reduces renewal churn on $89.99 transactions.

### Estimated Impact

| Action | Volume Shift | AR Effect | Estimated Revenue Impact/Month |
|--------|-------------|-----------|-------------------------------|
| PIX as checkout default | +5-10pp PIX share | +93% on shifted volume vs ~83% cards | +$2,000–$5,000/month |
| PIX Automático for renewals | Converts recurring card charges | Eliminates card declines on renewals | +$8,000–$20,000/month (post-integration) |
| PIX incentive for annual plan | Migrates ~20% annual subscribers to PIX | Eliminates 3DS friction on $89.99 transactions | Reduces est. 2-3% churn in annual cohort |

**StreamVibe engineering effort:** Low for checkout repositioning (UI change). Medium for PIX Automático (new integration — Yuno provides SDK support and technical spec).

---

## 4.2 Recommendation 2: Address Argentina by Solving the Right Problem — Wallet as Default + Pricing Strategy

### The Opportunity

Argentina's card approval rate (54%) is not improvable through routing changes — it is at the market floor. However, StreamVibe's **Mercado Pago wallet** is performing at **66.1%** — 15 points above cards, above market benchmark (64-70%), and completely underutilized. The path to improving Argentina's overall approval rate is shifting the subscriber mix toward the wallet, not changing processors.

There is also a structural pricing problem that is independent of payment method but feeds the decline pattern: annual inflation in Argentina exceeds 100%. A subscriber who signed up when the peso was at X now faces the same USD charge with materially eroded purchasing power. This partly explains why code 51 (insufficient funds) is the dominant decline — it's not a bank issuer policy, it's the subscriber's financial capacity.

### The Data

| Payment Method | Approval Rate | Benchmark | Differential vs Cards |
|----------------|---------------|-----------|----------------------|
| Cards (Mercado Pago) | 51.0% | 52-58% | — |
| **Mercado Pago wallet** | **66.1%** | 64-70% | **+15.1pp** |

If wallet share in Argentina volume grows from current (~20%) to 40%, StreamVibe gains ~2,200 additional transactions processed at 66% instead of 51%. At $9.99 average ticket, this is approximately **+$330 in monthly approvals per 1,000 Argentine subscribers** migrated to wallet — with no processor change.

### Recommended Actions

**1. Wallet as primary option in Argentina (product change — low engineering effort):**
In checkout for Argentine users, present Mercado Pago wallet as the first option — above card data entry. Most online buyers in Argentina have a Mercado Pago account. This is a market-specific checkout configuration change.

**2. Wallet enrollment prompt during signup (product change — low effort):**
For new Argentine subscribers, add a step in the signup process inviting them to link their Mercado Pago wallet. Explain the user benefit (faster checkout, no need to enter card data). This builds a wallet-first subscriber base in Argentina going forward.

**3. Evaluate ARS pricing or local price segmentation (commercial decision):**
This recommendation is outside the payments infrastructure, but important to flag: a significant portion of card declines in Argentina are code 51 (insufficient funds), driven by purchasing power erosion against USD pricing. Two options to consider:
- Introduce an ARS price tier that adjusts periodically with inflation (standard practice for streaming services in Argentina — Netflix, Spotify, and Disney+ use local pricing)
- Offer a lower-cost entry tier specific to Argentina

**4. Retry dunning strategy for soft declines in Argentina:**
Code 51 (insufficient funds) is a soft decline — it is retryable. Best practice for subscription businesses is implementing a dunning sequence: retry on day 3, day 7, and day 14 after a failed renewal, as the subscriber may have received their paycheck or topped up their account. Yuno can configure this retry cadence. There is currently no evidence that StreamVibe has a structured retry flow in Argentina.

### Estimated Impact

| Action | Volume Shift | AR Effect | Estimated Revenue Impact/Month |
|--------|-------------|-----------|-------------------------------|
| Wallet as checkout default | +10-20pp wallet share | +15pp on shifted volume | +$1,500–$4,000/month |
| Code 51 retry dunning | Captures recoverable declines | Recovers est. 10-15% of soft declines | +$800–$2,000/month |
| Local pricing (ARS tiers) | Reduces code 51 at source | Structural AR baseline improvement | Strategic — not quantifiable short-term |

**StreamVibe engineering effort:** Low for checkout repositioning. Minimal for retry cadence (Yuno configuration). Medium for local pricing tier (product and finance decision, backend pricing changes).

---

## 4.3 Recommendation 3: Tokenization + MIT for Recurring Billing Across All Markets

### The Opportunity

StreamVibe is a subscription business — the majority of its transactions are recurring renewals, not new card entries. Yet there is no evidence that StreamVibe is using **tokenization** and **MIT (Merchant Initiated Transaction)** flags systematically for subscription renewals. This is the highest-leverage technical configuration available across all five markets.

What the tokenization + MIT combination enables:

**Higher approval rates on renewals:** Issuing banks treat tokenized MIT transactions differently from standard card-not-present transactions. A recurring charge from a known, tokenized card is statistically less likely to be declined because the bank already knows the card and the transaction pattern is predictable.

**3DS exemption eligibility for recurring transactions:** Both Brazilian regulation and EMVco standards allow 3DS exemptions for subsequent recurring payments, once the initial transaction (CIT — Cardholder Initiated Transaction) has been authenticated. If StreamVibe is triggering 3DS on every monthly renewal from established subscribers, it is generating unnecessary friction — and with EBANX's current 3DS infrastructure issue, also unnecessary risk. Transactions flagged as MIT are eligible for mandatory 3DS challenge exemption.

**Reduced PCI scope:** Tokenization means StreamVibe never stores raw card data — the token is stored and used for future charges. This simplifies PCI DSS compliance and reduces risk surface area.

**Retry resilience:** A soft decline on a tokenized card can be retried immediately without requiring the subscriber to re-enter their data. This is critical for recovering the ~18% of EBANX declines that are code 51 (insufficient funds, retryable).

### Current State (Inferred)

Based on the decline pattern — particularly the code 91 spike correlated with 3DS on what should be established subscribers — StreamVibe is likely flagging recurring renewals as CIT instead of MIT, triggering 3DS challenges on subscribers with prior history. This is both a friction and approval rate problem.

### Recommended Actions

**1. Tokenize all cards at initial subscription (immediate priority):**
Ensure that when a subscriber enters their card at signup (CIT), the card is tokenized through Yuno's network tokenization. This token is used for all subsequent renewals. Yuno supports network tokenization for Visa, Mastercard, and Amex across all five StreamVibe markets.

**2. Flag subscription renewals as MIT:**
All charges after the initial signup (monthly and annual renewals) should be submitted with the MIT flag and the original transaction reference (the CIT). This:
- Signals to the issuing bank that this is a pre-authorized recurring charge
- Removes the 3DS requirement on renewal (eligible for exemption in Brazil)
- Improves approval rates by 2-5 percentage points on recurring transactions for most LatAm issuers

**3. Implement automatic credential update (account updater):**
A portion of card declines correspond to expired or replaced cards (codes 54 and 14). Yuno's account updater service automatically refreshes stored tokens when a subscriber's card is reissued — without requiring the subscriber to re-enter their data. This reduces involuntary churn from card expiration, particularly relevant for annual plan subscribers.

### Estimated Impact

| Action | Markets | AR Effect | Estimated Revenue Impact/Month |
|--------|---------|-----------|-------------------------------|
| Tokenization + MIT flag for renewals | All (5) | +2-5pp on recurring charges | +$4,600–$11,500/month |
| 3DS exemption for MIT in Brazil | Brazil | Eliminates 3DS friction for established subscribers | +1-3pp on Brazil card renewals |
| Account updater (expired/replaced cards) | All (5) | Recovers ~60-70% of code 14/54 declines | +$500–$1,200/month |
| **Total estimated impact** | | | **+$5,000–$13,000/month** |

**StreamVibe engineering effort:** Medium — requires updating the payment initiation flow to send MIT flags and store token references per subscription. Yuno provides the SDK, token vault, and account updater infrastructure. Estimated development time: 2-3 sprints. This is the highest-ROI engineering investment available in StreamVibe's payment stack.

---

## 4.4 Prioritized Summary Table

| Priority | Recommendation | Effort | Revenue Impact/Month | Time to Value |
|----------|---------------|--------|---------------------|---------------|
| **1** | PIX Automático for recurring billing in Brazil | Medium | +$8,000–$20,000 | 4-6 weeks |
| **2** | Tokenization + MIT for all renewals | Medium | +$5,000–$13,000 | 4-6 weeks |
| **3** | PIX as checkout default (Brazil) | Low | +$2,000–$5,000 | 1-2 weeks |
| **4** | Mercado Pago wallet as default (Argentina) | Low | +$1,500–$4,000 | 1-2 weeks |
| **5** | Retry dunning for code 51 (Argentina) | Low | +$800–$2,000 | 1 week |
| **6** | Account updater (expired/replaced cards) | Low | +$500–$1,200 | 1-2 weeks |

**Combined improvement potential (conservative estimate): +$17,800–$45,200/month across all markets**

Items 1 and 2 have the highest impact ceiling and require medium engineering effort. Items 3-6 are quick wins that require no or minimal engineering changes, activatable in 1-2 weeks.

---

### Suggested Next Steps

1. **This week:** Agree on items 3, 4, and 5 — low effort, high value, no engineering required. Can be activated in days.
2. **Next 2 weeks:** Define scope for items 1 and 2 with Rafael's team. Yuno's integrations team provides the technical spec and SDK documentation.
3. **30-day review:** Re-evaluate Argentina card performance after activating wallet as default. If still below 54%, initiate the conversation on local pricing strategy with Claudia and the finance team.

---

*Document prepared by Yuno TAM Team — November 25, 2024*
*For internal use and authorized merchant distribution only.*
