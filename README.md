# StreamVibe — Crisis de Autorización LatAm: Caso TAM

**Merchant:** StreamVibe (plataforma de streaming de video, América Latina)
**Período analizado:** Últimos 30 días (al 25 de nov de 2024)
**Estado del incidente:** Activo — tarjetas Brasil, exclusivo de EBANX

---

## Contexto

StreamVibe procesa ~$2.3M/mes en Brasil, México, Colombia, Argentina y Chile, con 87,000 suscriptores activos. Hace tres días, su Head of Payments reportó una caída en las tasas de aprobación de Brasil. Este repositorio contiene el análisis técnico completo, el informe del incidente y el plan de acción producidos en respuesta.

## Estructura del Repositorio

Convención de nomenclatura: archivos internos en inglés (`.md`), archivos dirigidos al merchant en español (`-es.md`).

```
├── 00-triage/
│   └── triage-hypothesis-framework.md       # [EN] Triage interno TAM: hipótesis, falsas pistas, mapa de evidencia
│
├── 01-rca/
│   ├── rca-technical-internal.md            # [EN] RCA técnico completo (interno)
│   └── rca-merchant-report-es.md            # [ES] Informe de incidente para el merchant
│
├── 02-action-plan/
│   └── action-plan-technical-es.md          # [ES] Plan de acción técnico con RACI y timelines
│
├── 03-merchant-comms/
│   ├── email-rca-merchant-es.md             # [ES] Email de entrega del RCA al merchant
│   └── speech-pre-call-merchant-es.md       # [ES] Guía de llamada pre-call con stakeholders
│
├── 04-proactive-optimizations/
│   └── proactive-optimizations-es.md        # [ES] Recomendaciones proactivas de optimización
│
└── deliverable/
    └── streamvibe-latam-auth-crisis-final.md # [EN] Documento consolidado final — Secciones 1–4
```

## Hallazgos Clave (TL;DR)

- **La crisis en Brasil es exclusiva de EBANX.** dLocal alcanza 82.7% AR en las mismas tarjetas y emisores.
- **Causa probable:** La actualización del Directory Server v2.2 de EBANX del 15 de nov causó errores sistémicos de timeout en el emisor (código 91, +829%).
- **Tasa de aprobación de tarjetas Brasil:** cayó de 82.5% → 63.0% (-19.5pp). ~3,744 aprobaciones perdidas/mes neto.
- **Argentina no es un problema técnico.** Está en el piso del mercado (51% vs benchmark 52-58%). Causa macroeconómica.
- **Mitigación inmediata:** Yuno re-enrutó el primario de Brasil a dLocal mientras EBANX investiga.
- **Impacto estimado en ingresos:** $37K–$130K/mes en riesgo (dependiendo del mix de planes).
