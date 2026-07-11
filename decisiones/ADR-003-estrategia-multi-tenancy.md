# ADR-003 — Aislamiento multi-tenant mediante `tenant_id` centralizado en la capa de autorización

| Campo | Detalle |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-07-04 |
| **Autores** | Edgar Jacob, Brandon Garita, Alejandro Mora |

**Contexto**

REST-03 obliga a que la plataforma sea SaaS multi-tenant. En un sistema fiscal esto pesa más de lo normal: si se filtra información entre tenants, hablamos de datos legalmente vinculantes de una empresa apareciendo donde no deben. QS-01 pone el número exacto: 100% de los intentos de acceso cruzado bloqueados, y ninguna consulta debería poder ejecutarse sin un `tenant_id` válido. Al mismo tiempo, REST-06 nos limita bastante en presupuesto y tamaño de equipo, así que las soluciones de aislamiento físico por tenant quedan fuera de alcance para el volumen inicial de PYMES que esperamos.

**Decisión**

Usamos una sola base de datos compartida, con columna `tenant_id` en todas las tablas transaccionales y fiscales, y concentramos la resolución de ese `tenant_id` en la capa de autorización: Keycloak lo incluye como claim del JWT, y hay un único punto de autorización en la API de Aplicación que lo valida y lo propaga a cada consulta, en vez de confiar en que cada repositorio lo filtre por su cuenta.

**Alternativas consideradas**

| Alternativa | Ventajas | Desventajas | Por qué se descartó |
|---|---|---|---|
| Base de datos aparte para cada tenant | Aislamiento físico, difícil que se filtre algo | El costo y el trabajo operativo (migraciones, respaldos) se multiplican por cada tenant nuevo | No es sostenible con el presupuesto y equipo que tenemos (REST-06) |
| Un esquema separado por tenant dentro de la misma base | Más aislado que compartir esquema, sin llegar a bases separadas | Igual multiplica las migraciones a medida que crecen los tenants | Se descarta por ahora como opción por defecto; queda como posible camino para un tenant grande o con requisitos especiales más adelante |
| Filtrar `tenant_id` a mano en cada consulta o repositorio, sin un punto central | Es lo más rápido de implementar al inicio | Basta con olvidar un `WHERE` en un solo lugar para filtrar datos de otro tenant | No garantiza lo que pide QS-01 |

**Consecuencias positivas**
- Se cumple con REST-06 al operar una sola base de datos para todos los tenants.
- QS-01 queda cubierto porque la validación vive en un solo punto, fácil de probar y de auditar.
- Las migraciones de esquema son mucho más simples: un solo camino para toda la plataforma.

**Consecuencias negativas**
- Todo el aislamiento depende de ese único mecanismo de autorización; si falla ahí, falla para todos los tenants a la vez. Por eso la suite de pruebas de autorización de QS-01 no es opcional.
- Puede aparecer el problema de "noisy neighbor": un tenant con mucha carga afecta el rendimiento de los demás porque comparten la misma base. Este ADR no resuelve eso, queda como riesgo pendiente.
- Si algún cliente grande exige por contrato un aislamiento físico de sus datos, este modelo no se lo puede ofrecer tal cual.

**Revisión requerida si:** algún tenant pide aislamiento físico por contrato o regulación, o si empiezan a repetirse incidentes de "noisy neighbor" por encima de lo que el equipo considere aceptable.
