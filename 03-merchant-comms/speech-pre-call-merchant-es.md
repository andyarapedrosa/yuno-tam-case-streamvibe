# Guía de Llamada — Pre-Call con StreamVibe
**Fecha:** 25 de noviembre, 2024
**Participantes:** Claudia Mendez (Head of Payments), Rafael Santos (CTO)
**Conducida por:** TAM Senior, Yuno
**Duración estimada:** 30–40 minutos

---

## Antes de la llamada — Checklist

- [ ] Confirmar que el reporte de RCA fue enviado por escrito antes de la llamada
- [ ] Tener a la mano los datos: auth rate actual con dLocal (82.7%), histórico con EBANX, curva de decline code 91
- [ ] Preparar pantalla compartida con dashboard de Yuno si es necesario
- [ ] Confirmar que dLocal ya está activo como procesador primario en Brasil

---

## 1. APERTURA — Tono, agenda y reconocimiento de urgencia

**Objetivo:** Establecer confianza desde el primer minuto. Claudia va a estar impaciente. Rafael va a estar escaneando si esto le va a generar trabajo extra. Hay que ir al grano rápido.

**Tono:** Directo, calmado, dueño de la situación. No defensivo.

---

**Frases sugeridas para abrir:**

> "Gracias por el tiempo, Claudia, Rafael. Voy directo al grano porque sé que tienen poco tiempo y esto merece atención real."

> "Les mandamos el reporte escrito en paralelo a esta llamada — el objetivo de esta sesión no es leerlo juntos, sino que salgan con claridad total sobre qué pasó, qué ya está resuelto, y qué sigue pendiente."

**Agenda explícita (decirla en voz alta):**

> "En los próximos 30 minutos vamos a cubrir cuatro cosas: uno, qué causó la caída en Brasil y por qué el problema es puntual; dos, qué ya activamos desde Yuno sin que tuvieran que hacer nada; tres, qué necesitamos de EBANX para cerrar el loop; y cuatro, el tema de Argentina que entiendo está generando ruido con el equipo de finanzas."

---

## 2. DIAGNÓSTICO — Qué pasó y por qué

**Objetivo:** Explicar la causa raíz con claridad. Sin jerga técnica innecesaria. Claudia necesita el "qué" y el "por qué". Rafael puede aguantar más detalle técnico si lo quiere.

**Mensaje central:** El problema es de EBANX, no del mercado brasileño, no de las tarjetas de StreamVibe, no de Yuno.

---

**Narrativa sugerida:**

> "El 15 de noviembre, EBANX hizo una actualización de infraestructura en su capa de autenticación. Esa actualización introdujo una regresión — en términos simples, algo que funcionaba dejó de funcionar correctamente."

> "El código de rechazo que explotó fue el 91 — significa que el banco no recibió respuesta a tiempo. Ese error subió un 829% respecto al período anterior. Eso no es variación normal de mercado. Es una señal técnica muy específica."

> "El número que lo confirma todo: dLocal, que tiene el mismo portafolio de tarjetas y los mismos emisores en Brasil, está autorizando al 82.7% en ese mismo período. Si el problema fuera el mercado, dLocal también caería. No cayó. El problema es EBANX."

**Datos clave a tener visibles:**

| Métrica | Antes | Ahora |
|---|---|---|
| Auth rate tarjetas Brasil | 82.5% | 63.0% |
| EBANX (primario) | 83.1% | 54.6% |
| dLocal (mismo portafolio) | 81.0% | 82.7% |
| Decline code 91 en EBANX | baseline | +829% |

**Para Rafael si profundiza técnicamente:**

> "La regresión está en el servidor de directorio de EBANX — el intermediario entre ellos y los sistemas de autenticación de cada banco. Cuando ese servidor falla en el handshake con el banco, la transacción cae con timeout. Eso explica por qué todos los bancos cayeron al mismo nivel simultáneamente — no es un problema de cada banco, es un problema del intermediario."

---

## 3. LO QUE YA HICIMOS — Mitigación activa

**Objetivo:** Demostrar que Yuno ya actuó. No vinimos a explicar un problema — vinimos a confirmar que ya lo mitigamos operativamente mientras cerramos la solución de fondo.

**Mensaje central:** No tienen que hacer nada técnico. Ya está mitigado.

---

**Frases sugeridas:**

> "Desde Yuno ya activamos dLocal como procesador primario en Brasil. Esto no requirió ningún cambio de integración del lado de StreamVibe — lo manejamos a nivel de enrutamiento dentro de la plataforma."

> "Claudia, la respuesta corta a '¿qué están haciendo?' es: ya lo estamos haciendo. Las transacciones están siendo procesadas por dLocal mientras EBANX resuelve el problema."

> "Rafael, esto no te genera ningún ticket nuevo ni trabajo de integración. El cambio fue en la capa de orquestación de Yuno — eso es exactamente para lo que están usando la plataforma."

**Para cerrar este punto con fuerza:**

> "La ventaja de tener orquestación es esta: cuando un procesador falla, no dependen de que les escribamos un correo — podemos accionar directamente. Eso es lo que hicimos."

---

## 4. LO QUE NECESITAMOS DE EBANX — Pendientes y timeline

**Objetivo:** Ser transparentes sobre lo que no está en nuestras manos. No prometer fechas que no podemos garantizar.

**Mensaje central:** La mitigación está activa. La resolución definitiva depende de EBANX, pero estamos encima de ellos.

---

**Frases sugeridas:**

> "Tenemos escalado el caso con el equipo técnico de EBANX. No voy a darles una fecha que no puedo garantizar — lo que sí les digo es que este es un bug documentado con evidencia clara y estamos presionando para resolución en los próximos 7 a 14 días."

> "Mientras no tengamos esa confirmación con datos, EBANX no vuelve a ser primario en Brasil. Punto."

**Preguntas abiertas con EBANX (compartir con el merchant):**
1. ¿Cuándo se detectó internamente la regresión y por qué no hubo notificación proactiva?
2. ¿Cuál es el ETA para el fix en producción?
3. ¿Habrá un post-mortem formal de su parte?
4. ¿Qué controles implementarán para que una actualización de infraestructura no genere este impacto sin alertas?

> "Estas preguntas las tenemos abiertas con EBANX y vamos a compartir las respuestas con ustedes en cuanto las tengamos. No vamos a aceptar un 'ya está resuelto' sin datos que lo demuestren."

---

## 5. ARGENTINA — Neutralizar el pánico

**Objetivo:** Breve, directo y definitivo. Finanzas quiere deshabilitar Mercado Pago. Eso sería un error. Decirlo claramente, sin generar alarma adicional.

**Mensaje central:** Argentina está en baseline de mercado. Deshabilitar Mercado Pago haría más daño.

---

**Frases sugeridas:**

> "Entiendo que finanzas está mirando el 54% en Argentina con preocupación. Quiero ser directo: eso no es un problema técnico. Es el baseline del mercado argentino ahora mismo — inflación, restricciones cambiarias, comportamiento de los bancos locales. No hay un bug que arreglar."

> "Deshabilitar Mercado Pago en Argentina no mejora ese número — lo empeora. Es el procesador con mayor cobertura y mejores tasas de aprobación disponibles en ese mercado. Quitarlo elimina la capacidad de procesar, no resuelve la tasa baja."

**Si presionan con "¿entonces qué hacemos con Argentina?":**

> "Lo que hacemos es monitorear y evaluar si hay tácticas de routing que puedan mover un par de puntos. Pero las expectativas tienen que ser realistas: en el contexto macroeconómico actual de Argentina, un 54% es lo que está viendo buena parte del mercado. No es una anomalía de StreamVibe."

> "Podemos agendar una sesión separada específicamente para Argentina si quieren profundizar — pero mezclarlo con el problema de Brasil haría ruido en esta llamada."

---

## 6. CIERRE — Próximos pasos y cadencia de seguimiento

**Objetivo:** Salir de la llamada con acciones claras, dueños definidos y fechas.

---

**Próximos pasos — decirlos en voz alta:**

| Acción | Responsable | Plazo |
|---|---|---|
| Respuesta de EBANX sobre ETA del fix | Yuno (escalación activa) | 7–10 días hábiles |
| Reporte de performance semanal Brasil | Yuno | Cada lunes |
| Criterios de reactivación de EBANX como primario | Acordar hoy | Esta llamada |
| Sesión opcional sobre Argentina | StreamVibe (si lo decide) | Semana próxima |

**Criterios de reactivación de EBANX — proponer en la llamada:**

> "Propongo que acordemos los criterios para volver a activar EBANX como primario: auth rate sostenido por encima del 80% durante al menos 5 días consecutivos, sin picos de decline code 91, y confirmación escrita del fix por parte de EBANX. ¿Están de acuerdo con esos parámetros?"

**Cierre de tono:**

> "Sé que este tipo de situaciones generan ruido. Lo que quiero que se lleven de esta llamada es esto: el problema está identificado, está mitigado operativamente, y estamos encima del cierre definitivo con EBANX. No los voy a dejar esperando actualizaciones — van a saber lo que sé yo, cuando yo lo sé."

---

## 7. MANEJO DE OBJECIONES

### "¿Por qué pasó esto? ¿No deberían haberlo detectado antes?"

**No decir:** "No es culpa nuestra" / "EBANX no nos avisó"

**Decir:**
> "Es una pregunta válida. EBANX hizo una actualización el 15 de noviembre sin notificación previa al ecosistema de partners. Desde Yuno lo detectamos a través del monitoreo de decline codes cuando el spike de código 91 se hizo estadísticamente significativo."

> "Lo que sí cambió: vamos a establecer alertas más agresivas sobre degradamiento de auth rate por procesador para detectar estos patrones en 24–48 horas, no en días."

---

### "¿Deberíamos sacar a EBANX permanentemente?"

**Decir:**
> "Es una pregunta que vale hacerse, y quiero darles una respuesta honesta. En este momento, sacar a EBANX permanentemente sería una decisión reactiva basada en un incidente puntual. Lo que recomiendo: mantener dLocal como primario durante los próximos 30–60 días, evaluar la performance comparativa con datos, y tomar una decisión de estrategia de procesadores con información, no con el ruido del incidente."

> "No les voy a decir que se queden con EBANX porque sí. Les digo que la decisión correcta se toma con datos."

---

### "¿Qué tiene que hacer ingeniería de StreamVibe?"

**Decir:**
> "Nada. El cambio a dLocal se hizo en la capa de orquestación de Yuno. No hay tickets nuevos, no hay cambios de API, no hay trabajo de integración requerido. Eso es exactamente para lo que están usando la plataforma."

---

### Si mencionan pérdida de ingresos o compensación

**No prometer nada en la llamada.**

> "Es una conversación que podemos tener. Primero resolvamos el problema técnico y tengamos los datos completos sobre el impacto real — eso nos da una base sólida para cualquier conversación de ese tipo."

---

## Señal de éxito de la llamada

- Claudia dice: *"Ok, entonces estamos cubiertos por ahora."*
- Rafael dice: *"Bien, no necesito hacer nada de mi lado."*

Eso es el cierre ideal.
