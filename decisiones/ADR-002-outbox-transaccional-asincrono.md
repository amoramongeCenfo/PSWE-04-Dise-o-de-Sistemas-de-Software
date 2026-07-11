# ADR-002 — Patrón Transactional Outbox con procesamiento asíncrono para auditoría e idempotencia

| Campo | Detalle |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-07-04 |
| **Autores** | Edgar Jacob, Brandon Garita, Alejandro Mora |

**Contexto**

Aquí chocan dos escenarios directamente. QS-01 y QS-04 piden que cualquier intento de acceso no autorizado, y cada emisión fiscal, quede en un log de auditoría inmutable antes de darse por completado. Pero QS-03 exige que una respuesta automática en redes sociales salga en menos de 4 segundos (percentil 95) con 50 eventos por minuto llegando de golpe. Si escribiéramos el log de auditoría de forma síncrona, dentro del mismo hilo que atiende la petición, esa latencia se dispara apenas hay algo de carga.

A eso se le suma RF-06: el sistema recibe webhooks repetidos, preventas duplicadas y reintentos de Hacienda, así que necesitamos alguna forma explícita de no procesar dos veces el mismo evento.

**Decisión**

Cada operación fiscal guarda su evento de auditoría en una tabla outbox, dentro de la misma transacción que el cambio de estado del comprobante. El Procesador Asíncrono (los workers) toma esos registros del outbox y los manda a RabbitMQ, de ahí al Almacén de Auditoría y a las notificaciones, con reintentos, orden y deduplicación por `event_id`. Lo único que espera el camino crítico es la escritura en el outbox, unos 80 ms más; el resto pasa después, en segundo plano.

**Alternativas consideradas**

| Alternativa | Ventajas | Desventajas | Por qué se descartó |
|---|---|---|---|
| Escribir el log de auditoría directo y de forma síncrona en el mismo request | Es lo más simple de entender | Bajo la carga de QS-03 la latencia se dispara, y el log pasa a ser un punto único de falla | No cumple la medida de QS-03 y pone en riesgo la disponibilidad del flujo de facturación |
| Publicar a RabbitMQ sin pasar por un outbox (fire-and-forget) | Latencia mínima | Si la publicación falla justo después de confirmar el estado, el evento de auditoría se pierde y no hay forma de recuperarlo | Rompe la garantía de RF-05 de que toda emisión quede auditada |
| Usar Change Data Capture sobre la base de datos transaccional | El log se alimenta solo, sin tocar el código de la app | Requiere infraestructura y conocimiento que el equipo no tiene disponible ahora mismo | No se justifica frente a REST-06 dado el beneficio marginal sobre el outbox |

**Consecuencias positivas**
- El cambio de estado fiscal y la intención de auditarlo quedan atados en la misma transacción, así que nunca se confirma un comprobante sin que su evento ya esté guardado de forma durable.
- La latencia de auditoría y notificaciones deja de estar en el camino crítico, lo que permite cumplir con QS-03.
- El mismo `event_id` que usamos para el outbox nos sirve también para resolver la deduplicación de RF-06 y QS-06.

**Consecuencias negativas**
- Queda una ventana de tiempo donde el evento fiscal ya ocurrió pero todavía no aparece en el log final; hay que dejarlo claro para que no se confunda con una falta de auditoría.
- Se suma complejidad operativa: hay que vigilar el outbox, el broker y los workers, y pensar qué pasa con los eventos que agoten reintentos.
- Programar y probar bien la deduplicación por `event_id` toma tiempo de desarrollo que una escritura directa no hubiera requerido.

**Revisión requerida si:** algún regulador llega a pedir evidencia de auditoría en tiempo real dentro de la misma transacción, o si el volumen de eventos crece más de lo que una tabla outbox relacional puede sostener con buen rendimiento.
