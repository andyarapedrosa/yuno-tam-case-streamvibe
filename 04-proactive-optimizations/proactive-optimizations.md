# Sección 4: Recomendaciones de Optimización Proactiva

**Merchant:** StreamVibe
**Fecha:** 25 de noviembre, 2024
**Preparado por:** Equipo Técnico Yuno
**Audiencia:** Claudia Mendez (Head of Payments) · Rafael Santos (CTO)

> Las siguientes recomendaciones son independientes del incidente activo en Brasil. Identifican oportunidades estructurales en la configuración de pagos de StreamVibe que hoy están dejando ingresos sobre la mesa — con independencia del timeline de resolución de EBANX.

---

## Recomendación 1: Convertir PIX en la Experiencia Principal de Pago en Brasil

### La Oportunidad

PIX ya es el método de pago con mayor tasa de aprobación de StreamVibe en Brasil: **93.0%** — 30 puntos por encima de tarjetas incluso en un período normal (82.5%), y 30 puntos por encima de donde están las tarjetas hoy (63.0%). En solo 6 semanas desde su lanzamiento, PIX representa el 24% del volumen de Brasil, lo que confirma que la adopción orgánica es sólida.

La oportunidad está en dejar de tratar PIX como una opción secundaria y posicionarlo estructuralmente como la primera opción del checkout.

### Los Datos

| Método de Pago | Tasa de Aprobación | Intentos | % del Volumen |
|----------------|-------------------|----------|---------------|
| Tarjetas | 63.0% (crisis) / ~82.5% (normal) | 19,200 | 67.5% |
| **PIX** | **93.0%** | **6,850** | **24.1%** |
| Boleto | 78.3% | 2,400 | 8.4% |

Si la participación de PIX crece del 24% al 40% del volumen de Brasil — un objetivo realista a 6 meses dado su ritmo de adopción — StreamVibe gana ~2,300 transacciones mensuales adicionales procesadas al 93% de aprobación en lugar del ~83% de tarjetas. A $9.99 de ticket promedio, esto representa aproximadamente **+$190 en aprobaciones mensuales por cada 1,000 suscriptores brasileños** migrados a PIX, sin ningún cambio de procesador.

PIX también liquida de forma **instantánea** (T+0 frente a T+1 o T+2 de tarjetas), mejorando el flujo de caja de StreamVibe en Brasil.

### Acciones Recomendadas

**1. Reposicionamiento en el checkout (cambio de producto — esfuerzo de ingeniería bajo):**
Mostrar PIX como la primera opción visible para usuarios brasileños en el checkout — por encima de las tarjetas, no debajo. Los datos de comportamiento en checkout indican que el orden de presentación de métodos de pago influye significativamente en la selección, especialmente para usuarios que no tienen una preferencia preestablecida.

**2. PIX Automático para cobros recurrentes (esfuerzo de ingeniería medio):**
Brasil lanzó **PIX Automático** en 2024 — un esquema que permite a los merchants iniciar débitos PIX recurrentes con autorización previa del cliente, similar a un mandato de débito directo. Es el primer método de pago en Brasil que combina la tasa de aprobación del 93%+ de PIX con cobro recurrente totalmente automatizado.

Para un negocio de suscripción, esta es la oportunidad de integración de mayor prioridad en Brasil: elimina tanto el riesgo de rechazo de tarjeta como la fricción de autenticación 3DS en las renovaciones. Yuno soporta PIX Automático — la implementación requiere que StreamVibe recolecte mandatos PIX de los nuevos suscriptores durante el proceso de alta.

**3. Incentivo PIX para el plan anual (decisión comercial — cero ingeniería):**
Ofrecer un incentivo moderado (por ejemplo, 1 mes gratis o un pequeño descuento) a los suscriptores anuales que paguen vía PIX. Esto traslada las transacciones de mayor valor al canal de mayor aprobación, mejora el flujo de caja (liquidación inmediata del año completo) y reduce el churn por renovación fallida en transacciones de $89.99.

### Impacto Estimado

| Acción | Desplazamiento de Volumen | Efecto en Tasa de Aprobación | Impacto Estimado en Ingresos/Mes |
|--------|--------------------------|------------------------------|----------------------------------|
| PIX como default en checkout | +5-10pp de participación PIX | +93% en volumen desplazado vs ~83% tarjeta | +$2,000–$5,000/mes |
| PIX Automático para renovaciones | Convierte cobros recurrentes de tarjeta | Elimina rechazos de tarjeta en renovaciones | +$8,000–$20,000/mes (post-integración) |
| Incentivo PIX plan anual | Migra ~20% suscriptores anuales a PIX | Elimina fricción 3DS en transacciones de $89.99 | Reduce churn est. 2-3% en cohorte anual |

**Esfuerzo de ingeniería de StreamVibe:** Bajo para el reposicionamiento en checkout (cambio de UI). Medio para PIX Automático (nueva integración — Yuno provee soporte de SDK y especificación técnica).

---

## Recomendación 2: Resolver Argentina Atacando el Problema Correcto — Wallet como Default + Estrategia de Pricing

### La Oportunidad

La tasa de aprobación de tarjetas en Argentina (54%) no es mejorable a través de cambios de routing — está en el piso del mercado. Sin embargo, la **billetera de Mercado Pago** de StreamVibe está rindiendo al **66.1%** — 15 puntos por encima de tarjetas, por encima del benchmark del mercado (64-70%), y completamente subutilizada. El camino para mejorar la tasa de aprobación global en Argentina es trasladar el mix de suscriptores hacia la billetera, no cambiar de procesador.

Existe también un problema estructural de pricing que es independiente del método de pago pero que alimenta el patrón de rechazos: la inflación anual en Argentina supera el 100%. Un suscriptor que se dio de alta cuando el peso estaba en X ahora enfrenta el mismo cargo en USD con un poder adquisitivo materialmente deteriorado. Esto explica en parte por qué el código 51 (fondos insuficientes) es el rechazo dominante — no es una política del banco emisor, es la capacidad financiera del suscriptor.

### Los Datos

| Método de Pago | Tasa de Aprobación | Benchmark | Diferencial vs Tarjeta |
|----------------|-------------------|-----------|------------------------|
| Tarjetas (Mercado Pago) | 51.0% | 52-58% | — |
| **Billetera Mercado Pago** | **66.1%** | 64-70% | **+15.1pp** |

Si la participación de la billetera en el volumen de Argentina crece del nivel actual (~20%) al 40%, StreamVibe gana ~2,200 transacciones adicionales procesadas al 66% en lugar del 51%. A $9.99 de ticket promedio, esto representa aproximadamente **+$330 en aprobaciones mensuales por cada 1,000 suscriptores argentinos** migrados a billetera — sin ningún cambio de procesador.

### Acciones Recomendadas

**1. Billetera como opción principal en Argentina (cambio de producto — esfuerzo de ingeniería bajo):**
En el checkout para usuarios argentinos, presentar Mercado Pago wallet como la primera opción — por encima del ingreso de datos de tarjeta. La mayoría de los compradores online en Argentina tienen cuenta de Mercado Pago. Este es un cambio de configuración de checkout específico por mercado.

**2. Prompt de alta en billetera durante el registro (cambio de producto — esfuerzo bajo):**
Para nuevos suscriptores argentinos, agregar un paso en el proceso de alta que invite a vincular su billetera de Mercado Pago. Explicar el beneficio para el usuario (checkout más rápido, sin necesidad de ingresar datos de tarjeta). Esto construye una base de suscriptores wallet-first en Argentina de cara al futuro.

**3. Evaluar pricing en ARS o segmentación de precios local (decisión comercial):**
Esta recomendación está fuera de la infraestructura de pagos, pero es importante señalarla: una parte significativa de los rechazos de tarjeta en Argentina son código 51 (fondos insuficientes), impulsados por la erosión del poder adquisitivo frente a precios en USD. Dos opciones a considerar:
- Introducir un nivel de precio en ARS que se ajuste periódicamente con la inflación (práctica habitual de servicios de streaming en Argentina — Netflix, Spotify y Disney+ utilizan precios locales)
- Ofrecer un nivel de entrada de menor costo específico para Argentina

Esta es una decisión de producto y comercial de StreamVibe, no una recomendación de pagos per se. Pero desde la perspectiva de pagos, optimizar routing y checkout sin resolver el desajuste de pricing tiene un techo. Si un suscriptor genuinamente no puede pagar el cargo, ningún método de pago lo va a aprobar.

**4. Estrategia de reintentos para rechazos blandos en Argentina:**
El código 51 (fondos insuficientes) es un rechazo blando — es reintentable. La mejor práctica para negocios de suscripción es implementar una secuencia de dunning: reintento en el día 3, día 7 y día 14 después de una renovación fallida, ya que el suscriptor puede haber recibido su sueldo o recargado su cuenta. Yuno puede configurar esta cadencia de reintentos. Actualmente no hay evidencia de que StreamVibe tenga un flujo estructurado de reintentos en Argentina.

### Impacto Estimado

| Acción | Desplazamiento de Volumen | Efecto en Tasa de Aprobación | Impacto Estimado en Ingresos/Mes |
|--------|--------------------------|------------------------------|----------------------------------|
| Billetera como default en checkout | +10-20pp de participación wallet | +15pp en volumen desplazado | +$1,500–$4,000/mes |
| Dunning de reintentos para código 51 | Captura rechazos recuperables | Recupera est. 10-15% de rechazos blandos | +$800–$2,000/mes |
| Pricing local (niveles en ARS) | Reduce código 51 en origen | Mejora estructural del baseline de AR | Estratégico — no cuantificable a corto plazo |

**Esfuerzo de ingeniería de StreamVibe:** Bajo para reposicionamiento en checkout. Mínimo para cadencia de reintentos (configuración de Yuno). Medio para nivel de pricing local (decisión de producto y finanzas, cambios en backend de precios).

---

## Recomendación 3: Tokenización + MIT para Cobros Recurrentes en Todos los Mercados

### La Oportunidad

StreamVibe es un negocio de suscripción — la mayoría de sus transacciones son renovaciones recurrentes, no ingresos de datos de tarjeta nuevos. Sin embargo, no hay evidencia de que StreamVibe esté utilizando **tokenización** y **MIT (Merchant Initiated Transaction — transacción iniciada por el merchant)** de forma sistemática para las renovaciones de suscripción. Esta es la configuración técnica de mayor apalancamiento disponible en los cinco mercados.

Lo que habilita la combinación de tokenización + MIT:

**Mayor tasa de aprobación en renovaciones:** Los bancos emisores tratan las transacciones MIT tokenizadas de forma diferente a las transacciones card-not-present estándar. Un cargo recurrente de una tarjeta conocida y tokenizada tiene estadísticamente menor probabilidad de ser rechazado porque el banco ya conoce la tarjeta y el patrón de transacción es predecible.

**Elegibilidad para exención de 3DS en transacciones recurrentes:** Tanto la regulación brasileña como los estándares de EMVco permiten exenciones de 3DS para pagos recurrentes posteriores, una vez que la transacción inicial (CIT — Cardholder Initiated Transaction) ha sido autenticada. Si StreamVibe está activando 3DS en cada renovación mensual de suscriptores ya establecidos, está generando fricción innecesaria — y con el problema actual de infraestructura 3DS de EBANX, también está generando riesgo innecesario. Las transacciones con flag MIT son elegibles para exención de desafío 3DS obligatorio.

**Reducción del alcance PCI:** La tokenización implica que StreamVibe nunca almacena datos de tarjeta en crudo — el token se almacena y se usa para cargos futuros. Esto simplifica el cumplimiento de PCI DSS y reduce la superficie de riesgo.

**Resiliencia en reintentos:** Un rechazo blando en una tarjeta tokenizada puede reintentarse de inmediato sin requerir que el suscriptor vuelva a ingresar sus datos. Esto es fundamental para recuperar el ~18% de rechazos de EBANX que son código 51 (fondos insuficientes, reintentable).

### Estado Actual (Inferido)

Con base en el patrón de rechazos — particularmente el spike de código 91 correlacionado con 3DS en lo que deberían ser suscriptores ya establecidos — es probable que StreamVibe esté marcando las renovaciones recurrentes como CIT en lugar de MIT, activando desafíos 3DS en suscriptores con historial previo. Esto es tanto un problema de fricción como de tasa de aprobación.

### Acciones Recomendadas

**1. Tokenizar todas las tarjetas en la suscripción inicial (prioridad inmediata):**
Asegurar que cuando un suscriptor ingresa su tarjeta en el alta (CIT), la tarjeta sea tokenizada a través de la tokenización de red de Yuno. Este token se utiliza para todas las renovaciones posteriores. Yuno soporta tokenización de red para Visa, Mastercard y Amex en los cinco mercados de StreamVibe.

**2. Marcar las renovaciones de suscripción como MIT:**
Todos los cargos posteriores al alta inicial (renovaciones mensuales y anuales) deben enviarse con el flag MIT y la referencia de la transacción original (el CIT). Esto:
- Le señala al banco emisor que se trata de un cargo recurrente pre-autorizado
- Elimina el requerimiento de 3DS en la renovación (elegible para exención en Brasil)
- Mejora las tasas de aprobación entre 2 y 5 puntos porcentuales en transacciones recurrentes en la mayoría de los emisores de LatAm

**3. Implementar actualización automática de credenciales (account updater):**
Una parte de los rechazos de tarjeta corresponde a tarjetas vencidas o reemplazadas (códigos 54 y 14). El servicio de account updater de Yuno refresca automáticamente los tokens almacenados cuando la tarjeta de un suscriptor es reemitida — sin requerir que el suscriptor vuelva a ingresar sus datos. Esto reduce el churn involuntario por vencimiento de tarjeta, especialmente relevante para suscriptores de plan anual.

### Impacto Estimado

| Acción | Mercados | Efecto en Tasa de Aprobación | Impacto Estimado en Ingresos/Mes |
|--------|----------|------------------------------|----------------------------------|
| Tokenización + flag MIT para renovaciones | Todos (5) | +2-5pp en cargos recurrentes | +$4,600–$11,500/mes |
| Exención 3DS para MIT en Brasil | Brasil | Elimina fricción 3DS en suscriptores establecidos | +1-3pp en renovaciones de tarjeta Brasil |
| Account updater (tarjetas vencidas) | Todos (5) | Recupera ~60-70% de rechazos código 14/54 | +$500–$1,200/mes |
| **Impacto total estimado** | | | **+$5,000–$13,000/mes** |

**Esfuerzo de ingeniería de StreamVibe:** Medio — requiere actualizar el flujo de iniciación de pagos para enviar flags MIT y almacenar referencias de token por suscripción. Yuno provee el SDK, el vault de tokens y la infraestructura de account updater. Tiempo estimado de desarrollo: 2-3 sprints. Esta es la inversión de ingeniería de mayor ROI disponible en el stack de pagos de StreamVibe.

---

## Resumen: Recomendaciones Priorizadas

| Prioridad | Recomendación | Esfuerzo | Impacto en Ingresos/Mes | Tiempo al Valor |
|-----------|--------------|----------|------------------------|-----------------|
| **1** | PIX Automático para cobros recurrentes en Brasil | Medio | +$8,000–$20,000 | 4-6 semanas |
| **2** | Tokenización + MIT para todas las renovaciones | Medio | +$5,000–$13,000 | 4-6 semanas |
| **3** | PIX como default en checkout (Brasil) | Bajo | +$2,000–$5,000 | 1-2 semanas |
| **4** | Billetera Mercado Pago como default (Argentina) | Bajo | +$1,500–$4,000 | 1-2 semanas |
| **5** | Dunning de reintentos para código 51 (Argentina) | Bajo | +$800–$2,000 | 1 semana |
| **6** | Account updater (tarjetas vencidas/reemplazadas) | Bajo | +$500–$1,200 | 1-2 semanas |

**Potencial de mejora combinado (estimación conservadora): +$17,800–$45,200/mes en todos los mercados**

Los ítems 1 y 2 tienen el mayor techo de impacto y requieren esfuerzo de ingeniería medio. Los ítems 3 al 6 son ganancias rápidas que no requieren cambios de ingeniería o son mínimos, activables en 1-2 semanas.

---

## Próximos Pasos Sugeridos

1. **Esta semana:** Acordar los ítems 3, 4 y 5 — bajo esfuerzo, alto valor, sin ingeniería requerida. Pueden activarse en días.
2. **Próximas 2 semanas:** Definir el alcance de los ítems 1 y 2 con el equipo de Rafael. El equipo de integraciones de Yuno provee la especificación técnica y documentación de SDK.
3. **Revisión a 30 días:** Reevaluar la performance de tarjetas en Argentina después de activar la billetera como default. Si sigue por debajo del 54%, iniciar la conversación sobre estrategia de pricing local con Claudia y el equipo de finanzas.
