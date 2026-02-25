# Email — Entrega de RCA al Merchant

**Para:** claudia.mendez@streamvibe.com
**CC:** rafael.santos@streamvibe.com
**Asunto:** [StreamVibe] Análisis de causa raíz — caída en tasa de aprobación Brasil
**Fecha:** 25 de noviembre, 2024

---

Claudia, Rafael,

Les comparto el análisis formal de lo que está ocurriendo con las tasas de aprobación en Brasil.

**Lo más importante en tres puntos:**

1. **El problema está identificado y localizado.** La caída en tarjetas (de 82.5% a 63.0%) está fuertemente correlacionada con una actualización de infraestructura realizada por EBANX el 15 de noviembre. Nuestro análisis apunta a una regresión en su capa de autenticación como causa probable — la cual estamos confirmando formalmente con ellos. El resto del ecosistema — PIX, Boleto, y el procesador dLocal — opera con normalidad.

2. **Ya hay mitigación activa.** Desde Yuno activamos dLocal como procesador primario en Brasil. Esto no requirió ningún cambio técnico del lado de StreamVibe. Las transacciones están siendo procesadas con normalidad por dLocal (82.7% de tasa de aprobación) mientras trabajamos con EBANX en la resolución definitiva.

3. **Argentina es un tema separado.** La tasa del 54% en Argentina no es un fallo técnico — está dentro del rango de referencia del mercado local y tiene causas macroeconómicas. Deshabilitar Mercado Pago empeoraría ese número. Lo explico en detalle en el reporte adjunto.

---

El documento adjunto cubre en detalle: cronología del incidente, evidencia que confirma la causa raíz, lo que descartamos, el impacto estimado en ingresos, y las acciones en curso con responsables y plazos.

Estaremos en contacto para coordinar la llamada de seguimiento. Cualquier pregunta antes de eso, respondo de inmediato.

Saludos,
[Nombre TAM]
Technical Account Manager — Yuno
[email] | [teléfono]

---
> 📎 Adjunto: `informe-causa-raiz-streamvibe-brasil-nov2024.pdf`
