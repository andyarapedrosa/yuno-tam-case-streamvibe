# StreamVibe — LatAm Authorization Crisis: TAM Case

**Merchant:** StreamVibe (video streaming platform, Latin America)
**Period analyzed:** Last 30 days (as of Nov 25, 2024)
**Incident status:** Active — Brazil cards, EBANX-exclusive

---

## Context

StreamVibe processes ~$2.3M/month across Brazil, Mexico, Colombia, Argentina, and Chile, with 87,000 active subscribers. Three days ago, their Head of Payments reported a drop in Brazil approval rates. This repository contains the complete technical analysis, incident report, and action plan produced in response.

## File Naming Convention

Internal files are kept in English (`.md`). Merchant-facing files are in Spanish (`-es.md`).

## Repository Structure

```
├── 00-triage/
│   └── triage-hypothesis-framework.md        # [EN] Internal TAM triage: hypotheses, red herrings, evidence map
│
├── 01-rca/
│   ├── rca-technical-internal.md             # [EN] Full technical RCA (internal)
│   └── rca-merchant-report-es.md             # [ES] Merchant-facing incident report
│
├── 02-action-plan/
│   └── action-plan-technical-es.md           # [ES] Technical action plan with RACI and timelines
│
├── 03-merchant-comms/
│   ├── email-rca-merchant-es.md              # [ES] RCA delivery email to merchant
│   └── speech-pre-call-merchant-es.md        # [ES] Pre-call speech guide for stakeholder call
│
├── 04-proactive-optimizations/
│   └── proactive-optimizations-es.md         # [ES] Proactive optimization recommendations
│
└── deliverable/
    ├── executive-summary-es.md               # [ES] One-page executive summary for merchant sharing
    └── streamvibe-latam-auth-crisis-final.md  # [EN] Consolidated final document — Sections 1–4
```

## Key Findings (TL;DR)

- **Brazil crisis is entirely EBANX-specific.** dLocal reaches 82.7% AR on the same cards and issuers.
- **Probable cause:** EBANX's Nov 15 Directory Server v2.2 upgrade caused systemic issuer timeout errors (code 91, +829%).
- **Brazil card approval rate:** dropped from 82.5% → 63.0% (-19.5pp). ~3,744 net lost approvals/month.
- **Argentina is not a technical problem.** It is at the market floor (51% vs 52-58% benchmark). Macroeconomic cause.
- **Immediate mitigation:** Yuno re-routed Brazil primary to dLocal while EBANX investigates.
- **Estimated revenue at risk:** $37K–$130K/month (depending on plan mix).
