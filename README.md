# StreamVibe LatAm Authorization Crisis — TAM Case Study

**Merchant:** StreamVibe (video streaming, Latin America)
**Period analyzed:** Last 30 days (as of Nov 25, 2024)
**Incident status:** Active — Brazil cards, EBANX-specific

---

## Context

StreamVibe processes ~$2.3M/month across Brazil, Mexico, Colombia, Argentina, and Chile with 87,000 active subscribers. Three days ago, their Head of Payments flagged a drop in Brazil authorization rates. This repository contains the full technical analysis, incident report, and action plan produced in response.

## Repository Structure

```
├── 00-triage/
│   └── triage-hypothesis-framework.md   # Internal TAM triage: hypotheses, red herrings, evidence map
│
├── 01-rca/
│   ├── rca-technical-internal.md        # Full technical RCA (internal, English)
│   └── rca-merchant-report-es.md        # Merchant-facing incident report (Spanish)
│
├── 02-action-plan/                      # (in progress)
│   └── ...
│
├── 03-merchant-comms/                   # (in progress)
│   └── ...
│
└── 04-proactive-optimizations/          # (in progress)
    └── ...
```

## Key Findings (TL;DR)

- **Brazil crisis is EBANX-only.** dLocal achieves 82.7% AR on the same cards/issuers.
- **Root cause:** EBANX's Nov 15 3DS2 Directory Server v2.2 upgrade caused systemic issuer timeout errors (code 91, +829%).
- **Brazil card AR:** dropped from 82.5% → 63.0% (-19.5pp). Net ~3,744 lost approvals/month.
- **Argentina is not a technical issue.** It's at market floor (51% vs 52-58% benchmark). Macroeconomic.
- **Immediate mitigation:** Yuno re-routed Brazil primary to dLocal while EBANX investigates.
