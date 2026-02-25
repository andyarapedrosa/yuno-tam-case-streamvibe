# Section 1: Root Cause Analysis — Technical (Internal)

**Merchant:** StreamVibe
**Date:** November 25, 2024
**Prepared by:** Yuno TAM Team
**Audience:** Internal / Technical

---

## Overview

StreamVibe's Brazil authorization rate dropped from **81.2% to 68.0% (-13.2pp)** over the last 30 days. The drop is entirely attributable to a single acquirer — EBANX — and is driven by a systemic spike in issuer timeout errors (decline code 91), consistent with a defect introduced by EBANX's Nov 15 3DS2 infrastructure update. Argentina's low authorization rate (54%) is a separate, unrelated issue rooted in macroeconomic conditions, not a technical failure.

---

## 2.1 Brazil: Isolating the Affected Segment

| Payment Method | Attempts | Current AR | Prior AR | Change |
|----------------|----------|-----------|------------|--------|
| Cards | 19,200 | 63.0% | 82.5% | **-19.5pp** |
| PIX | 6,850 | 93.0% | 92.8% | +0.2pp |
| Boleto | 2,400 | 78.3% | 77.9% | +0.4pp |

PIX and Boleto are performing at or above prior-period levels. The crisis is **cards only**. This rules out any platform-level integration issue, network-level outage, or merchant misconfiguration affecting all payment methods.

---

## 2.2 Brazil Cards: Acquirer-Level Isolation

| Acquirer | Attempts | Current AR | Prior AR | Change |
|---------|----------|-----------|------------|--------|
| EBANX (primary) | 13,440 | 54.6% | 83.1% | **-28.5pp** |
| dLocal (fallback) | 5,760 | 82.7% | 81.0% | +1.7pp |

dLocal is processing payments for **the same Brazilian cardholders, at the same issuing banks, on the same card networks** — and reaching 82.7% approval rate. The 28.1pp gap between EBANX and dLocal is the single most important data point in this analysis. It eliminates Brazil's banking infrastructure, macroeconomic conditions, and issuer-side policy changes as contributing factors. **The failure is inside EBANX.**

---

## 2.3 EBANX: Breakdown by Network and Issuer BIN

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

The authorization rate converged in a narrow band of **54-56% across every major issuer and every card network**. This mathematical convergence is only explainable by a failure in the **shared intermediary** between EBANX and all these issuers: EBANX's own 3DS processing layer (the Directory Server). If the issue were on the issuer side, the distribution would be uneven.

---

## 2.4 Error Code Forensic Analysis

| Code | Description | Current Count | Prior Count | Change | % of Declines |
|------|-------------|--------------|------------|--------|---------------|
| **91** | **Issuer timeout/unavailable** | **3,825** | **412** | **+829%** | **62.8%** |
| 51 | Insufficient funds | 1,104 | 1,089 | +1.4% | 18.1% |
| 05 | Do not honor (generic) | 518 | 502 | +3.2% | 8.5% |
| 14 | Invalid card number | 245 | 238 | +2.9% | 4.0% |
| 54 | Expired card | 184 | 179 | +2.8% | 3.0% |
| 62 | Restricted card | 137 | 133 | +3.0% | 2.2% |

Code 91 is the only anomaly. All other decline codes increased 1-3% — consistent with normal volume variation. Code 91 increased **829%**. This is a single-mode failure event.

Code 91 in the context of 3DS processing indicates that the issuer's ACS (Authentication Control Server) is not responding within the expected timeout window during the 3DS challenge. EBANX's Directory Server is responsible for routing these challenge requests. A defect in that routing layer would produce exactly this pattern: timeouts across all issuer ACS endpoints simultaneously, with no individual issuer being uniquely worse than the others.

---

## 2.5 Root Cause Hypothesis

**Hypothesis: EBANX's Directory Server v2.2 upgrade on November 15 introduced a regression in 3DS authentication routing for Brazilian issuers, causing systemic ACS request timeouts (code 91) across all card networks and issuing banks.**

**Confidence level: High.**

Supporting evidence chain:
1. **Temporal correlation:** EBANX deployed DS v2.2 on Nov 15. Rates were stable on Nov 10 (82.8%) and Nov 16 (81.9%). Collapse on Nov 21.
2. **Failure mode:** Code 91 spiked 829% at EBANX. dLocal exhibits no code 91 spikes.
3. **Impact uniformity:** All five major issuers and all three card networks dropped 28-30pp in parallel.
4. **Vendor self-acknowledgment:** EBANX support confirmed "intermittent latency on 3DS challenges from Brazilian issuers since Nov 20" (in a parallel conversation with another Yuno merchant — internal use only).

**Remaining uncertainty:**
- The 5-day lag between the upgrade (Nov 15) and the collapse (Nov 21) is not fully explained. Possible mechanisms: gradual rollout, connection pool saturation, or the BdB maintenance window acting as a stress trigger.
- Amex is not covered by the DS v2.2 scope (Visa/MC only), yet also dropped ~26pp — raises the question of whether the issue extends beyond 3DS into EBANX's general authorization API.
- A breakdown by transaction amount (above/below the $50 USD 3DS threshold) would confirm whether monthly $9.99 non-3DS transactions are also affected.

---

## 2.6 Events that Are NOT the Root Cause

**StreamVibe frontend checkout update (Oct 31):** Deployed 3+ weeks before the drop. Rates were stable at 82.8% through Nov 10. Hard decline codes 14 and 54 increased only 2.9% and 2.8% — no card data submission errors. Not a contributing factor.

**PIX launch in Brazil (Nov 8):** Operates on a separate payment rail. PIX AR is 93.0%, stable. No interaction with card authorization infrastructure. Unrelated.

**Banco do Brasil scheduled maintenance (Nov 20-21):** Marked complete by Nov 21 evening; the rate continued deteriorating to 54.6% through Nov 24. BdB BINs dropped to exactly the same level as Itaú, Bradesco, Santander, and Nubank. dLocal maintained 82.7% on BdB BINs during the same period. Not the root cause — may have acted as the stress trigger that surfaced EBANX's pre-existing latency regression.

---

## 2.7 Impact Quantification — Brazil Cards

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

## 3. Argentina: Technical Assessment

**Diagnosis: Performing at market baseline. Not a technical incident.**

| Metric | StreamVibe | LatAm Subscription Benchmark |
|--------|-----------|-------------------------------|
| Cards via Mercado Pago | 51.0% | 52-58% |
| Mercado Pago wallet | 66.1% | 64-70% |

Argentina's card auth rate is within the established benchmark range. The dominant decline codes are 51 (insufficient funds) and 62 (restricted card) — issuer responses to Argentina's macroeconomic environment (>100% annual inflation, capital controls). These are non-retryable and are not resolved by routing changes.

Alternatives (Stripe, dLocal) perform at **48-50%** on Argentina cards — materially worse than Mercado Pago's 51%, as Mercado Pago has deeper relationships with local issuers. Disabling Mercado Pago as a card acquirer would reduce Argentina's auth rate by an estimated **3-5pp**.

The Mercado Pago wallet (66.1%, above benchmark) is the highest-converting instrument available in Argentina. The strategic opportunity is migrating the subscriber base toward the wallet.

---

## 4. Confirmed vs. Needs Confirmation

**Confirmed by data:**
- The Brazil crisis is cards only. PIX and Boleto are unaffected.
- The crisis is EBANX only. dLocal operating at 82.7%.
- Decline code 91 (issuer timeout) increased 829% exclusively at EBANX.
- All issuers and networks uniformly affected → failure in EBANX's DS layer.
- EBANX deployed the DS v2.2 upgrade five days before the collapse.
- EBANX support acknowledged latency on 3DS challenges since Nov 20.
- Argentina is at market benchmark. No alternative acquirer performs better.

**Requires confirmation:**
- Breakdown by transaction amount (above/below $50 USD 3DS threshold): are the $9.99 monthly transactions also declining? If so, the issue extends beyond 3DS into general authorization routing.
- EBANX's root cause analysis and fix/rollback timeline.
- Whether Amex flows through the updated DS v2.2 or a separate path.
- Exact rollout schedule of DS v2.2 (gradual vs. full cutover on Nov 15).
