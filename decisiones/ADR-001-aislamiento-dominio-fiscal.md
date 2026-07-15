# ADR-001 — Aislamiento del dominio fiscal en un servicio dedicado (Servicio de Facturación Fiscal)

| Campo | Detalle |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-07-04 |
| **Autores** | Edgar Jacob, Brandon Garita, Alejandro Mora |

**Contexto**

El motor de automatización externo (todavía no elegimos la herramienta puntual, ver sección 3.4) entrega preventas al sistema desde redes sociales. Ya habíamos decidido a nivel de negocio, en REST-05, que ese motor jamás puede decidir ni pedir la emisión de un comprobante fiscal. Lo que nos faltaba resolver era cómo hacer cumplir eso en el código y no solo en el papel: si la lógica fiscal vive mezclada con el resto de la aplicación, nada impide en la práctica que en algún punto se termine llamando desde donde no debería.

Otro problema que teníamos encima era QS-05: Hacienda cambia el esquema XML cada cierto tiempo y el equipo tiene que poder adaptarse en menos de 10 días hábiles sin tocar el resto del sistema. Con todo mezclado en un solo monolito eso es más difícil de garantizar.

**Decisión**

Separamos todo lo fiscal (armar el XML, firmarlo con XADES-EPES, la máquina de estados del comprobante y la comunicación con Hacienda) en su propio contenedor: el Servicio de Facturación Fiscal. Solo la API de Aplicación puede llamarlo, por REST interno. El motor de automatización nunca tiene una ruta de red hacia este servicio, ni la va a tener.

**Alternativas consideradas**

| Alternativa | Ventajas | Desventajas | Por qué se descartó |
|---|---|---|---|
| Meter la lógica fiscal dentro del mismo monolito de la API de Aplicación | Menos piezas que mantener al inicio | Cada cambio de esquema de Hacienda obliga a redesplegar toda la aplicación; no hay una frontera real frente a herramientas externas | No cumple QS-05 y la frontera de REST-05 queda como buena intención, no como algo forzado por el diseño |
| Dejar que el motor de automatización llame directamente al servicio fiscal | El handoff sería un poco más rápido | Contradice REST-05 de frente; abre el dominio fiscal a un componente externo que ni siquiera hemos terminado de elegir | Se descarta, sobre todo por seguridad (QA-01) |
| Un microservicio distinto por cada tipo de comprobante (factura, nota de crédito, nota de débito) | Cada uno podría desplegarse aparte | Demasiada infraestructura para el tamaño del equipo; los tres comparten la misma máquina de estados | No se justifica frente a REST-06 (presupuesto y equipo limitados) |

**Consecuencias positivas**
- El módulo fiscal se puede actualizar y desplegar solo, sin arrastrar al resto del sistema (esto es justamente lo que pide QS-05).
- Se reduce bastante la superficie de ataque del dominio fiscal, porque solo un componente interno puede hablarle.
- La frontera de REST-05 deja de ser una regla escrita en un documento y pasa a estar forzada por la arquitectura.

**Consecuencias negativas**
- Se agrega un salto de red más entre la API de Aplicación y el Servicio de Facturación Fiscal, que hay que vigilar para no comernos el presupuesto de latencia de QS-03.
- Ahora hay un servicio adicional que desplegar, versionar y monitorear.
- Se necesita autenticación entre servicios internos, además de la que ya existe para los usuarios.

**Revisión requerida si:** el volumen de comprobantes crece tanto que haga falta escalar cada tipo de documento por separado, o si Hacienda llega a exigir validación síncrona en tiempo real (lo que eliminaría el estado intermedio `pendiente_validacion_hacienda`).
