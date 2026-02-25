# Resumen Ejecutivo — Crisis de Autorización LatAm

**Merchant:** StreamVibe
**Fecha:** 25 de noviembre, 2024
**Preparado por:** Equipo TAM Yuno
**Para:** Claudia Mendez · Rafael Santos

---

## Situación

En los últimos 7 días, la tasa de aprobación de pagos con tarjeta en Brasil cayó de **82.5% a 63.0%** — una pérdida de 19.5 puntos porcentuales. El impacto estimado en ingresos es de **$37,000–$130,000 por mes** en función del mix de planes de suscripción, sin contar el churn de suscriptores que no reintentaron el pago.

Los demás métodos de pago en Brasil (PIX y Boleto) y los demás países (México, Colombia, Chile) operan con normalidad.

---

## Causa

El problema está localizado en un único procesador de pagos: **EBANX**. Nuestro análisis confirma con alta confianza que una actualización de infraestructura realizada por EBANX el 15 de noviembre introdujo una regresión en su capa de autenticación, generando fallas sistémicas de timeout que afectan a todos los bancos emisores y redes de tarjeta en Brasil simultáneamente.

Tres datos lo confirman:

1. **El procesador de respaldo dLocal aprueba el 82.7%** de las mismas tarjetas, en los mismos bancos, en el mismo período. Si el problema fuera del mercado o de los emisores, dLocal también caería. No lo hace.
2. **El error de timeout (código 91) subió 829%** — de 412 instancias a 3,825 en EBANX exclusivamente. Todos los demás tipos de rechazo se mantienen en niveles normales.
3. **Los cinco principales bancos emisores cayeron simultáneamente al mismo nivel** (~54-55%), lo cual solo es explicable por un fallo en el intermediario compartido entre EBANX y los bancos — no en los bancos individuales.

---

## Mitigación en Curso

**Yuno ya activó dLocal como procesador primario en Brasil.** Este cambio no requirió ninguna modificación técnica del lado de StreamVibe y está activo desde el 25 de noviembre. Las transacciones de tarjeta se están procesando con normalidad a través de dLocal mientras trabajamos con EBANX en la resolución definitiva.

---

## Próximas Acciones

| Acción | Responsable | Plazo |
|--------|-------------|-------|
| Escalación formal a EBANX con evidencia del problema | Yuno | 24 horas |
| Análisis de causa raíz y ETA del fix de EBANX | EBANX | 48–72 horas |
| Definir criterios de reactivación de EBANX como primario | Yuno + StreamVibe | 48 horas |
| Reporte semanal de performance en Brasil | Yuno | Cada lunes |

EBANX no será reactivado como procesador primario hasta contar con datos que confirmen la resolución — no solo con su confirmación verbal.

---

## Argentina

La tasa de aprobación en Argentina (54%) es un tema independiente del incidente de Brasil. Está dentro del rango de referencia del mercado local (52-58%) para negocios de suscripción y responde a condiciones macroeconómicas — no a una falla técnica. Ningún procesador alternativo disponible en Argentina rinde mejor. Tenemos recomendaciones específicas para mejorar la performance en Argentina que compartiremos por separado.

---

## Resumen en Una Línea

> El problema está identificado, la mitigación está activa, y StreamVibe no necesita hacer nada técnico. La tasa de aprobación en Brasil se estabilizó en ~82.7% con dLocal mientras cerramos el loop con EBANX.
