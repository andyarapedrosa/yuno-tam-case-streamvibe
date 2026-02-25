# Sección 2: Plan de Acción Técnico

**Merchant:** StreamVibe
**Fecha:** 25 de noviembre, 2024
**Incidente:** Caída en tasa de autorización de tarjetas Brasil — regresión 3DS2 de EBANX
**Preparado por:** Equipo TAM Yuno
**Audiencia:** Interno + StreamVibe (Claudia Mendez, Rafael Santos)

---

## Descripción General

Este plan de acción aborda dos workstreams en paralelo:

- **Workstream A — Incidente activo (Brasil):** Estabilizar las tasas de autorización mientras EBANX resuelve la causa raíz, luego restaurar la configuración de routing óptima con salvaguardas.
- **Workstream B — Hardening estructural:** Implementar cambios que prevengan que esta clase de fallo se repita, independientemente del adquirente involucrado.

Argentina queda excluida de este plan ya que requiere una discusión estratégica separada — no es una respuesta a un incidente técnico.

---

## Matriz de Prioridades

| Prioridad | Acción | Owner | Timeline | Impacto Esperado |
|-----------|--------|-------|----------|-----------------|
| P0 | Mantener dLocal como primario en Brasil | Yuno | ✅ Hecho | Estabiliza AR en ~82.7% |
| P0 | Escalación formal a EBANX con paquete de evidencia completo | Yuno | 24h | Desbloquea el timeline de resolución |
| P1 | Definir criterios de reactivación de EBANX | Yuno + StreamVibe | 48h | Evita un rollback prematuro |
| P1 | EBANX: RCA formal + ETA del fix | EBANX | 48–72h | Cierra el loop de causa raíz |
| P2 | Lógica de reintentos para rechazos blandos código 91 | Yuno | 1 semana | Recupera rechazos reintentables |
| P2 | Split de routing en Brasil post-resolución | Yuno | Post-fix | Elimina exposición a adquirente único |
| P3 | Monitoreo de salud del adquirente + auto-failover | Yuno | 2–3 semanas | Reduce tiempo de detección de días a horas |
| P3 | Revisión del umbral 3DS para Brasil | Yuno + StreamVibe | 2–3 semanas | Reduce fricción innecesaria en planes anuales |

---

## Workstream A: Respuesta al Incidente Activo

### A1 — Mantener dLocal como Primario en Brasil [P0 — HECHO]
**Owner:** Yuno
**Estado:** ✅ Activo
**Acción:** dLocal maneja ahora el 100% del tráfico de tarjetas de Brasil. No se requieren cambios de ingeniería en StreamVibe.
**Impacto esperado:** La tasa de autorización de tarjetas Brasil se estabiliza en ~82.7% — equivalente al rendimiento actual de dLocal, recuperando aproximadamente **3,744 aprobaciones perdidas/mes** vs. el estado actual de EBANX.
**Nota:** No revertir a EBANX hasta que los criterios de reactivación (A3) estén formalmente cumplidos y validados con datos.

---

### A2 — Escalación Formal a EBANX [P0 — En Curso]
**Owner:** Yuno
**Timeline:** 24 horas
**Acción:** Presentar una escalación formal P1 al TAM/soporte de EBANX con el siguiente paquete de evidencia:

Evidencia a incluir:
- Tasa de autorización de tarjetas Brasil por adquirente: EBANX 54.6% vs. dLocal 82.7% (mismo período, mismos BINs)
- Tendencia del código de rechazo 91: 412 → 3,825 (+829%), representando el 62.8% de todos los rechazos de EBANX
- Desglose por BIN de emisor: los 5 principales emisores en ~54-55% simultáneamente
- Desglose por red de tarjeta: Visa, Mastercard, Amex todos igualmente afectados
- Cronología del incidente anclada en la entrada del changelog DS v2.2 del 15 nov

Solicitudes específicas a EBANX:
1. Reconocimiento formal de la regresión correlacionada con la actualización DS v2.2 del 15 nov
2. Análisis de causa raíz con detalle técnico sobre el mecanismo de timeout
3. ETA del fix con hitos intermedios
4. Confirmación de si el problema afecta solo transacciones autenticadas por 3DS o todas las autorizaciones de tarjeta
5. Evaluación de viabilidad de rollback (¿puede revertirse el DS v2.2 mientras se desarrolla el fix?)
6. Protocolo post-incidente: ¿qué notificación de cambios se compromete a dar EBANX para futuras actualizaciones de infraestructura?

**Output esperado:** Ticket P1 de EBANX reconocido en 24h, RCA + timeline de fix en 72h.

---

### A3 — Definir Criterios de Reactivación de EBANX [P1]
**Owner:** Yuno (propone) + StreamVibe (aprueba)
**Timeline:** Acordar en 48 horas
**Acción:** Establecer criterios medibles y objetivos que deben cumplirse antes de restaurar EBANX como primario en Brasil. Estos criterios protegen a StreamVibe de un rollback prematuro que re-exponga el problema.

**Criterios propuestos (a confirmar con StreamVibe):**
- Tasa de autorización de tarjetas Brasil en EBANX ≥ 80% sostenida por **5 días consecutivos** en testing en sombra
- Código de rechazo 91 en o por debajo del baseline (<5% de los rechazos de EBANX)
- Confirmación escrita de EBANX de que la regresión del DS v2.2 está resuelta en producción
- Período de procesamiento paralelo: correr EBANX y dLocal simultáneamente al 20%/80% durante 48h antes de la reactivación completa

**Por qué importa:** Sin criterios explícitos, hay presión para reactivar EBANX en cuanto dicen "está arreglado." Estos criterios garantizan que validemos con datos, no con promesas.

---

### A4 — EBANX: Análisis de Causa Raíz + Fix [P1]
**Owner:** EBANX (Yuno hace seguimiento y escala)
**Timeline:** RCA en 72h, ETA del fix en 1 semana
**Acciones requeridas de EBANX:**
- Identificar el mecanismo exacto: qué componente del DS v2.2 está generando el timeout (configuración del connection pool, routing de endpoints, parámetros del handshake TLS, o mapeo de URLs de ACS)
- Determinar si un rollback al DS v2.1 es factible como medida provisional
- Confirmar el alcance: solo el flujo 3DS o también el path de autorización general
- Proveer ETA del fix y ventana de despliegue con aviso de control de cambios a Yuno

**Ruta de escalación si EBANX no responde en 72h:** Escalar al equipo de Partner Integrations de Yuno para involucrarse con EBANX a nivel de partnership, no solo de soporte.

---

### A5 — Lógica de Reintentos para Rechazos Blandos Código 91 [P2]
**Owner:** Yuno
**Timeline:** En 1 semana
**Acción:** El código 91 (timeout del emisor) es un **rechazo blando** — es reintentable. Actualmente, cuando EBANX devuelve código 91, la lógica de routing de Yuno puede no estar re-enrutando inmediatamente a dLocal para esa transacción específica. Validar y ajustar el comportamiento de reintento:

- Confirmar que las respuestas de código 91 de cualquier adquirente activan un único reintento inmediato vía el adquirente de fallback disponible
- Confirmar que el reintento ocurre dentro de la misma sesión de checkout (sin requerir que el cliente reinicie el proceso)
- Validar que el reintento no genera riesgo de doble cobro (comportamiento de idempotency key)

**Impacto esperado:** Para rechazos de código 91 reintentables que actualmente no se están recuperando, un reintento en la misma sesión a dLocal podría recuperar un estimado del **15-25% de rechazos blandos**, equivalente a ~200-400 aprobaciones adicionales/mes.
**Requerimiento de ingeniería de StreamVibe:** Ninguno — es un cambio de configuración de routing en Yuno.

---

## Workstream B: Hardening Estructural (Post-Incidente)

### B1 — Split de Routing en Brasil Post-Resolución [P2]
**Owner:** Yuno
**Timeline:** Implementar en 1 semana desde la resolución de EBANX
**Acción:** Una vez que EBANX supere los criterios de reactivación, no restaurar el 100% del tráfico de tarjetas de Brasil a EBANX. Implementar un split permanente:

**Configuración propuesta:**
- dLocal: 50% del tráfico de tarjetas Brasil (co-primario)
- EBANX: 50% del tráfico de tarjetas Brasil (co-primario)
- Fallback: dLocal maneja rechazos de EBANX, EBANX maneja rechazos de dLocal

**Por qué 50/50 y no la configuración anterior 70/30:**
La configuración anterior (70% EBANX, 30% dLocal fallback) creó riesgo de concentración — cuando EBANX se degradó, el 70% del volumen de tarjetas quedó inmediatamente expuesto. Un split equilibrado significa que un fallo de adquirente único degrada a lo sumo el 50%, no el 70%+.

**Impacto esperado:** Reduce el blast radius de cualquier futuro incidente de adquirente único en ~40%. También habilita la comparación continua de rendimiento A/B entre adquirentes.
**Requerimiento de ingeniería de StreamVibe:** Ninguno.

---

### B2 — Monitoreo de Salud del Adquirente + Auto-Failover [P3]
**Owner:** Yuno
**Timeline:** 2–3 semanas
**Acción:** Implementar monitoreo en tiempo real de salud del adquirente con triggers de failover automatizados:

- Umbral de alerta: si la tasa de autorización por hora de cualquier adquirente cae >5pp por debajo de su baseline de 7 días → disparar alerta al TAM + ops de Yuno
- Umbral de failover duro: si la tasa de autorización por hora de cualquier adquirente cae >15pp por debajo del baseline → enrutar automáticamente el 100% del tráfico de ese adquirente al fallback
- Monitoreo específico por código: spike en código 91 o código 05 por encima de 2× baseline → alerta, independientemente de la tasa general de autorización

**Contexto:** En este incidente, la tasa de EBANX se degradó desde el 15 de nov hasta el 21 de nov antes de ser detectada. Esta herramienta habría marcado la anomalía en 24-48 horas de la actualización del 15 de nov, no 6 días después.

**Impacto esperado:** Reduce el tiempo de detección a mitigación de ~6 días a <24 horas para futuros incidentes del lado del adquirente.
**Requerimiento de ingeniería de StreamVibe:** Ninguno — configuración de dashboard/alertas en el lado de Yuno.

---

### B3 — Revisión del Umbral 3DS para Brasil [P3]
**Owner:** Yuno + StreamVibe
**Timeline:** 2–3 semanas (post-estabilización del incidente)
**Acción:** El trigger 3DS actual en Brasil está fijado en **>$50 USD**. El pricing de StreamVibe es:
- Plan mensual: $9.99 → **por debajo del umbral** → sin 3DS requerido
- Plan anual: $89.99 → **por encima del umbral** → 3DS activado

Esto significa que la fricción 3DS está actualmente concentrada en las transacciones de mayor valor de StreamVibe (suscriptores anuales). Dos optimizaciones a evaluar:

**Opción 1 — Ajuste del umbral:**
Revisar si el umbral de $50 refleja los mandatos reales de los emisores por BIN, no solo una regla genérica. Algunos emisores brasileños exigen 3DS para transacciones card-not-present por encima de cierto monto (varía por emisor). Revisar el mandato real por rango de BIN vs. la regla general actualmente aplicada.

**Opción 2 — Exenciones 3DS para transacciones recurrentes:**
La regulación brasileña permite exenciones de 3DS para pagos recurrentes posteriores una vez que la transacción inicial ha sido autenticada (MIT — Merchant Initiated Transaction). Si StreamVibe está activando 3DS en cada renovación mensual del mismo suscriptor, esto es fricción innecesaria y una fuga de tasa de aprobación resoluble.

**Impacto esperado:** Reducir los challenges 3DS innecesarios en renovaciones anuales podría mejorar la tasa de EBANX en un estimado de 2-4pp una vez resuelta la regresión. También reduce el abandono por fricción 3DS.
**Requerimiento de ingeniería de StreamVibe:** Menor — puede requerir actualizar el flag de iniciación de pago para transacciones recurrentes (MIT vs. CIT). Yuno provee guía de integración.

---

## Resumen de Impacto

| Acción | Impacto en Tasa de Autorización | Impacto en Ingresos | Timeline |
|--------|---------------------------------|---------------------|----------|
| A1 (dLocal como primario) | Tarjetas Brasil: ~82.7% (vs 63% actual) | Recupera ~$37K–$130K/mes | ✅ Activo ahora |
| A5 (Lógica de reintentos código 91) | +15-25% de rechazos blandos recuperados | +~$2K–$5K/mes incremental | 1 semana |
| B1 (Split de routing) | Reduce blast radius futuros incidentes ~40% | Previene impacto del próximo incidente | Post-fix |
| B3 (Umbral 3DS) | +2-4pp en renovaciones anuales EBANX post-fix | +~$3K–$8K/mes incremental | 2-3 semanas |

---

## Resumen RACI

| Acción | Yuno | StreamVibe | EBANX |
|--------|------|------------|-------|
| dLocal como primario (A1) | **R** | I | — |
| Escalación EBANX (A2) | **R** | I | A |
| Criterios de reactivación (A3) | **R** | **A** | I |
| Causa raíz + fix (A4) | I/escala | I | **R** |
| Lógica de reintentos código 91 (A5) | **R** | I | — |
| Split de routing (B1) | **R** | I | — |
| Monitoreo de salud (B2) | **R** | I | — |
| Revisión umbral 3DS (B3) | **R** | **A** | — |

*R = Responsable, A = Aprueba, I = Informado*

---

## Qué Necesita Hacer Ingeniería de StreamVibe

**Inmediato:** Nada. Todas las acciones de mitigación son cambios de routing en el lado de Yuno.

**En 2-3 semanas (esfuerzo bajo):**
- Revisar y aprobar los criterios de reactivación de EBANX (decisión de negocio, no de ingeniería)
- Evaluar la implementación del flag MIT para transacciones recurrentes (B3) — cambio menor de integración, Yuno provee guía

**Opcional pero recomendado:**
- Monitorear los datos de churn de suscriptores para el período del 21 de nov al presente. Los clientes que tuvieron una renovación de tarjeta fallida y no reintentaron representan una cohorte de churn recuperable — una campaña de win-back dirigida (prompt de reintento, descuento, método de pago alternativo) puede recuperar una parte. Esta es una decisión de producto/marketing, no de pagos.

---

*Próxima actualización: 27 de noviembre, 2024 — o antes si EBANX provee respuesta de RCA.*
