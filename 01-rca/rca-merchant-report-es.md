# Informe de Causa Raíz — Caída en Tasa de Aprobación Brasil

**Merchant:** StreamVibe
**Fecha:** 25 de noviembre, 2024
**Preparado por:** Equipo Técnico Yuno
**Estado del incidente:** Activo — en proceso de resolución
**Audiencia:** Claudia Mendez (Head of Payments) · Rafael Santos (CTO)

---

## Resumen Ejecutivo

Durante la última semana, StreamVibe experimentó una caída significativa en la tasa de aprobación de pagos con tarjeta en Brasil: pasó del **81.2% al 68.0%** a nivel general, y del **82.5% al 63.0%** específicamente en tarjetas. El problema está localizado en un único procesador de pagos — EBANX — y está siendo causado por un error técnico en su infraestructura de autenticación, introducido el 15 de noviembre en una actualización de su sistema.

Los métodos de pago alternativos (PIX y Boleto) funcionan con normalidad. El procesador de respaldo, dLocal, está manteniendo una tasa de aprobación del 82.7% para las mismas tarjetas afectadas, lo que confirma que el problema es exclusivo de EBANX y no refleja el comportamiento del mercado ni de los bancos emisores en Brasil. La situación en Argentina es independiente y tiene una causa diferente, que se detalla al final de este documento.

---

## ¿Qué está pasando?

Cuando un cliente intenta pagar con tarjeta en Brasil, la transacción pasa por un proceso de verificación de identidad llamado **autenticación 3DS** (es el paso donde el banco le pide al cliente confirmar la compra por SMS, app o contraseña). EBANX actúa como intermediario entre StreamVibe y los bancos emisores para gestionar este proceso.

El 15 de noviembre, EBANX realizó una actualización técnica en el servidor que gestiona estas autenticaciones. A partir del 20 de noviembre, ese servidor comenzó a generar tiempos de espera excesivos al comunicarse con los bancos emisores brasileños. Como consecuencia, los bancos interpretan que no hay respuesta en tiempo y rechazan la transacción con el código de error **91 ("issuer timeout" — banco no disponible)**.

Este error pasó de representar el **3% de los rechazos** a ser la causa del **62.8% de todos los rechazos en EBANX**, con un incremento del **829%** en volumen absoluto. Todos los demás tipos de rechazo (fondos insuficientes, tarjeta expirada, etc.) se mantienen en niveles normales.

---

## Cronología del Incidente

| Fecha | Evento |
|-------|--------|
| 2 nov | Yuno actualiza el enrutamiento de Brasil: EBANX pasa a ser el procesador primario (por costo operativo) |
| 10 nov | Tasa de aprobación via EBANX: **82.8%** — operación normal |
| 15 nov | EBANX implementa actualización en su infraestructura de autenticación 3DS |
| 16 nov | Tasa de aprobación via EBANX: **81.9%** — todavía estable |
| 20-21 nov | Mantenimiento programado del Banco do Brasil (3DS) — marcado como completado el 21 nov |
| 21 nov | Tasa de aprobación via EBANX cae a **68.2%**. El error de timeout sube de ~3% a 61% de los rechazos |
| 24 nov | Tasa de aprobación via EBANX: **54.6%** — continúa deteriorándose |
| 25 nov | Yuno ha activado mitigación via dLocal. Escalación activa con EBANX |

---

## Evidencia que Confirma la Causa Raíz

Hay tres datos clave que apuntan a EBANX como causa exclusiva del problema:

### 1. dLocal procesa las mismas tarjetas al 82.7%

El procesador de respaldo dLocal está aprobando tarjetas de exactamente los mismos bancos brasileños (Banco do Brasil, Itaú, Bradesco, Santander, Nubank) con una tasa del **82.7%** — prácticamente idéntica al período anterior. Si el problema fuera de los bancos o del mercado brasileño, dLocal también estaría afectado. No lo está.

| Procesador | Tasa de aprobación actual | Variación vs período anterior |
|-----------|--------------------------|-------------------------------|
| EBANX (primario) | 54.6% | **-28.5 puntos porcentuales** |
| dLocal (respaldo) | 82.7% | +1.7 puntos porcentuales |

### 2. Todos los bancos y redes cayeron al mismo tiempo y al mismo nivel

Los cinco principales bancos emisores en Brasil (Banco do Brasil, Itaú, Bradesco, Santander y Nubank) cayeron entre 28 y 30 puntos porcentuales de manera simultánea, convergiendo todos en una tasa de ~54-55%. Visa, Mastercard y Amex cayeron de manera casi idéntica. Esta uniformidad matemática no puede ser resultado de cambios en los bancos individuales. Solo puede explicarse por un fallo en el sistema que media entre EBANX y todos los bancos: su servidor de autenticación.

### 3. El único error que explotó fue el de timeout

| Código de error | Descripción | Antes | Ahora | Variación |
|----------------|-------------|-------|-------|-----------|
| **91** | **Banco no responde (timeout)** | **412** | **3,825** | **+829%** |
| 51 | Fondos insuficientes | 1,089 | 1,104 | +1.4% |
| 05 | Rechazo genérico | 502 | 518 | +3.2% |
| 14 | Número de tarjeta inválido | 238 | 245 | +2.9% |
| 54 | Tarjeta expirada | 179 | 184 | +2.8% |

Todos los demás rechazos crecieron solo 1-3%, consistente con el crecimiento normal del volumen. El código 91 creció 829%. Es una falla de un único tipo, con un origen identificable.

---

## ¿Qué NO causó el problema?

Con los datos disponibles, podemos descartar con certeza las siguientes causas:

**La actualización del checkout (31 oct):** Implementada casi cuatro semanas antes de la caída. Las tasas de aprobación se mantuvieron estables hasta el 16 de noviembre. Los rechazos por error en los datos de tarjeta (códigos 14 y 54) no muestran ningún cambio anormal.

**El lanzamiento de PIX (8 nov):** PIX opera sobre una red de pagos completamente separada. Su tasa de aprobación es del **93.0%**, estable y sin cambios. No tiene ninguna interacción con el sistema de autorización de tarjetas.

**El mantenimiento del Banco do Brasil (20-21 nov):** Esta ventana de mantenimiento coincidió con el inicio de la caída, pero no es la causa raíz. El mantenimiento fue marcado como completado y la tasa siguió cayendo días después. Además, todos los demás bancos sin mantenimiento cayeron exactamente igual que el Banco do Brasil, y dLocal mantiene 82.7% de aprobación en tarjetas del mismo banco durante el mismo período. El mantenimiento puede haber actuado como detonante que expuso un problema preexistente en EBANX, pero no es la causa.

---

## Impacto en Ingresos

| | Período anterior | Período actual | Diferencia |
|-|-----------------|----------------|------------|
| Aprobaciones via EBANX | ~11,169/mes | ~7,338/mes | **-3,831 aprobaciones** |
| Recuperación via dLocal | — | +98 adicionales | Mitigación parcial |
| **Pérdida neta de aprobaciones** | | | **~3,744/mes** |

**Estimación de impacto en ingresos:**
- Escenario conservador (suscripciones mensuales a $9.99): **~$37,400/mes en riesgo**
- Escenario mixto (mensuales + anuales): **hasta ~$127,000/mes en riesgo**

Estas cifras reflejan únicamente los pagos que fallaron y no fueron recuperados. No incluyen el impacto de cancelaciones de suscripciones por parte de clientes que enfrentaron un rechazo y no reintentaron — lo cual históricamente multiplica el impacto entre 1.5× y 2.5× en negocios de suscripción.

---

## Argentina: Una Situación Distinta

La tasa de aprobación en Argentina (54%) lleva 8 semanas sin mejora significativa y no está relacionada con el incidente de Brasil. Su causa es estructural y macroeconómica.

**Diagnóstico:** Argentina tiene inflación superior al 100% anual y controles de capital estrictos que limitan el uso internacional de tarjetas. Los principales tipos de rechazo son fondos insuficientes y tarjetas con límite restringido por el banco emisor — ambos reflejan condiciones económicas del país, no fallas técnicas de la integración.

**La tasa del 51-54% está dentro del rango de referencia del mercado** (52-58%) para negocios de suscripción en Argentina. No hay procesador alternativo que mejore este número: Stripe y dLocal en Argentina promedian **48-50%** de aprobación en tarjetas, por debajo de lo que Mercado Pago logra actualmente gracias a sus relaciones con los bancos locales.

**Recomendación:** No se recomienda deshabilitar Mercado Pago como procesador de tarjetas. Hacerlo reduciría la tasa de aprobación entre 3 y 5 puntos porcentuales adicionales. El camino de optimización para Argentina pasa por una estrategia diferente (ver sección de recomendaciones proactivas).

---

## Acciones en Curso

| Acción | Responsable | Estado | Plazo |
|--------|-------------|--------|-------|
| Activar dLocal como procesador primario en Brasil (temporalmente) | Yuno | ✅ En proceso | Inmediato |
| Escalar incidente a EBANX con evidencia completa del código 91 y la actualización del 15 nov | Yuno | 🔄 En curso | 24 horas |
| Solicitar análisis de causa raíz formal y plan de rollback a EBANX | Yuno + EBANX | 🕐 Pendiente respuesta | 48 horas |
| Confirmar si transacciones bajo $50 USD (plan mensual $9.99) también están afectadas | Yuno + EBANX | 🕐 Pendiente datos | 48 horas |
| Revisar estrategia de enrutamiento en Brasil post-resolución | Yuno | 🕐 Pendiente | Próxima semana |

---

## Qué Está Confirmado y Qué Aún Necesitamos Validar

**Confirmado:**
- El problema es exclusivo de tarjetas. PIX y Boleto operan con normalidad.
- El problema es exclusivo de EBANX. dLocal funciona al 82.7% en las mismas condiciones.
- El error dominante (código 91 — timeout) creció 829% en EBANX exclusivamente.
- La uniformidad del impacto en todos los bancos y redes apunta al servidor de autenticación de EBANX como origen.
- El incidente de Argentina es independiente y no es una falla técnica.

**Pendiente de confirmación:**
- Si las suscripciones mensuales ($9.99, por debajo del umbral de autenticación 3DS) también están siendo rechazadas — esto determinaría si el problema está confinado al flujo de autenticación o es más amplio dentro de la infraestructura de EBANX.
- Cronograma de resolución por parte de EBANX.
- Causa exacta del lag de 5 días entre la actualización (15 nov) y la caída (21 nov).

---

*Próxima actualización: 27 de noviembre, 2024 — o antes si hay novedades de EBANX.*
*Contacto directo para este incidente: [TAM Name] — [email]*
