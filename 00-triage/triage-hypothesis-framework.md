# StreamVibe LatAm Authorization Crisis — TAM Triage & Hypothesis Framework

---

## 1. Executive Summary (5-7 bullets)

- **Brazil's authorization crisis is entirely EBANX-specific**: dLocal (fallback) is performing at 82.7% — near-identical to its prior period. This instantly rules out market-wide or issuer-side structural issues in Brazil.
- **The smoking gun is decline code 91 (issuer timeout)**: It spiked +829% at EBANX, from 412 instances to 3,825, now representing 62.8% of all EBANX declines. This is not a normal distribution — it's a systemic failure mode introduced at a specific point in time.
- **EBANX's Nov 15 3DS2 Directory Server v2.2 upgrade is the primary suspect**: Auth rates were stable at 82.8% on Nov 10 and 81.9% on Nov 16, then collapsed to 68.2% on Nov 21 — five days after the upgrade went live. EBANX's own support has acknowledged "intermittent latency on 3DS challenges from Brazilian issuers" since Nov 20.
- **The impact is uniform across ALL issuers and ALL networks** (~54-56% for Visa, MC, Amex; ~54-55% for Banco do Brasil, Itaú, Bradesco, Santander, Nubank): this pattern is diagnostic of a failure *inside* EBANX's 3DS routing layer, not at any individual issuer's ACS endpoint.
- **Brazil's overall auth rate dilution (81.2% → 68.0%) is partially masked by PIX**: PIX has a 93% auth rate on 6,850 attempts — it is healthy and not contributing to the problem. Isolating cards shows the true severity: 82.5% → 63.0% (-19.5pp).
- **Argentina is NOT a technical failure**: StreamVibe's 51% card auth rate via Mercado Pago is within the LatAm subscription benchmark of 52-58% for cards in Argentina. The decline mix (codes 51 + 62) reflects macroeconomic conditions — hyperinflation + capital controls — not a solvable routing or integration issue. Finance's impulse to "disable Mercado Pago" would worsen outcomes (Stripe/dLocal average 48-50% there).
- **Yuno's automatic fallback to dLocal is the only thing preventing a full-blown revenue crisis**: dLocal absorbed ~30% of Brazil card volume at 82.7% AR. Without this, StreamVibe's Brazil card rate would be closer to 54.6% with no mitigation.

---

## 2. Primary vs. Secondary Problems

### Primary Problem (Urgent, ~$38K–$130K/month revenue at risk)
**EBANX's 3DS2 infrastructure regression in Brazil**
- Root cause: EBANX's Nov 15 Directory Server v2.2 upgrade degraded 3DS authentication routing, causing systemic timeout responses (code 91) from Brazilian issuer ACS endpoints.
- The problem is active, worsening (Nov 21: 68.2% → Nov 24: 54.6%), and fully attributable to EBANX.
- **This is fixable** — either by EBANX resolving the DS routing regression or by Yuno temporarily re-routing Brazil card primary to dLocal.

### Secondary Problems (Chronic, structural, lower priority)
1. **Argentina auth rate stagnation (54%, 8 weeks)**: Market-baseline issue. No technical fix will move the needle materially. Optimization is about retention tactics and wallet shift, not routing changes.
2. **Brazil card volume concentration risk**: Routing 70% of Brazil cards to a single acquirer (EBANX) with no automated cap created a single point of failure. A proactive routing split should be implemented post-incident.
3. **3DS threshold calibration**: The current ">$50 USD" rule for 3DS in Brazil may be misaligned with StreamVibe's subscription price points ($9.99 monthly is below threshold; $89.99 annual triggers 3DS), creating selective friction on high-value renewals.

---

## 3. Brazil Authorization Rate Drop — Hypotheses (Ranked)

### Hypothesis 1: EBANX 3DS2 Directory Server v2.2 Regression [HIGH CONFIDENCE — PRIMARY CAUSE]

**Evidence:**
- Nov 15: EBANX deployed DS v2.2 upgrade for Visa/Mastercard Brazil
- Nov 21: Code 91 spikes from ~3% of declines to 61% of declines
- Code 91 = issuer ACS timeout during 3DS challenge — the DS is the intermediary between EBANX and issuer ACS
- Auth rate drop is perfectly uniform across ALL 5 major issuers (Banco do Brasil, Itaú, Bradesco, Santander, Nubank all at ~54-55%) — only explainable by a failure in the shared infrastructure layer (EBANX's DS), not at individual issuers
- Auth rate drop is uniform across ALL 3 networks (Visa 54%, MC 55%, Amex 56%) — DS v2.2 was specifically for Visa/Mastercard; Amex also affected suggests broader DS routing or latency issue
- dLocal is unaffected (82.7%) — same Brazilian issuers, same card BINs, different 3DS infrastructure
- EBANX support (in a separate merchant conversation) confirmed "intermittent latency on 3DS challenges from Brazilian issuers since Nov 20"
- EBANX status page shows no declared incident (typical — latency issues often go undeclared until confirmed)

**Mechanism:** The DS v2.2 upgrade likely changed endpoint routing, timeout parameters, or protocol handshake with Brazilian issuer ACS servers. When 3DS challenge is initiated, the ACS request times out → issuer returns code 91 → transaction declines. The 5-day lag between the upgrade (Nov 15) and the collapse (Nov 21) could be explained by: gradual rollout, issuer ACS connection pool exhaustion, or the Banco do Brasil maintenance window on Nov 20 exposing a pre-existing latency issue in EBANX's 3DS layer.

**Weakness:** We don't have a breakdown of declines by transaction amount (above/below $50 threshold). If 3DS is only applied to $89.99 annual transactions, the ~45% decline rate on all EBANX cards would imply most volume is annual — plausible but unconfirmed.

---

### Hypothesis 2: EBANX Infra-Wide Latency Issue (Not 3DS-Specific) [MEDIUM CONFIDENCE — SECONDARY OR AMPLIFYING CAUSE]

**Evidence:**
- EBANX deployed an "infrastructure update" on Nov 15 broadly, not just 3DS
- If the latency is in EBANX's general authorization API (not only 3DS), it would explain why even non-3DS transactions (<$50) are timing out
- Code 91 can mean general issuer communication timeout, not just 3DS-specific ACS timeout

**Why it might explain the full picture:** Monthly subscribers ($9.99 < $50 threshold) would not trigger 3DS but could still be declining at 54%. If hypothesis 1 alone were true, we'd expect only annual subscribers to decline sharply, with monthly near-normal. The uniform 54% rate suggests either (a) most volume is annual, or (b) the EBANX issue extends beyond 3DS.

**Weakness:** More speculative — no direct evidence of EBANX general infra issues beyond the 3DS update.

---

### Hypothesis 3: Banco do Brasil Maintenance Residual Impact [LOW CONFIDENCE — NOT THE ROOT CAUSE]

**Evidence:**
- BdB announced maintenance for Nov 20-21, which coincides with the crash on Nov 21
- Maintenance was marked complete by Nov 21 evening

**Why this is NOT the root cause:**
- BdB (BIN 4011xx) declined at 55% — identical to Itaú (54%), Bradesco (54%), Santander (54.1%), and Nubank (55%). If BdB maintenance were the issue, BdB would show a uniquely worse rate.
- dLocal, routing to the SAME BdB issuer endpoint, maintains 82.7% approval rate — definitively ruling out BdB infrastructure as the cause

---

### Hypothesis 4: Oct 31 Frontend Checkout Update Breaking Card Tokenization/Submission [VERY LOW CONFIDENCE]

**Evidence (against):**
- Deployed Oct 31, auth rates were stable for 3+ weeks afterward (EBANX at 82.8% on Nov 10)
- Hard declines (invalid card number = code 14, expired = code 54) are essentially flat (+2.9%, +2.8%)
- If card data were malformed, we'd see spikes in hard decline codes, not code 91

**Verdict:** Not the cause.

---

## 4. Argentina Assessment: Technical Issue vs. Market Baseline

**Verdict: Market baseline. No actionable technical fix exists in the short or medium term.**

| Signal | Observation | Interpretation |
|--------|-------------|----------------|
| Cards via Mercado Pago | 51.0% AR | Within 52-58% benchmark → AT FLOOR |
| Mercado Pago wallet | 66.1% AR | Within 64-70% benchmark → HEALTHY |
| Primary decline codes | 51 (insufficient funds) + 62 (restricted card) | Macroeconomic, not integration |
| Trend (8 weeks) | -1.1pp | Flat, not actively degrading |
| Alternatives (Stripe, dLocal) | 48-50% in Argentina | WORSE than current Mercado Pago |

**The finance team's proposal to disable Mercado Pago should be rejected clearly and firmly.**

---

## 5. Red Herrings and Non-Causal Events

| Event | Date | Why It's a Red Herring |
|-------|------|----------------------|
| **EBANX set as Brazil primary** | Nov 2 | Happened 4 weeks before the crash. Auth rates were stable at ~83% through Nov 16. The routing change is not the cause — it's what makes EBANX the primary surface area for this failure. |
| **StreamVibe checkout UI update** | Oct 31 | 3+ week gap before any rate change. Hard decline codes (14, 54) are flat. |
| **PIX enablement in Brazil** | Nov 8 | PIX is at 93% AR, stable. No effect on card routing or issuer authentication. |
| **Banco do Brasil maintenance window** | Nov 20-21 | Marked complete. All other issuers declined at the same rate. dLocal shows 82.7% on BdB BINs. |
| **Argentina "terrible for weeks"** | Ongoing | At market floor for 8 weeks, not actively worsening. Not a crisis — it's a ceiling. |

---

## 6. Key Metrics & Tables to Use as Evidence in Final RCA

1. **Brazil Cards by Acquirer** → EBANX 54.6% vs. dLocal 82.7% — eliminates all market/issuer explanations
2. **Error Code Distribution (EBANX)** → Code 91: 412 → 3,825 (+829%) — the smoking gun
3. **EBANX by Issuer BIN** → all 5 major issuers at ~54-55% simultaneously — points to shared DS layer
4. **EBANX by Card Network** → Visa, MC, Amex equally affected — Amex not in DS v2.2 scope = broader issue
5. **Incident Timeline** → Nov 15 upgrade → Nov 21 crash (5-day lag)
6. **Brazil by Payment Method** → PIX/Boleto healthy — cards-only crisis
7. **Argentina by Payment Method** → wallet (66%) healthy, cards at benchmark

### Revenue Impact
- Lost EBANX approvals: 13,440 × (83.1% - 54.6%) = **~3,831/month**
- Net lost (after dLocal recovery): **~3,744 approvals/month**
- Conservative revenue impact: **~$37,400/month** (monthly plan avg $9.99)
- Blended impact (70% monthly / 30% annual): **~$127,000/month**
