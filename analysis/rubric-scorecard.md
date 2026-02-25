# Rubric Scorecard — StreamVibe LatAm Authorization Crisis
## Consolidated Deliverable Audit

**Document audited:** `deliverable/streamvibe-latam-auth-crisis-final.md`
**Audit date:** November 25, 2024
**Auditor:** Yuno TAM Review

---

## Scoring Dimensions — PASS / FAIL Checklist

---

### D1 — Problem Scoping & Prioritization
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Primary problem (Brazil) correctly isolated from secondary (Argentina) | ✅ | Clear separation maintained throughout all sections |
| Urgency calibrated to revenue exposure | ✅ | $37K–$130K/month range anchored in Section 1 intro and exec intro |
| PIX/Boleto correctly excluded from scope | ✅ | §1.1 table rules this out with data |
| Argentina not treated as an incident requiring routing action | ✅ | §1.8 and §3.2 both correctly diagnose as market baseline |
| Finance team's Mercado Pago proposal explicitly rejected with data | ✅ | §1.8 and §3.3 both address this directly |

---

### D2 — Diagnostic Isolation Methodology
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Three-layer isolation used (payment method → acquirer → network/BIN) | ✅ | §1.1 → §1.2 → §1.3 follows the methodology |
| dLocal used as the control variable to eliminate market/issuer causes | ✅ | "Same cards, same issuers, 28.1pp gap" — §1.2 |
| Impact uniformity across issuers and networks used as diagnostic signal | ✅ | §1.3: 54-56% convergence across 5 issuers and 3 networks |
| Hypotheses ranked by confidence | ✅ | Triage file: Hypothesis 1–4 with confidence levels |
| Alternative hypotheses formally falsified (not just ignored) | ✅ | §1.6 dedicates a section to each ruled-out event |
| Red herrings distinguished from contributing factors | ✅ | BdB maintenance correctly classified as stress trigger, not root cause |

---

### D3 — Evidence Quality & Quantification
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Every major claim is backed by a data point | ✅ | All AR changes are percentage-point deltas with attempt counts |
| Code 91 spike (+829%) used as forensic centerpiece | ✅ | §1.4 table + narrative |
| Normal decline codes (51, 05, 14, 54, 62) benchmarked to show code 91 is anomalous | ✅ | §1.4: all others at +1-3% |
| Attempt volumes included (not just percentages) | ✅ | §1.1–1.3 tables include attempt counts |
| Evidence chain explicitly stated in hypothesis section | ✅ | §1.5: four-point supporting chain |

---

### D4 — Causal Chain Integrity
**Result: ⚠️ CONDITIONAL PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Temporal correlation (Nov 15 → Nov 21) established | ✅ | §1.5: stable Nov 10 and Nov 16, collapse Nov 21 |
| 5-day lag acknowledged as unexplained | ✅ | §1.5: three possible mechanisms named |
| Amex anomaly identified (not in DS v2.2 scope, yet dropped 26pp) | ✅ | §1.5 and §1.9 both flag it |
| **Amex anomaly elevated to a named action item** | ❌ | Buried in "Remaining Uncertainty" — never assigned an owner or timeline in Section 2 |
| **Transaction-amount segmentation gap drives an immediate data request** | ❌ | Flagged as unknown in §1.9 but no corresponding P0/P1 action in Section 2. If $9.99 transactions are also declining at 54%, the failure scope is not 3DS — it changes the entire root cause picture |
| EBANX acknowledgment footnoted as internal-only | ✅ | "(internal use only)" note in §1.5 |

**Why conditional:** Two open questions — Amex path and transaction-amount segmentation — are correctly identified but left dangling. For top quartile, each must trigger a named action with an owner and a 24-48h deadline.

---

### D5 — Revenue Impact Quantification
**Result: ⚠️ CONDITIONAL PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Net approvals lost calculated (not just EBANX drop) | ✅ | §1.7: -3,744/month after dLocal partial recovery |
| Conservative scenario present ($9.99 avg) | ✅ | ~$37,402/month |
| Blended scenario present (70/30 monthly/annual) | ✅ | ~$127,000/month |
| **Churn-adjusted scenario present** | ❌ | The 1.5×–2.5× multiplier is mentioned in §1.7 prose but never applied to produce a third scenario row. This is the number CFOs and VPs care about most — it's the long-term P&L exposure, not just the missed charges |
| Pre/current period calculation shown transparently | ✅ | Math shown inline: 13,440 × 83.1% vs 13,440 × 54.6% |
| Downstream churn explicitly acknowledged as not quantified | ✅ | §1.7 final paragraph |

**Why conditional:** Mentioning the multiplier without applying it is the analytical equivalent of naming the punchline without landing it. A third row in the revenue table ("Churn-adjusted conservative: ~$56K–$94K/month") converts a hedge into a decision-support number.

---

### D6 — Action Plan Completeness & Specificity
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| P0/P1/P2/P3 priority tiers used | ✅ | §2.1 priority matrix |
| Every action has a named owner | ✅ | RACI in §2.4 |
| Every action has a timeline | ✅ | 24h / 48h / 1 week / post-fix |
| Reactivation criteria for EBANX defined with objective thresholds | ✅ | §A3: AR ≥80% for 5 consecutive days, code 91 ≤5%, written confirmation |
| EBANX escalation evidence package defined | ✅ | §A2: six specific bullet points |
| "No verbal confirmation" stance explicit | ✅ | "Not just with their confirmation verbal" language in §A3 |
| StreamVibe engineering workload explicitly addressed | ✅ | §2.5: "Immediate: Nothing" |
| **30/60/90-day recovery KPIs defined** | ❌ | Actions have timelines but no success state defined at 30/60/90 days — see Edit #5 |

---

### D7 — Risk Controls & Safeguards
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Protection against premature EBANX rollback | ✅ | A3: 5-day data window + parallel processing period |
| 50/50 routing split proposed instead of restoring 70/30 | ✅ | B1: rationale explicitly stated |
| Single-point-of-failure re-exposure addressed | ✅ | B1 eliminates the pre-incident concentration risk |
| Auto-failover monitoring proposed | ✅ | B2: alert at -5pp, hard failover at -15pp |
| **B2 (auto-failover) priority level reflects actual detection gap** | ❌ | B2 is classified P3 but this was exactly the gap that caused the 6-day detection delay. Should be P2 with explicit callout: "this mechanism would have surfaced the incident within 24-48h of Nov 15." |

---

### D8 — Merchant Communication Fit
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Three communication artifacts present (email, RCA, pre-call) | ✅ | §3.1, §3.2, §3.3 |
| Email concise and action-forward (≤3 key bullets) | ✅ | §3.1 |
| Merchant RCA uses business language, not engineering jargon | ✅ | §3.2: "bank not responding (timeout)" instead of "ACS timeout" |
| Pre-call guide includes objection handling | ✅ | §3.3: four objection scenarios |
| Pre-call success criteria defined | ✅ | §3.3: "Claudia says / Rafael says" |
| External-safe (no competitor %, no internal EBANX acknowledgment) | ✅ | Competitor performance framed as "market benchmark reference" |
| Causation language appropriately hedged for merchant-facing docs | ✅ | "Strongly correlated with" in §3.2 (not "caused by") |
| **Section 3 ordering mirrors logical delivery sequence** | ❌ | Current: email → RCA → pre-call. Logical: pre-call (happens first) → email → RCA. Misalignment creates confusion in operational use |

---

### D9 — Proactive Recommendations
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Recommendations independent of the active incident | ✅ | Framed as structural, not incident response |
| Each recommendation quantifies revenue impact | ✅ | §4.1–4.3 each have impact tables |
| Recommendations prioritized by impact and effort | ✅ | §4.4 summary table with 6-item priority ranking |
| Combined uplift estimate provided | ✅ | +$17,800–$45,200/month |
| Engineering effort explicitly scoped for each rec | ✅ | "Low / Medium" per recommendation |
| **PIX Automático and MIT engineering dependency surfaced** | ❌ | Recs 1 and 3 both modify the signup flow (mandate collection at onboarding for PIX Automático; tokenization at initial CIT for MIT). This is the same engineering entry point. Not flagging this risks two sequential sprints that could be one |
| **Argentina wallet share (~20%) assumption sourced** | ❌ | §4.2 uses "current (~20%)" without a data footnote or "estimated" qualifier |

---

### D10 — Document Structure & Navigability
**Result: ✅ PASS**

| Check | Status | Notes |
|-------|--------|-------|
| Table of contents with section anchors | ✅ | Full TOC with subsection links |
| Executive introduction present | ✅ | Covers situation, root cause, mitigation, revenue at risk |
| Section numbering consistent | ✅ | §1.1–1.9, §2.1–2.5, §3.1–3.3, §4.1–4.4 |
| Audience tag on each section | ✅ | "Internal / Technical" on §1; "Internal + StreamVibe" on §2 |
| Internal-only information confined to internal sections | ✅ | EBANX acknowledgment note in §1, not in §3 |
| **Cross-reference to executive-summary-es.md missing** | ❌ | The one-page summary is a separate deliverable but not referenced in the consolidated doc intro — readers don't know it exists unless they explore the repo |
| **"What does steady state look like?" absent from exec intro** | ❌ | Intro says "recoverable with the actions described herein" but never states the target state: e.g., "Brazil card AR of 82-84% with 50/50 routing split post-recovery" |

---

## Summary Scorecard

| Dimension | Result | Critical Gaps |
|-----------|--------|---------------|
| D1 — Problem Scoping & Prioritization | ✅ PASS | — |
| D2 — Diagnostic Isolation Methodology | ✅ PASS | — |
| D3 — Evidence Quality & Quantification | ✅ PASS | — |
| D4 — Causal Chain Integrity | ⚠️ CONDITIONAL PASS | Amex anomaly not actioned; segmentation gap not actioned |
| D5 — Revenue Impact Quantification | ⚠️ CONDITIONAL PASS | Churn-adjusted scenario missing |
| D6 — Action Plan Completeness | ✅ PASS | 30/60/90-day KPIs missing |
| D7 — Risk Controls & Safeguards | ✅ PASS | B2 priority understated |
| D8 — Merchant Communication Fit | ✅ PASS | Section 3 ordering wrong |
| D9 — Proactive Recommendations | ✅ PASS | Engineering dependency not flagged; wallet share unsourced |
| D10 — Document Structure & Navigability | ✅ PASS | Executive summary not cross-referenced; target state missing from intro |

**Overall: 8 PASS / 2 CONDITIONAL PASS / 0 FAIL**

---

## Top 10 Edits to Reach Top Quartile

---

### Edit 1 — Add a churn-adjusted revenue scenario in §1.7
**Dimension:** D5 | **Effort:** Low

The document correctly states that downstream churn multiplies revenue impact 1.5×–2.5× but never applies it. Add a third row to the revenue table:

| Scenario | Monthly Impact |
|----------|---------------|
| Conservative ($9.99 monthly) | ~$37,402/month |
| Blended (70/30 monthly/annual) | ~$127,000/month |
| **Churn-adjusted conservative (1.5×–2.5×)** | **~$56K–$94K/month** |
| **Churn-adjusted blended (1.5×–2.5×)** | **~$190K–$318K/month** |

This is the P&L exposure number that matters for a CEO or CFO conversation — it's the LTV cost, not just the billing failure cost.

---

### Edit 2 — Elevate the Amex anomaly to a named action item
**Dimension:** D4 | **Effort:** Low

Amex falling ~26pp is currently buried in "Remaining Uncertainty." The implication is significant: if Amex is not in the DS v2.2 scope and it still collapsed, EBANX's issue may extend beyond 3DS into its general authorization API. Add to §2.2 as action A2.x or a sub-bullet under A2:

> **A2.1 — Confirm Amex routing path** | Owner: Yuno | Timeline: 24h
> Request from EBANX: Does Amex flow through the updated DS v2.2, a legacy DS, or a direct authorization path? This confirms or rules out a broader API-layer failure beyond 3DS.

If Amex uses a different path and is still degraded, the mitigation strategy changes.

---

### Edit 3 — Add an immediate data pull action for transaction-amount segmentation
**Dimension:** D4 | **Effort:** Low

§1.9 flags the $9.99 vs $89.99 segmentation as unknown, but there is no corresponding action in Section 2 to close it. Add to §2.2:

> **A2.2 — Request transaction-amount breakdown from EBANX** | Owner: Yuno | Timeline: 24h
> If $9.99 monthly transactions (below the $50 3DS threshold) are also declining at ~54%, the failure is not limited to 3DS routing — it implicates EBANX's general authorization layer. This changes the root cause classification from "3DS regression" to "authorization API degradation" and would require a different fix path from EBANX.

This is the single most diagnostic data point not yet in hand.

---

### Edit 4 — Reorder Section 3 to match operational delivery sequence
**Dimension:** D8 | **Effort:** Low

Current order: 3.1 Email → 3.2 Merchant RCA → 3.3 Pre-Call Guide.
Logical order: 3.1 Pre-Call Guide → 3.2 Email → 3.3 Merchant RCA.

The pre-call happens *before* the email and RCA are sent. The current order implies you send the report and then prepare for the call — the opposite of how TAMs actually work. Reordering makes the section usable as a sequential ops checklist.

---

### Edit 5 — Add 30/60/90-day recovery KPIs to §2 or §2.1
**Dimension:** D6 | **Effort:** Low

The action plan has timelines per action but no definition of what "recovered" looks like at the program level. Add a brief KPI table:

| Milestone | Target | Date |
|-----------|--------|------|
| 30 days | EBANX reactivation decision made (reactivate or permanently remove); B2 monitoring live | Dec 25 |
| 60 days | B1 50/50 routing split live; code 91 rate ≤5% sustained | Jan 25 |
| 90 days | Tokenization + MIT flag implemented; PIX share ≥30% of Brazil volume | Feb 25 |

This converts the action plan from a task list into a recovery program with measurable outcomes.

---

### Edit 6 — Upgrade B2 (auto-failover monitoring) from P3 to P2 in §2.1
**Dimension:** D7 | **Effort:** Low

B2 is currently P3 but it is the direct structural answer to the 6-day detection gap that made this incident so costly. The priority justification should be explicit:

> "Classified as P2, not P3. Had this monitoring been in place on Nov 15, the EBANX degradation would have triggered an alert within 24-48h — not 6 days. The detection gap is estimated to have cost ~$6K–$25K in additional revenue loss beyond what would have been incurred with faster detection."

Upgrading to P2 also signals to StreamVibe that Yuno is investing in prevention, not just reaction.

---

### Edit 7 — Surface the shared engineering entry point for Recs 1 and 3 in §4
**Dimension:** D9 | **Effort:** Low

Recommendation 1 (PIX Automático) requires collecting PIX mandates from new subscribers at onboarding. Recommendation 3 (Tokenization + MIT) requires tokenizing the card at initial signup. Both modifications touch the same engineering entry point: the subscriber signup/onboarding flow.

Add a note in §4.4:

> **Engineering note:** Recommendations 1 (PIX Automático) and 3 (Tokenization + MIT) share the same engineering entry point — the subscriber signup flow. These should be scoped as a single sprint rather than two sequential ones. Yuno will provide a unified integration spec covering both changes to minimize engineering overhead.

This prevents two separate 4-6 week engagements from becoming a 10-12 week sequential project.

---

### Edit 8 — Source the Argentina wallet share (~20%) assumption in §4.2
**Dimension:** D9 | **Effort:** Low

"If wallet share grows from current (~20%) to 40%" — the 20% figure is stated without attribution. Replace with:

> "...from current (~20%, based on attempt volume in the analysis period: approximately 1,400 wallet attempts out of ~7,000 total Argentina attempts)"

If the figure is an estimate and not derived from data, flag it:

> "...from current (~20%, estimated — exact wallet vs. card mix to be confirmed against raw attempt data)"

An unsourced assumption in a quantified impact projection weakens the credibility of the entire recommendation.

---

### Edit 9 — Add target steady-state AR to the executive introduction
**Dimension:** D10 | **Effort:** Low

The intro ends with "recoverable with the actions described herein" but never states what "recovered" means quantitatively. Add one sentence:

> "With the proposed 50/50 EBANX-dLocal routing split, proactive optimizations (PIX as primary, MIT flag for renewals), and the structural hardening measures in Section 2, Brazil's card authorization rate is expected to stabilize at **83-86%** — above the pre-incident baseline."

This closes the loop the exec intro opens and gives leadership a concrete target to track against.

---

### Edit 10 — Cross-reference the executive summary in the consolidated doc
**Dimension:** D10 | **Effort:** Low

`deliverable/executive-summary-es.md` is not referenced anywhere in the consolidated document. A reader of the main file has no way to know it exists. Add a single line at the bottom of the executive introduction:

> "For a one-page summary in Spanish for distribution to StreamVibe stakeholders, see [`executive-summary-es.md`](./executive-summary-es.md)."

This makes the deliverable package self-contained and navigable without requiring the reader to explore the repository structure.

---

## Prioritized Edit Sequence

| Rank | Edit | Dimension | Business Impact |
|------|------|-----------|----------------|
| 1 | Add churn-adjusted revenue scenario | D5 | Closes the most important gap for C-suite conversations |
| 2 | Elevate Amex anomaly to named action | D4 | Could reframe entire root cause if Amex uses separate path |
| 3 | Add immediate segmentation data pull action | D4 | Most diagnostic unknown — determines whether root cause is 3DS or general API |
| 4 | Add 30/60/90-day recovery KPIs | D6 | Converts task list into measurable recovery program |
| 5 | Upgrade B2 to P2 with detection-gap rationale | D7 | Signals preventive investment; justifies the priority shift with math |
| 6 | Reorder Section 3 (pre-call → email → RCA) | D8 | Operational usability — should be immediately sequentially actionable |
| 7 | Surface Rec 1 + Rec 3 shared engineering entry point | D9 | Prevents 10-12 week sequential project from what could be one sprint |
| 8 | Source Argentina wallet share assumption | D9 | Credibility of quantified recommendation depends on input data quality |
| 9 | Add target steady-state AR to exec intro | D10 | Gives leadership a number to manage to |
| 10 | Cross-reference executive-summary-es.md | D10 | Navigability and package completeness |

---

*Edits 1–3 are the highest-signal changes. Edits 1 and 2 alter the analytical conclusions; Edit 3 could change the root cause classification entirely if the data shows $9.99 transactions are also declining.*
