# Section 4: Proactive Optimization Recommendations

**Merchant:** StreamVibe
**Date:** November 25, 2024
**Prepared by:** Yuno TAM Team
**Audience:** Claudia Mendez (Head of Payments) · Rafael Santos (CTO)

> These recommendations are independent of the active Brazil incident. They address structural opportunities in StreamVibe's payments configuration that are leaving measurable revenue on the table today — regardless of EBANX's resolution timeline.

---

## Recommendation 1: Make PIX the Default Checkout Experience in Brazil

### The Opportunity

PIX is already StreamVibe's highest-converting payment method in Brazil at **93.0% authorization rate** — 30 points above cards even in a healthy period (82.5%), and 30 points above where cards are today (63.0%). It now represents 24% of Brazil's payment volume after just 6 weeks since launch, which confirms organic adoption is strong.

The opportunity is to stop treating PIX as a secondary option and make it structurally prominent in the checkout experience.

### The Data

| Payment Method | Auth Rate | Attempts | Share of Volume |
|----------------|-----------|----------|-----------------|
| Cards | 63.0% (crisis) / ~82.5% (normal) | 19,200 | 67.5% |
| **PIX** | **93.0%** | **6,850** | **24.1%** |
| Boleto | 78.3% | 2,400 | 8.4% |

If PIX's volume share grows from 24% to 40% of Brazil transactions — a realistic 6-month target for a product with this adoption trajectory — StreamVibe gains ~2,300 additional monthly transactions going through a 93% approval channel instead of a ~83% card channel. At $9.99 average, that's approximately **+$190 in monthly approvals per 1,000 Brazil subscribers** shifted to PIX, with zero processing changes required.

PIX also settles **instantly** (T+0 vs. T+1 or T+2 for cards), reducing StreamVibe's working capital cycle in Brazil.

### Recommended Actions

**1. Checkout positioning (product change — low engineering effort):**
Display PIX as the first and visually prominent option for Brazilian users at checkout — above cards, not below them. A/B test shows that payment method order influences selection significantly for unfamiliar users.

**2. PIX Automático for recurring billing (medium engineering effort):**
Brazil launched **PIX Automático** (recurring PIX) in 2024 — a framework that allows merchants to initiate recurring PIX debits with customer pre-authorization, similar to a direct debit mandate. This is the first payment method in Brazil that combines PIX's 93%+ approval rate with fully automated recurring billing.

For a subscription business, this is the highest-priority integration opportunity in Brazil: it eliminates both card decline risk and 3DS friction on renewals. Yuno supports PIX Automático — implementation requires StreamVibe to collect PIX mandates from new subscribers at signup.

**3. Annual plan PIX incentive (commercial decision — zero engineering):**
Offer a modest incentive (e.g., 1 free month, or a small discount) for annual subscribers who pay via PIX. This shifts high-value transactions to the highest-approval channel, improves cash flow (immediate full-year settlement), and reduces renewal churn from card declines on $89.99 transactions.

### Expected Impact

| Change | Volume Shift | Auth Rate Gain | Estimated Monthly Revenue Impact |
|--------|-------------|----------------|----------------------------------|
| PIX as default checkout | +5-10pp PIX share | +93% on shifted volume vs ~83% card | +$2,000–$5,000/month |
| PIX Automático for renewals | Converts recurring card billing | Eliminates card decline on renewals | +$8,000–$20,000/month (post-integration) |
| Annual plan PIX incentive | Shifts ~20% annual subs to PIX | Eliminates 3DS friction on $89.99 transactions | Reduces churn by est. 2-3% on annual cohort |

**StreamVibe engineering effort:** Low for checkout repositioning (UI change). Medium for PIX Automático (new integration, Yuno provides SDK support).

---

## Recommendation 2: Fix Argentina by Solving the Right Problem — Wallet Default + Pricing Strategy

### The Opportunity

Argentina's 54% card authorization rate is not fixable through routing changes — it is at market floor. But StreamVibe's Mercado Pago **wallet** is performing at **66.1%** — 15 points higher than cards, above the 64-70% benchmark, and completely under-utilized. The path to improving Argentina's overall authorization rate is to shift the subscriber mix toward wallet, not to change acquirers.

There is also a structural pricing problem that is independent of payment method but drives the decline pattern: Argentina's annual inflation exceeds 100%. A subscriber who enrolled at $9.99/month when the peso was at X is now facing the same USD charge when their purchasing power has materially deteriorated. This is partially why code 51 (insufficient funds) is the dominant decline reason — not issuer policy, but subscriber financial capacity.

### The Data

| Payment Method | Auth Rate | Benchmark | Gap vs Cards |
|----------------|-----------|-----------|--------------|
| Cards (Mercado Pago) | 51.0% | 52-58% | — |
| **Mercado Pago Wallet** | **66.1%** | 64-70% | **+15.1pp** |

If wallet's share of Argentina volume grows from its current level (2,240 / 11,280 = ~20%) to 40%, StreamVibe gains ~2,200 additional transactions processed at 66% instead of 51%. At $9.99 average, that is approximately **+$330 in monthly approvals per 1,000 Argentina subscribers** shifted to wallet — with no change to the underlying acquirer.

### Recommended Actions

**1. Wallet as default for Argentina (product change — low engineering effort):**
In the checkout for Argentine users, present Mercado Pago wallet as the primary option — above card entry. Users who have a Mercado Pago account (a large majority of online purchasers in Argentina) will default to the higher-approval path. This is a single-market checkout configuration change.

**2. Wallet enrollment prompt at signup (product change — low effort):**
For new Argentine subscribers, add a step at signup that prompts wallet linkage. Explain the benefit to the user (faster checkout, no card details needed). This builds a wallet-first subscriber base for Argentina going forward.

**3. Evaluate ARS-denominated pricing or local pricing tiers (commercial decision):**
This is outside payments infrastructure but worth flagging: a significant portion of Argentina card declines are code 51 (insufficient funds), driven by the erosion of purchasing power relative to USD-denominated pricing. Two options:
- Introduce a local ARS price tier that adjusts periodically with inflation (common practice for streaming services in Argentina — Netflix, Spotify, Disney+ all use local pricing)
- Offer a lower-cost entry tier for Argentina specifically

This is not a Yuno recommendation per se — it is a product and commercial decision for StreamVibe. But from a payments perspective, optimizing routing and checkout without addressing the pricing mismatch has a ceiling. If a subscriber genuinely cannot afford the charge, no payment method will authorize it.

**4. Retry strategy for Argentina soft declines:**
Code 51 (insufficient funds) is a soft decline — it is retryable. Best practice for subscription businesses is to implement a dunning sequence: retry on day 3, day 7, and day 14 after a failed renewal, as subscribers may have received their salary or topped up their account. Yuno can configure this retry cadence. Currently there is no indication StreamVibe has a structured retry flow in Argentina.

### Expected Impact

| Change | Volume Shift | Auth Rate Effect | Estimated Monthly Revenue Impact |
|--------|-------------|-----------------|----------------------------------|
| Wallet as default checkout | +10-20pp wallet share | +15pp on shifted volume | +$1,500–$4,000/month |
| Retry dunning for code 51 | Captures delayed-recovery declines | Recovers est. 10-15% of soft declines | +$800–$2,000/month |
| Local pricing (ARS tiers) | Reduces code 51 at source | Structural improvement to baseline AR | Strategic — not quantifiable short-term |

**StreamVibe engineering effort:** Low for checkout positioning. Minimal for retry cadence (Yuno configuration). Medium for local pricing tier (product + finance decision, backend pricing changes).

---

## Recommendation 3: Tokenization + MIT for Recurring Billing Across All Markets

### The Opportunity

StreamVibe is a subscription business — the majority of its transactions are recurring renewals, not new card entries. Yet there is no indication that StreamVibe is using **tokenization** and **MIT (Merchant Initiated Transaction)** flags systematically for subscription renewals. This is the single highest-leverage technical configuration change available across all five markets.

Here is what tokenization + MIT unlocks:

**Higher authorization rates on renewals:** Issuers treat tokenized MIT transactions differently from card-not-present transactions. A recurring charge from a known, tokenized card is statistically less likely to be declined because the issuer has seen the card before and the transaction pattern is predictable.

**3DS exemption eligibility for recurring transactions:** Both Brazilian regulations and EMVco standards allow exemptions from 3DS for subsequent recurring payments after the initial CIT (Cardholder Initiated Transaction) is authenticated. If StreamVibe is triggering 3DS on every monthly renewal for existing subscribers, it is adding unnecessary friction — and with EBANX's current 3DS infrastructure issue, it is also adding unnecessary risk. MIT-flagged transactions are exempt from mandatory 3DS challenge.

**Reduced PCI scope:** Tokenization means StreamVibe never stores raw card data — the token is stored and used for future charges. This simplifies PCI DSS compliance and reduces the risk surface.

**Retry resilience:** A soft decline on a tokenized card can be retried immediately without requiring the subscriber to re-enter payment details. This is critical for recovering the ~18% of EBANX declines that are code 51 (insufficient funds, retryable).

### The Current State (Inferred)

Based on the pattern of declines — particularly the 3DS-correlated spike in code 91 for what should be repeat subscribers — it is likely that StreamVibe is currently flagging recurring renewals as CIT rather than MIT, and triggering 3DS challenges on established subscribers. This is both a friction and an authorization rate problem.

### Recommended Actions

**1. Tokenize all cards at initial subscription (immediate priority):**
Ensure that when a subscriber enters their card at signup (CIT), the card is tokenized via Yuno's network tokenization. This token is used for all subsequent renewals. Yuno supports network tokenization for Visa, Mastercard, and Amex across all five of StreamVibe's markets.

**2. Flag subscription renewals as MIT:**
All charges after the initial signup (monthly and annual renewals) should be submitted with the MIT flag and the original transaction reference (the CIT). This:
- Signals to the issuer that this is a pre-authorized recurring charge
- Removes the requirement for 3DS on the renewal (eligible for exemption in Brazil)
- Improves approval rates by 2-5pp on recurring transactions across most LatAm issuers

**3. Implement credential-on-file update (account updater):**
A portion of card declines are expired or replaced cards (codes 54 and 14). Yuno's account updater service automatically refreshes stored card tokens when a subscriber's card is reissued — without requiring the subscriber to re-enter their details. This reduces involuntary churn from card expiration, which is particularly relevant for annual subscribers.

### Expected Impact

| Change | Markets | Auth Rate Impact | Estimated Monthly Revenue Impact |
|--------|---------|-----------------|----------------------------------|
| Tokenization + MIT flag for renewals | All 5 | +2-5pp on recurring charges | +$4,600–$11,500/month across all markets |
| 3DS exemption for MIT in Brazil | Brazil | Eliminates 3DS friction on established subscribers | +1-3pp on Brazil card renewals |
| Account updater (expired cards) | All 5 | Recovers ~60-70% of code 14/54 declines | +$500–$1,200/month |
| **Total estimated impact** | | | **+$5,000–$13,000/month** |

**StreamVibe engineering effort:** Medium — requires updating the payment initiation flow to pass MIT flags and store token references per subscription. Yuno provides the SDK, token vault, and account updater infrastructure. Estimated development time: 2-3 sprint cycles. This is the highest-ROI engineering investment available in StreamVibe's payments stack.

---

## Summary: Prioritized Recommendations

| Priority | Recommendation | Effort | Monthly Revenue Impact | Time to Value |
|----------|---------------|--------|----------------------|---------------|
| **1** | PIX Automático for recurring billing in Brazil | Medium | +$8,000–$20,000 | 4-6 weeks |
| **2** | Tokenization + MIT for all subscription renewals | Medium | +$5,000–$13,000 | 4-6 weeks |
| **3** | PIX as default checkout in Brazil | Low | +$2,000–$5,000 | 1-2 weeks |
| **4** | Mercado Pago wallet default in Argentina | Low | +$1,500–$4,000 | 1-2 weeks |
| **5** | Argentina retry dunning for code 51 | Low | +$800–$2,000 | 1 week |
| **6** | Account updater (expired/replaced cards) | Low | +$500–$1,200 | 1-2 weeks |

**Combined potential uplift (conservative): +$17,800–$45,200/month across all markets**

Items 1 and 2 have the highest ceiling and require medium engineering effort. Items 3–6 are quick wins that require minimal or zero engineering changes and can be activated within 1-2 weeks.

---

## Suggested Next Steps with StreamVibe

1. **This week:** Agree on items 3, 4, and 5 — low-effort, high-value, no engineering required. These can be activated in days.
2. **Next 2 weeks:** Scope items 1 and 2 with Rafael's team. Yuno's integration team provides the technical specification and SDK documentation.
3. **30-day review:** Reassess Argentina card performance after wallet default is active. If still below 54%, discuss local pricing strategy with Claudia and the finance team.
