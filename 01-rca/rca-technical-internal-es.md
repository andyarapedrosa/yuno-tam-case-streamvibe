# Sección 1: Análisis de Causa Raíz — Técnico (Interno)

**Merchant:** StreamVibe
**Fecha:** 25 de noviembre, 2024
**Preparado por:** Equipo TAM Yuno
**Audiencia:** Interno / Técnico

---

## Descripción General

La tasa de autorización de Brasil de StreamVibe cayó del **81.2% al 68.0% (-13.2pp)** en los últimos 30 días. La caída es totalmente atribuible a un único adquirente — EBANX — y está impulsada por un spike sistémico de errores de timeout del emisor (código de rechazo 91), consistente con un defecto introducido por la actualización de infraestructura 3DS2 de EBANX del 15 de noviembre. La baja tasa de autorización de Argentina (54%) es un problema separado y no relacionado, cuya raíz está en condiciones macroeconómicas, no en una falla técnica.

---

## 2.1 Brasil: Aislando el Segmento Afectado

| Método de Pago | Intentos | AR Actual | AR Anterior | Cambio |
|----------------|----------|-----------|-------------|--------|
| Tarjetas | 19,200 | 63.0% | 82.5% | **-19.5pp** |
| PIX | 6,850 | 93.0% | 92.8% | +0.2pp |
| Boleto | 2,400 | 78.3% | 77.9% | +0.4pp |

PIX y Boleto están rindiendo en o por encima de los niveles del período anterior. La crisis es **solo de tarjetas**. Esto descarta cualquier problema de integración a nivel de plataforma, interrupción a nivel de red, o mala configuración del merchant que afecte a todos los métodos de pago.

---

## 2.2 Tarjetas Brasil: Aislamiento a Nivel de Adquirente

| Adquirente | Intentos | AR Actual | AR Anterior | Cambio |
|-----------|----------|-----------|-------------|--------|
| EBANX (primario) | 13,440 | 54.6% | 83.1% | **-28.5pp** |
| dLocal (fallback) | 5,760 | 82.7% | 81.0% | +1.7pp |

dLocal está procesando pagos para **los mismos tarjetahabientes brasileños, en los mismos bancos emisores, en las mismas redes de tarjeta** — y alcanzando una tasa de autorización del 82.7%. La brecha de 28.1pp entre EBANX y dLocal es el dato más importante de este análisis. Elimina la infraestructura bancaria de Brasil, las condiciones macroeconómicas y los cambios de política del lado del emisor como factores contribuyentes. **El fallo está dentro de EBANX.**

---

## 2.3 EBANX: Desglose por Red y BIN de Emisor

**Por Red:**

| Red | Intentos | AR Actual | AR Anterior | Cambio |
|-----|----------|-----------|-------------|--------|
| Visa | 7,392 | 54.0% | 84.2% | -30.2pp |
| Mastercard | 4,838 | 55.0% | 81.8% | -26.8pp |
| Amex | 1,210 | 56.4% | 82.5% | -26.1pp |

**Por Banco Emisor (Rango de BIN):**

| Emisor | Intentos | AR Actual | AR Anterior | Cambio |
|--------|----------|-----------|-------------|--------|
| Banco do Brasil (4011xx) | 2,688 | 55.0% | 85.1% | -30.1pp |
| Itaú (5211xx) | 3,360 | 54.0% | 83.8% | -29.8pp |
| Bradesco (4389xx) | 2,150 | 54.0% | 82.9% | -28.9pp |
| Santander (5543xx) | 1,478 | 54.1% | 83.5% | -29.4pp |
| Nubank (5201xx) | 1,881 | 55.0% | 84.0% | -29.0pp |
| Otros emisores | 1,883 | 55.7% | 82.4% | -26.7pp |

La tasa de autorización convergió en una banda estrecha de **54-56% en cada emisor principal y cada red de tarjeta**. Esta convergencia matemática solo es explicable por un fallo en el **intermediario compartido** entre EBANX y todos estos emisores: la propia capa de procesamiento 3DS de EBANX (el Directory Server). Si el problema estuviera en el lado del emisor, la distribución sería desigual.

---

## 2.4 Análisis Forense de Códigos de Error

| Código | Descripción | Conteo Actual | Conteo Anterior | Cambio | % de Rechazos |
|--------|-------------|--------------|----------------|--------|---------------|
| **91** | **Timeout/no disponible del emisor** | **3,825** | **412** | **+829%** | **62.8%** |
| 51 | Fondos insuficientes | 1,104 | 1,089 | +1.4% | 18.1% |
| 05 | No honor (genérico) | 518 | 502 | +3.2% | 8.5% |
| 14 | Número de tarjeta inválido | 245 | 238 | +2.9% | 4.0% |
| 54 | Tarjeta vencida | 184 | 179 | +2.8% | 3.0% |
| 62 | Tarjeta restringida | 137 | 133 | +3.0% | 2.2% |

El código 91 es la única anomalía. Todos los demás códigos de rechazo aumentaron entre 1-3% — consistente con la variación normal del volumen. El código 91 aumentó **829%**. Es un evento de modo de fallo único.

El código 91 en el contexto del procesamiento 3DS indica que el Authentication Control Server (ACS) del emisor no responde dentro de la ventana de timeout esperada durante el challenge 3DS. El Directory Server de EBANX es responsable de enrutar estas solicitudes de challenge. Un defecto en esa capa de routing produciría exactamente este patrón: timeouts en todos los endpoints ACS de los emisores simultáneamente, sin que ningún emisor individual sea uniquamente peor que los demás.

---

## 2.5 Hipótesis de Causa Raíz

**Hipótesis: La actualización del Directory Server v2.2 de EBANX del 15 de noviembre introdujo una regresión en el routing de autenticación 3DS para emisores brasileños, causando timeouts sistémicos de solicitudes ACS (código 91) en todas las redes de tarjetas y bancos emisores.**

**Nivel de confianza: Alto.**

Cadena de evidencia de soporte:
1. **Correlación temporal:** EBANX desplegó el DS v2.2 el 15 de nov. Las tasas estaban estables el 10 de nov (82.8%) y el 16 de nov (81.9%). Colapso el 21 de nov.
2. **Modo de fallo:** El código 91 subió 829% en EBANX. dLocal no exhibe spikes de código 91.
3. **Uniformidad del impacto:** Los cinco principales emisores y las tres redes de tarjetas cayeron 28-30pp en paralelo.
4. **Auto-reconocimiento del proveedor:** Soporte de EBANX confirmó "latencia intermitente en 3DS challenges de emisores brasileños desde el 20 de nov" (en conversación paralela con otro merchant de Yuno — uso interno únicamente).

**Incertidumbre remanente:**
- El lag de 5 días entre el upgrade (15 nov) y el colapso (21 nov) no está completamente explicado. Posibles mecanismos: rollout gradual, saturación del connection pool, o la ventana de mantenimiento de BdB como trigger de estrés.
- Amex no está cubierta por el scope del DS v2.2 (solo Visa/MC), pero también cayó ~26pp — plantea la pregunta de si el problema se extiende más allá de 3DS hacia la API de autorización general de EBANX.
- Un desglose por monto de transacción (por encima/por debajo del umbral de $50 USD para 3DS) confirmaría si las transacciones mensuales sin 3DS ($9.99) también están afectadas.

---

## 2.6 Eventos que NO Son la Causa Raíz

**Actualización del checkout frontend de StreamVibe (31 oct):** Desplegada 3+ semanas antes de la caída. Tasas estables al 82.8% hasta el 10 de nov. Los códigos de rechazo duro 14 y 54 aumentaron solo 2.9% y 2.8% — sin errores de envío de datos de tarjeta. No es un factor contribuyente.

**Lanzamiento de PIX en Brasil (8 nov):** Opera sobre un rail de pago separado. PIX AR es 93.0%, estable. Sin interacción con la infraestructura de autorización de tarjetas. No relacionado.

**Mantenimiento programado del Banco do Brasil (20-21 nov):** Marcado como completado la noche del 21 nov; la tasa continuó empeorando al 54.6% hasta el 24 nov. Los BINs de BdB cayeron exactamente al mismo nivel que Itaú, Bradesco, Santander y Nubank. dLocal mantiene 82.7% en BINs de BdB durante el mismo período. No es la causa raíz — puede haber actuado como el trigger de estrés que expuso la regresión de latencia preexistente de EBANX.

---

## 2.7 Cuantificación del Impacto — Tarjetas Brasil

| | Período Anterior | Período Actual | Delta |
|-|-----------------|----------------|-------|
| Aprobaciones de tarjeta EBANX | 11,169 (13,440 × 83.1%) | 7,338 (13,440 × 54.6%) | **-3,831** |
| Aprobaciones de tarjeta dLocal | 4,666 (5,760 × 81.0%) | 4,764 (5,760 × 82.7%) | +98 (recuperación parcial) |
| **Aprobaciones netas de tarjeta Brasil perdidas** | **15,840** | **12,096** | **-3,744/mes** |

**Ingresos en riesgo:**

| Escenario | Impacto Mensual |
|-----------|----------------|
| Conservador (100% plan mensual, promedio $9.99) | **~$37,402/mes** |
| Mixto (70% mensual / 30% anual, $89.99) | **~$127,000/mes** |

Estas cifras representan únicamente intentos de cobro fallidos. El churn de suscriptores posterior que no reintentaron no está cuantificado pero típicamente multiplica entre 1.5× y 2.5× el impacto inmediato en ingresos para negocios de suscripción.

---

## 3. Argentina: Evaluación Técnica

**Diagnóstico: Rendimiento en baseline de mercado. No es un incidente técnico.**

| Métrica | StreamVibe | Benchmark Suscripción LatAm |
|---------|-----------|------------------------------|
| Tarjetas vía Mercado Pago | 51.0% | 52-58% |
| Billetera Mercado Pago | 66.1% | 64-70% |

La tasa de autorización de tarjetas Argentina está dentro del rango benchmark establecido. Los códigos de rechazo dominantes son 51 (fondos insuficientes) y 62 (tarjeta restringida) — respuestas del emisor al entorno macroeconómico de Argentina (>100% de inflación anual, controles de capital). No son reintentables, no se resuelven con cambios de routing.

Las alternativas (Stripe, dLocal) rinden al **48-50%** en tarjetas Argentina — materialmente peor que el 51% de Mercado Pago, ya que Mercado Pago tiene mayor profundidad de relaciones con los emisores locales. Deshabilitar Mercado Pago como adquirente de tarjeta reduciría la tasa de autorización Argentina en un estimado de **3-5pp**.

La billetera de Mercado Pago (66.1%, por encima del benchmark) es el instrumento de mayor conversión disponible en Argentina. La oportunidad estratégica está en migrar la base de suscriptores hacia la billetera.

---

## 4. Confirmado vs. Necesita Confirmación

**Confirmado por datos:**
- La crisis en Brasil es solo de tarjetas. PIX y Boleto no están afectados.
- La crisis es solo de EBANX. dLocal operando al 82.7%.
- El código 91 (timeout del emisor) aumentó 829% exclusivamente en EBANX.
- Todos los emisores y redes uniformemente afectados → fallo en la capa DS de EBANX.
- EBANX desplegó la actualización DS v2.2 cinco días antes del colapso.
- Soporte de EBANX reconoció latencia en 3DS challenges desde el 20 de nov.
- Argentina está en benchmark de mercado. Ningún adquirente alternativo rinde mejor.

**Requiere confirmación:**
- Desglose por monto de transacción (por encima/por debajo del umbral de $50 USD para 3DS): ¿las transacciones mensuales de $9.99 también están cayendo? Si es así, el problema se extiende más allá de 3DS hacia el routing de autorización general.
- Análisis de causa raíz y timeline de fix/rollback de EBANX.
- Si Amex fluye por el DS v2.2 actualizado o por un path separado.
- Cronograma exacto del rollout del DS v2.2 (gradual vs. cutover completo el 15 de nov).
