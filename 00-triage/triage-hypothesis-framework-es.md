# StreamVibe Crisis de Autorización LatAm — Framework de Triage e Hipótesis TAM

---

## 1. Resumen Ejecutivo (5-7 bullets)

- **La crisis de autorización en Brasil es exclusiva de EBANX**: dLocal (fallback) está rindiendo al 82.7% — prácticamente idéntico al período anterior. Esto descarta de inmediato problemas estructurales del mercado o de los bancos emisores en Brasil.
- **El dato clave es el código de rechazo 91 (timeout del emisor)**: Subió +829% en EBANX, de 412 instancias a 3,825, representando ahora el 62.8% de todos los rechazos de EBANX. No es una distribución normal — es un modo de fallo sistémico introducido en un momento específico.
- **La actualización del Directory Server v2.2 de EBANX del 15 de noviembre es el principal sospechoso**: Las tasas de aprobación estaban estables al 82.8% el 10 de nov y al 81.9% el 16 de nov, luego colapsaron al 68.2% el 21 de nov — cinco días después de que el upgrade entrara en producción. El propio soporte de EBANX reconoció "latencia intermitente en los 3DS challenges de emisores brasileños" desde el 20 de nov.
- **El impacto es uniforme en TODOS los emisores y TODAS las redes** (~54-56% para Visa, MC, Amex; ~54-55% para Banco do Brasil, Itaú, Bradesco, Santander, Nubank): este patrón es diagnóstico de un fallo *dentro* de la capa de routing 3DS de EBANX, no en el ACS endpoint de ningún emisor individual.
- **La dilución de la tasa global en Brasil (81.2% → 68.0%) está parcialmente enmascarada por PIX**: PIX tiene 93% de tasa de aprobación en 6,850 intentos — está sano y no contribuye al problema. Aislando tarjetas se ve la verdadera severidad: 82.5% → 63.0% (-19.5pp).
- **Argentina NO es una falla técnica**: El 51% de tasa de aprobación de tarjetas de StreamVibe vía Mercado Pago está dentro del benchmark LatAm de suscripción (52-58%) para tarjetas en Argentina. El mix de rechazos (códigos 51 + 62) refleja condiciones macroeconómicas — hiperinflación + controles de capital — no un problema de routing o integración. Deshabilitar Mercado Pago empeoraría los resultados (Stripe/dLocal promedian 48-50% allí).
- **El fallback automático de Yuno a dLocal es lo único que evita una crisis de ingresos total**: dLocal absorbió ~30% del volumen de tarjetas Brasil al 82.7% AR. Sin esto, la tasa de tarjetas Brasil de StreamVibe estaría cerca del 54.6% sin mitigación alguna.

---

## 2. Problema Principal vs. Problemas Secundarios

### Problema Principal (Urgente, ~$38K–$130K/mes en riesgo de ingresos)
**Regresión de infraestructura 3DS2 de EBANX en Brasil**
- Causa raíz: La actualización del Directory Server v2.2 del 15 de nov degradó el routing de autenticación 3DS, causando respuestas sistémicas de timeout (código 91) desde los ACS endpoints de los emisores brasileños.
- El problema está activo y empeorando (21 nov: 68.2% → 24 nov: 54.6%), y es totalmente atribuible a EBANX.
- **Es resoluble** — ya sea porque EBANX resuelve la regresión del DS o porque Yuno re-enruta temporalmente el primario de tarjetas Brasil a dLocal.

### Problemas Secundarios (Crónicos, estructurales, menor prioridad)
1. **Estancamiento de tasa de aprobación en Argentina (54%, 8 semanas)**: Problema de baseline de mercado. Ningún ajuste técnico va a mover significativamente la aguja. La optimización pasa por tácticas de retención y migración a wallet, no por cambios de routing.
2. **Riesgo de concentración de volumen de tarjetas en Brasil**: Enrutar el 70% de las tarjetas de Brasil a un único adquirente (EBANX) sin cap automático creó un único punto de fallo. Debe implementarse un split de routing proactivo post-incidente.
3. **Calibración del umbral 3DS**: La regla actual ">$50 USD" para 3DS en Brasil puede estar desalineada con los puntos de precio de suscripción de StreamVibe ($9.99 mensual está por debajo del umbral; $89.99 anual activa 3DS), creando fricción selectiva en las renovaciones de mayor valor.

---

## 3. Caída de Tasa de Autorización Brasil — Hipótesis (Rankeadas)

### Hipótesis 1: Regresión del Directory Server v2.2 de EBANX [ALTA CONFIANZA — CAUSA PRINCIPAL]

**Evidencia:**
- 15 nov: EBANX desplegó la actualización DS v2.2 para Visa/Mastercard Brasil
- 21 nov: Código 91 sube de ~3% de rechazos a 61% de rechazos
- Código 91 = timeout del ACS del emisor durante el challenge 3DS — el DS es el intermediario entre EBANX y el ACS del emisor
- La caída de tasa de aprobación es perfectamente uniforme en LOS 5 principales emisores (Banco do Brasil, Itaú, Bradesco, Santander, Nubank todos en ~54-55%) — solo explicable por un fallo en la capa de infraestructura compartida (el DS de EBANX), no en emisores individuales
- La caída es uniforme en LAS 3 redes (Visa 54%, MC 55%, Amex 56%) — el DS v2.2 era específicamente para Visa/Mastercard; que Amex también esté afectada sugiere un problema más amplio de routing del DS o de latencia
- dLocal no está afectado (82.7%) — mismos emisores brasileños, mismos BINs de tarjeta, infraestructura 3DS diferente
- Soporte de EBANX (en una conversación separada con otro merchant) confirmó "latencia intermitente en 3DS challenges de emisores brasileños desde el 20 de nov"
- La página de estado de EBANX no muestra incidente activo (típico — los problemas de latencia suelen no declararse hasta ser confirmados)

**Mecanismo:** La actualización DS v2.2 probablemente cambió el routing de endpoints, los parámetros de timeout, o el handshake de protocolo con los servidores ACS de los emisores brasileños. Cuando se inicia el challenge 3DS, la solicitud al ACS hace timeout → el emisor devuelve código 91 → la transacción es rechazada. El lag de 5 días entre el upgrade (15 nov) y el colapso (21 nov) podría explicarse por: rollout gradual, agotamiento del connection pool del ACS del emisor, o la ventana de mantenimiento del Banco do Brasil el 20-21 nov exponiendo un problema de latencia preexistente en la capa 3DS de EBANX.

**Debilidad:** No tenemos un desglose de rechazos por monto de transacción (por encima/por debajo del umbral de $50). Si 3DS solo aplica a transacciones anuales de $89.99, la tasa de caída del ~45% en todas las tarjetas EBANX implicaría que la mayoría del volumen es anual — plausible pero no confirmado.

---

### Hipótesis 2: Problema de Latencia Global de Infraestructura EBANX (No Específico de 3DS) [CONFIANZA MEDIA — CAUSA SECUNDARIA O AMPLIFICADORA]

**Evidencia:**
- EBANX desplegó una "actualización de infraestructura" el 15 de nov de forma más amplia, no solo en 3DS
- Si la latencia está en la API general de autorización de EBANX (no solo en 3DS), explicaría por qué incluso las transacciones sin 3DS (<$50) están haciendo timeout
- El código 91 puede significar timeout general de comunicación con el emisor, no solo timeout específico del ACS 3DS

**Por qué podría explicar el panorama completo:** Los suscriptores mensuales ($9.99 < umbral de $50) no activarían 3DS pero podrían seguir rechazándose al 54%. Si solo fuera la hipótesis 1, esperaríamos que solo los suscriptores anuales cayeran marcadamente, con los mensuales cerca de lo normal. La tasa uniforme del 54% sugiere que (a) la mayoría del volumen es anual, o (b) el problema de EBANX va más allá de 3DS.

**Debilidad:** Más especulativo — sin evidencia directa de problemas generales de infraestructura de EBANX más allá de la actualización 3DS.

---

### Hipótesis 3: Impacto Residual del Mantenimiento del Banco do Brasil [BAJA CONFIANZA — NO ES LA CAUSA RAÍZ]

**Evidencia:**
- BdB anunció mantenimiento para el 20-21 de nov, que coincide con el colapso del 21 nov
- El mantenimiento fue marcado como completado la noche del 21 nov

**Por qué NO es la causa raíz:**
- BdB (BIN 4011xx) cayó al 55% — idéntico a Itaú (54%), Bradesco (54%), Santander (54.1%) y Nubank (55%). Si el mantenimiento de BdB fuera el problema, BdB mostraría una tasa uniquamente peor.
- dLocal, enrutando al MISMO endpoint del emisor BdB, mantiene 82.7% de tasa de aprobación — descartando definitivamente la infraestructura de BdB como causa

---

### Hipótesis 4: Actualización del Frontend de Checkout del 31 de Oct Rompiendo Tokenización/Envío de Tarjeta [MUY BAJA CONFIANZA]

**Evidencia (en contra):**
- Desplegado el 31 de oct, las tasas de aprobación se mantuvieron estables durante más de 3 semanas después (EBANX al 82.8% el 10 de nov)
- Los rechazos duros (número de tarjeta inválido = código 14, vencida = código 54) son esencialmente planos (+2.9%, +2.8%)
- Si los datos de la tarjeta estuvieran malformados, veríamos spikes en códigos de rechazo duro, no en código 91

**Veredicto:** No es la causa.

---

## 4. Evaluación de Argentina: Problema Técnico vs. Baseline de Mercado

**Veredicto: Baseline de mercado. No existe ningún ajuste técnico accionable en el corto o mediano plazo.**

| Señal | Observación | Interpretación |
|-------|-------------|----------------|
| Tarjetas vía Mercado Pago | 51.0% AR | Dentro del benchmark 52-58% → EN EL PISO |
| Billetera Mercado Pago | 66.1% AR | Dentro del benchmark 64-70% → SALUDABLE |
| Códigos de rechazo principales | 51 (fondos insuficientes) + 62 (tarjeta restringida) | Macroeconómico, no de integración |
| Tendencia (8 semanas) | -1.1pp | Plano, no en deterioro activo |
| Alternativas (Stripe, dLocal) | 48-50% en Argentina | PEOR que Mercado Pago actual |

**La propuesta del equipo de finanzas de deshabilitar Mercado Pago debe rechazarse de forma clara y definitiva.**

---

## 5. Falsas Pistas y Eventos No Causales

| Evento | Fecha | Por Qué Es una Falsa Pista |
|--------|-------|---------------------------|
| **EBANX pasa a ser primario en Brasil** | 2 nov | Ocurrió 4 semanas antes del colapso. Las tasas estuvieron estables al ~83% hasta el 16 de nov. El cambio de routing no es la causa — es lo que hace a EBANX la superficie principal de este fallo. |
| **Actualización del UI del checkout de StreamVibe** | 31 oct | Brecha de 3+ semanas antes de cualquier cambio de tasa. Los códigos de rechazo duro (14, 54) son planos. |
| **Habilitación de PIX en Brasil** | 8 nov | PIX está al 93% AR, estable. Sin efecto en el routing de tarjetas o la autenticación del emisor. |
| **Ventana de mantenimiento del Banco do Brasil** | 20-21 nov | Marcado como completado. Todos los demás emisores cayeron al mismo nivel que BdB. dLocal muestra 82.7% en BINs de BdB. |
| **Argentina "terrible durante semanas"** | Continuo | En el piso del mercado durante 8 semanas, sin deterioro activo. No es una crisis — es un techo. |

---

## 6. Métricas y Tablas Clave a Usar como Evidencia en el RCA Final

1. **Tarjetas Brasil por Adquirente** → EBANX 54.6% vs. dLocal 82.7% — descarta todas las explicaciones de mercado/emisor
2. **Distribución de Códigos de Error (EBANX)** → Código 91: 412 → 3,825 (+829%) — la evidencia definitiva
3. **EBANX por BIN de Emisor** → los 5 principales emisores en ~54-55% simultáneamente — apunta a la capa compartida del DS
4. **EBANX por Red de Tarjeta** → Visa, MC, Amex igualmente afectados — Amex no está en el scope del DS v2.2 = problema más amplio
5. **Cronología del Incidente** → Upgrade 15 nov → colapso 21 nov (lag de 5 días)
6. **Brasil por Método de Pago** → PIX/Boleto sanos — crisis solo en tarjetas
7. **Argentina por Método de Pago** → wallet (66%) sano, tarjetas en benchmark

### Cuantificación del Impacto en Ingresos
- Aprobaciones EBANX perdidas: 13,440 × (83.1% - 54.6%) = **~3,831/mes**
- Pérdida neta (después de recuperación dLocal): **~3,744 aprobaciones/mes**
- Impacto conservador en ingresos: **~$37,400/mes** (ticket promedio plan mensual $9.99)
- Impacto mixto (70% mensual / 30% anual): **~$127,000/mes**
