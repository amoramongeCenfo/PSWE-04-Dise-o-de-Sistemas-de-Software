# SmartBilling Connect
> Documento de Diseño de Software — PSWE-04  
> Universidad Cenfotec · Maestría Profesional en Ingeniería del Software

---

| Campo | Detalle |
|---|---|
| **Nombre del sistema** | SmartBilling Connect |
| **Grupo** | D |
| **Integrantes** | Edgar Jacob — 117660514, Brandon Garita — 207180701, Alejandro Mora — 111890536 |
| **URL del repositorio** | https://github.com/amoramongeCenfo/PSWE-04-Dise-o-de-Sistemas-de-Software/tree/main |
| **Docente** | Juan Mauricio Leandro Jiménez |
| **Cuatrimestre** | 2026 — 02 |
| **Versión del documento** | 1.0 — Entrega final |
| **Fecha de última actualización** | 2026-08-11 |

---

## Historial de versiones

| Versión | Fecha | Hito | Cambios principales | Autor(es) |
|---|---|---|---|---|
| 0.1 | 2026-05-23 | Propuesta (S03) | Creación del documento inicial | Edgar Jacob, Brandon Garita, Alejandro Mora |
| 0.2 | 2026-06-12 | Avance 1 (S07) | Descripción del sistema, alcance, stakeholders, drivers arquitectónicos, escenarios de calidad y Vista de Contexto C4 | Edgar Jacob, Brandon Garita, Alejandro Mora |
| 0.3 | 2026-07-11 | Avance 2 (S11) | Vista de estructura interna C4 nivel 2 (contenedores) con justificación de notación, diagrama y tabla descriptiva, y definición del stack tecnológico (.NET 8 / ASP.NET Core, SQL Server, Keycloak, RabbitMQ, MinIO); vista de comportamiento con diagramas de secuencia de los flujos críticos; estilo(s) arquitectónico(s) adoptado(s) con alternativas y análisis de trade-offs; registro de decisiones (ADRs); y primer componente con diseño detallado. | Edgar Jacob, Brandon Garita, Alejandro Mora |
| 1.0 | 2026-08-11 | Entrega final (S14) | Refinamiento de las vistas de contexto (7.1) y contenedores (7.2) para consolidar su consistencia mutua; **vista de componentes C4 nivel 3** de dos subsistemas —Servicio de Facturación Fiscal y Procesador Asíncrono— (7.2.4); **vista de comportamiento completa** con cinco flujos y sus caminos de error (7.3); **vista de despliegue** sobre VM cloud + Docker Compose con nodos, artefactos y conectividad (7.4); **vista de concurrencia** con modelo de outbox/competing consumers, idempotencia y concurrencia optimista (7.5); **documentación de la evolución del diseño** entre avances (7.6); y consolidación del diseño detallado de componentes, patrones, análisis de calidad, tendencias, glosario y referencias. | Edgar Jacob, Brandon Garita, Alejandro Mora |

---

## Tabla de contenidos

1. [Descripción del sistema y alcance](#1-descripción-del-sistema-y-alcance)
2. [Stakeholders](#2-stakeholders)
3. [Drivers arquitectónicos](#3-drivers-arquitectónicos)
4. [Requerimientos de calidad — Escenarios](#4-requerimientos-de-calidad--escenarios)
5. [Restricciones](#5-restricciones)
6. [Principios de diseño adoptados](#6-principios-de-diseño-adoptados)
7. [Vistas arquitectónicas](#7-vistas-arquitectónicas)
   - 7.1 [Vista de contexto](#71-vista-de-contexto)
   - 7.2 [Vista de estructura interna](#72-vista-de-estructura-interna)
   - 7.3 [Vista de comportamiento](#73-vista-de-comportamiento)
   - 7.4 [Vista de despliegue](#74-vista-de-despliegue)
   - 7.5 [Vista de concurrencia](#75-vista-de-concurrencia-opcional) *(si aplica)*
   - 7.6 [Evolución del diseño entre avances](#76-evolución-del-diseño-entre-avances)
8. [Estilo arquitectónico](#8-estilo-arquitectónico)
9. [Registro de decisiones — ADRs](#9-registro-de-decisiones--adrs)
10. [Diseño detallado de componentes](#10-diseño-detallado-de-componentes)
11. [Patrones de diseño aplicados](#11-patrones-de-diseño-aplicados)
12. [Principios y técnicas habilitadoras — evidencia](#12-principios-y-técnicas-habilitadoras--evidencia)
13. [Análisis de calidad del diseño](#13-análisis-de-calidad-del-diseño)
14. [Secciones específicas por tipo de sistema](#14-secciones-específicas-por-tipo-de-sistema) *(si aplica)*
15. [Tendencias y evolución del diseño](#15-tendencias-y-evolución-del-diseño)
16. [Glosario](#16-glosario)
17. [Referencias](#17-referencias)

---

# BLOQUE 1 — CONTEXTO Y PROBLEMA
*Hito: Propuesta (S03) y Avance 1 (S07)*

---

## 1. Descripción del sistema y alcance

### 1.1 Descripción general
El sistema propuesto, **SmartBilling Connect**, consiste en una plataforma de facturación electrónica inteligente orientada a pequeñas y medianas empresas (PYMES) que comercializan productos y servicios mediante redes sociales y canales digitales. Su propósito principal es centralizar y simplificar la gestión comercial y administrativa de negocios que actualmente manejan sus ventas de forma manual, utilizando múltiples herramientas desconectadas entre sí. La plataforma permitirá administrar clientes, registrar ventas y gestionar comprobantes electrónicos, brindando trazabilidad y control sobre el proceso comercial.

> **Alcance del término "inteligente":** en este diseño, "inteligente" se refiere a la **automatización del flujo comercial basada en reglas y workflows** (disparadores, conectores y respuestas configuradas), no a inteligencia artificial generativa, clasificación, recomendación o asistencia conversacional avanzada. Esto es coherente con la exclusión 1.3.18. Por lo tanto, el sistema **no** asume los retos arquitectónicos propios de la IA generativa (trazabilidad de decisiones del modelo, evaluación de respuestas, fallback de modelo, explicabilidad); su complejidad arquitectónica proviene de la **coordinación confiable** de un flujo fiscal entre sistemas heterogéneos, no de un componente de IA.

Actualmente, muchas PYMES utilizan redes sociales como principal canal de ventas, especialmente plataformas de mensajería y comercio digital. Sin embargo, estos negocios enfrentan problemas relacionados con la duplicidad de información, pérdida de seguimiento de clientes, errores en la generación de facturas y procesos operativos poco eficientes. En muchos casos, las conversaciones con clientes ocurren en redes sociales mientras la facturación se realiza manualmente en otros sistemas, generando retrasos, inconsistencias y una alta dependencia de tareas repetitivas realizadas por el personal administrativo.

El sistema busca resolver este problema mediante una solución integrada que permita conectar la actividad comercial con los procesos de facturación electrónica y seguimiento de clientes. El valor principal de la plataforma radica en reducir la carga operativa, mejorar la eficiencia administrativa y aumentar la trazabilidad de las ventas realizadas por medios digitales. Además, permitirá a los negocios responder con mayor rapidez a sus clientes, disminuir errores humanos y cumplir con las obligaciones fiscales de forma más ordenada y automatizada.

### 1.2 Contexto del negocio o dominio
El sistema se desarrolla dentro del dominio de comercio digital y facturación electrónica para pequeñas y medianas empresas (PYMES). En Costa Rica y otros países de la región, una gran cantidad de negocios utilizan redes sociales como principal canal de ventas y atención al cliente, especialmente mediante plataformas de mensajería y comercio social. Muchos emprendimientos realizan cotizaciones, coordinan pedidos y concretan ventas directamente desde aplicaciones como Instagram, Facebook o WhatsApp, mientras que la facturación y el control administrativo se gestionan posteriormente de forma manual o mediante herramientas independientes.

Este modelo operativo genera múltiples problemas de negocio: duplicidad de información, errores de digitación, falta de trazabilidad entre conversaciones y ventas, dificultad para monitorear clientes potenciales y dependencia excesiva de procesos manuales. Además, conforme aumenta el volumen de ventas, las empresas enfrentan dificultades para mantener consistencia en la información fiscal y brindar tiempos de respuesta rápidos a sus clientes. El sistema propuesto busca soportar y optimizar estos procesos mediante la integración de la gestión comercial con la facturación electrónica y automatización de flujos operativos.

Dentro del dominio existen regulaciones relevantes asociadas principalmente al cumplimiento tributario. En Costa Rica, la emisión de comprobantes electrónicos debe cumplir con los lineamientos definidos por el Ministerio de Hacienda, incluyendo validación fiscal, almacenamiento de documentos electrónicos y manejo adecuado de información tributaria. Esto implica requisitos relacionados con seguridad, integridad de datos, auditoría y trazabilidad de transacciones. El sistema deberá garantizar que las facturas emitidas sean válidas conforme a la normativa vigente y que exista un historial verificable de las operaciones realizadas.

El sistema también se relaciona con procesos comerciales y de atención al cliente. Entre los procesos soportados se encuentran: registro y administración de clientes, generación de cotizaciones, emisión de comprobantes electrónicos, seguimiento de ventas, automatización de respuestas y sincronización de información proveniente de redes sociales. Parte importante del dominio es la automatización de tareas repetitivas, permitiendo que eventos generados en plataformas externas activen flujos de negocio relacionados con ventas o facturación.

Desde una perspectiva arquitectónica, el dominio presenta retos relevantes relacionados con integración de sistemas externos, manejo seguro de información fiscal, tolerancia a fallos en procesos de facturación y coordinación de eventos provenientes de múltiples canales digitales. También existe una tensión importante entre rapidez de respuesta en la atención comercial y consistencia de la información transaccional, especialmente cuando varios usuarios o integraciones interactúan simultáneamente con los mismos datos comerciales y fiscales.

### 1.3 Alcance del sistema

**Dentro del alcance**

#### 1.3.1 Gestión de usuarios y acceso

El sistema permite el registro, autenticación y administración de usuarios con roles diferenciados, incluyendo administrador, vendedor y asistente administrativo, cada uno con permisos específicos según sus responsabilidades.

#### 1.3.2 Administración de clientes

El sistema permite registrar, consultar y actualizar información de clientes, incluyendo datos fiscales, información de contacto e historial de compras y facturación.

#### 1.3.3 Emisión de facturación electrónica

El sistema permite generar, emitir, almacenar y consultar comprobantes electrónicos conforme a la normativa tributaria vigente.

#### 1.3.4 Gestión de cotizaciones

El sistema permite generar cotizaciones comerciales y convertirlas posteriormente en facturas electrónicas manteniendo trazabilidad entre ambos procesos.

#### 1.3.5 Integración con redes sociales

El sistema permite recibir y relacionar eventos o interacciones provenientes de redes sociales y canales digitales con procesos comerciales internos.

#### 1.3.6 Monitoreo de comprobantes electrónicos

El sistema permite consultar y monitorear el estado de documentos electrónicos enviados a la administración tributaria.

#### 1.3.7 Gestión de productos y servicios

El sistema permite administrar catálogos básicos de productos y servicios utilizados en cotizaciones y facturación.

#### 1.3.8 Consulta y trazabilidad comercial

El sistema permite consultar historial de ventas, facturación e interacciones comerciales mediante filtros y búsquedas avanzadas.

#### 1.3.9 Gestión de permisos y auditoría

El sistema permite controlar accesos según roles y registrar auditoría de acciones realizadas por usuarios y automatizaciones externas.

#### 1.3.10 Notificaciones y comunicación

El sistema permite enviar comprobantes electrónicos y notificaciones a clientes mediante correo electrónico u otros canales configurados.


**Fuera del alcance**

#### 1.3.11 ERP financiero y contabilidad avanzada

El sistema no implementa funcionalidades completas de ERP financiero o contable, como contabilidad general, conciliaciones bancarias, cuentas por pagar o generación de estados financieros.

#### 1.3.12 Gestión avanzada de inventarios

El sistema no administra procesos avanzados de inventario, tales como manejo multi-bodega, trazabilidad física, logística de distribución o planificación de abastecimiento.

#### 1.3.13 Procesamiento directo de pagos

El sistema no procesa pagos electrónicos directamente ni funciona como pasarela de pago financiera. Las transacciones monetarias dependerán de servicios externos especializados.

#### 1.3.14 CRM empresarial avanzado

El sistema no reemplaza plataformas completas de CRM empresarial; únicamente administra información comercial relacionada con ventas, clientes y facturación.

#### 1.3.15 Marketing digital y publicidad

El sistema no realiza campañas de marketing digital, segmentación publicitaria ni administración de anuncios pagados en redes sociales.

#### 1.3.16 Almacenamiento completo de conversaciones

El sistema no almacena conversaciones completas provenientes de redes sociales de forma indefinida, excepto la información necesaria para trazabilidad comercial y automatización de procesos.

#### 1.3.17 Gestión de recursos humanos

El sistema no incluye funcionalidades relacionadas con nómina, control de personal, recursos humanos o administración de empleados.

#### 1.3.18 Inteligencia artificial conversacional avanzada

El sistema no implementa asistentes inteligentes avanzados ni reemplaza completamente la atención humana al cliente; únicamente soporta automatizaciones configuradas.

#### 1.3.19 Disponibilidad de servicios externos

El sistema no garantiza disponibilidad continua de APIs externas, plataformas tributarias, servicios de redes sociales o herramientas de automatización integradas.

#### 1.3.20 Soporte multinacional

El sistema no contempla inicialmente adaptación automática a regulaciones fiscales de múltiples países; el alcance se limita al contexto tributario costarricense.

#### 1.3.21 Business Intelligence avanzado

El sistema no incluye funcionalidades avanzadas de analítica empresarial, minería de datos o procesamiento masivo orientado a Business Intelligence.

#### 1.3.22 Plataforma completa de comercio electrónico

El sistema no implementa una tienda virtual pública, marketplace o carrito de compras completo para comercio electrónico.

### 1.4 Usuarios y casos de uso principales
| Tipo de usuario | Casos de uso principales |
|---|---|
| Administrador del sistema | Configurar parámetros fiscales y de integración, administrar usuarios y permisos, monitorear automatizaciones, auditar actividades del sistema, consultar métricas operativas |
| Vendedor / Ejecutivo comercial | Registrar clientes, generar cotizaciones, convertir cotizaciones en facturas electrónicas, consultar historial comercial, gestionar seguimiento de ventas provenientes de redes sociales |
| Asistente administrativo | Emitir facturas electrónicas, validar información fiscal de clientes, reenviar comprobantes electrónicos, consultar estados tributarios, corregir errores operativos de facturación |
| Cliente final | Recibir comprobantes electrónicos, consultar cotizaciones enviadas, confirmar pedidos o servicios, recibir notificaciones comerciales, interactuar mediante canales digitales |
| Motor de automatización | Atender mensajes y preguntas de usuarios en redes sociales, responder de forma automática, guiar al usuario hacia la aplicación ante intención de compra y generar una preventa que ingresa al sistema (no participa en facturación) |
| Servicios externos tributarios | Validar comprobantes electrónicos, recibir documentos fiscales, retornar estados de aceptación o rechazo, verificar cumplimiento tributario |

### 1.5 Núcleo arquitectónico: el flujo crítico

La capacidad funcional del sistema (clientes, cotizaciones, facturación, notificaciones, auditoría) no es lo que determina su dificultad arquitectónica. El **problema central de SmartBilling Connect no es "facturar desde redes sociales", sino coordinar de forma confiable un flujo transaccional de larga duración** en el que participan canales externos no confiables, una capa de automatización conversacional, usuarios internos, un módulo fiscal, una autoridad tributaria (Hacienda), la auditoría y la notificación al cliente. Si este flujo no se modela como una cadena explícita de estados y responsabilidades, la arquitectura corre el riesgo de convertirse en una suma de módulos sin lógica transaccional clara.

El flujo crítico tiene **dos tramos con responsables distintos**. El primero es la **captación social**, gestionada por el motor de automatización: atiende mensajes y preguntas de usuarios en redes sociales, responde de forma automática y, cuando detecta intención de compra, **guía al usuario hacia la aplicación generando una preventa** (lead de pre-venta) que ingresa al sistema. A partir de la preventa, el segundo tramo es **interno y fiscal**: dentro de la aplicación un usuario (o proceso interno) convierte la preventa en cotización y comprobante, y el sistema gestiona el ciclo fiscal hasta Hacienda. **La frontera entre ambos tramos es deliberada: el motor de automatización no entra al dominio fiscal** (ver REST-05 y sección 3.4).

El ciclo de vida se formaliza así (la marca ⟦motor⟧ delimita el alcance del motor de automatización):

```
⟦motor de automatización: captación social⟧
  interacción social → respuesta/guía automática → preventa (intención de compra)
        │
        ▼  (handoff: la preventa ingresa a la aplicación)
⟦sistema: dominio interno y fiscal⟧
  oportunidad/cotización → comprobante generado → enviado a Hacienda →
  pendiente | aceptado | rechazado → notificación → auditoría
```

![Ciclo de vida del comprobante](../diagramas/ciclo-vida-factura.png)
*Figura 1 — Ciclo de vida: captación social (motor de automatización) y tramo interno/fiscal, con responsables de cada transición*

Cada transición de este ciclo plantea preguntas arquitectónicas que el diseño debe responder en los hitos siguientes, pero que quedan formuladas desde ya: qué componente es responsable de cada transición, qué pasa si una transición falla o se ejecuta dos veces, y cuándo una transición es definitiva (irreversible). Estas preocupaciones se concretan en las fuentes de verdad e invariantes de la sección 1.6, en el escenario de idempotencia QS-06 (sección 4) y en la frontera del motor de automatización (REST-05 y sección 3.4).

### 1.6 Fuentes de verdad e invariantes del dominio

El sistema integra varias fuentes de datos (CRM interno, módulo fiscal, motor de automatización, Hacienda, auditoría, redes sociales). Para evitar inconsistencias, ambigüedad transaccional y exposición de datos, se declara explícitamente **quién es la fuente de verdad de cada entidad** y quién puede cambiar su estado.

| Entidad | Fuente de verdad | Quién puede cambiar el estado | Reversibilidad |
|---|---|---|---|
| **Factura — datos previos al envío** (borrador, líneas, montos) | SmartBilling Connect (módulo fiscal) | Usuario autorizado o integración, dentro del sistema principal | Reversible mientras esté en borrador / no enviada |
| **Factura — estado fiscal** (aceptado / rechazado) | **Hacienda** es la autoridad; SmartBilling refleja y conserva ese estado | Solo Hacienda determina aceptado/rechazado; el sistema nunca lo fija por su cuenta | **Irreversible** una vez aceptada |
| **Estado `pendiente_validacion_hacienda`** | SmartBilling Connect (estado interno legítimo y auditable) | El sistema, mientras espera la respuesta de Hacienda | Transitorio: converge a aceptado/rechazado |
| **Preventa** (lead de pre-venta) | Motor de automatización: la genera desde la conversación social y la entrega a la app | El motor la crea; una vez ingresada, el sistema/usuario interno la gestiona. El motor no la modifica después del handoff | Reversible (puede descartarse antes de convertirse en oportunidad) |
| **Oportunidad de venta** | CRM interno de SmartBilling (no la red social, que es solo canal de origen) | Usuario interno (vendedor) dentro de la app; el motor de automatización solo aporta la preventa inicial, no la oportunidad | Reversible |
| **Registro de auditoría** | Log append-only de SmartBilling | Nadie lo modifica tras escribirlo (ni administradores) | **Irreversible / inmutable** |
| **Notificación al cliente** | Servicio de correo/mensajería (mero ejecutor, nunca fuente de verdad del estado fiscal) | El sistema dispara; el proveedor solo entrega | N/A |

**Invariantes no negociables del dominio fiscal.** El diseño debe preservar estas reglas en todos los hitos:

1. Una factura **aceptada por Hacienda no puede ser modificada ni eliminada**; la corrección se hace mediante nota de crédito/débito, no editando el original.
2. Toda factura pertenece a **exactamente un tenant**; ninguna operación puede cruzar esa frontera.
3. Todo comprobante emitido tiene **trazabilidad del actor** (usuario o integración) que lo originó.
4. Todo intento de acceso a datos fiscales es **autorizable y auditable**.
5. Una factura **no puede pasar a "notificada"** si no existe un estado fiscal válido o explícitamente `pendiente_validacion_hacienda`.
6. Un **rechazo de Hacienda** debe generar una ruta de corrección controlada, no un estado terminal silencioso.
7. Ninguna consulta a repositorios de datos fiscales se ejecuta **sin un `tenant_id` válido** resuelto por el mecanismo central de autorización.

---
## 2. Stakeholders
 
**Plataforma de Facturación Electrónica Inteligente con Automatización Comercial y Redes Sociales**
 
Un stakeholder es cualquier persona, grupo u organización que tiene interés en el sistema, no solo los usuarios directos. Esta sección incluye desarrolladores, operadores, reguladores, áreas de negocio y proveedores externos. Para cada uno se identifica qué quieren del sistema y qué les preocupa. Esta tabla es la base de los drivers arquitectónicos de la sección 3.
 
---
 
## STK-01: Dueños de PYMES
 
**Rol:** Propietarios y tomadores de decisiones de pequeñas y medianas empresas que venden a través de redes sociales.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Reducir la carga operativa en facturación y gestión de ventas. | Costo de adopción: la solución debe ser asequible para una PYME. |
| Centralizar ventas, mensajería y facturación en una sola plataforma. | Curva de aprendizaje: no tienen equipo técnico dedicado. |
| Convertir interacciones en redes sociales en ventas registradas y facturadas. | Continuidad: el sistema no puede fallar en horas de venta. |
| Obtener visibilidad sobre el estado comercial del negocio. | Cumplimiento fiscal sin complejidad adicional. |
---
 
## STK-02: Personal administrativo
 
**Rol:** Empleados que operan el sistema día a día: emiten facturas, gestionan clientes y procesan cotizaciones.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Rapidez en la emisión de comprobantes electrónicos. | Interfaces complejas que ralenticen el trabajo. |
| Reducción de errores por ingreso manual de datos. | Pérdida de datos por fallos del sistema. |
| Flujos de trabajo claros y predecibles. | Doble digitación entre sistemas desconectados. |
| Acceso a historial de transacciones por cliente. | Falta de soporte ante errores de facturación. |
 
---
 
## STK-03: Clientes finales
 
**Rol:** Personas o empresas que compran productos/servicios de las PYMES y reciben los comprobantes electrónicos.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Respuestas rápidas a consultas realizadas por redes sociales. | Tiempos de espera excesivos en la atención automatizada. |
| Recibir comprobantes electrónicos válidos de forma oportuna. | Recibir comprobantes con errores o inválidos ante Hacienda. |
| Transparencia en precios y condiciones de venta. | Privacidad de sus datos personales y fiscales. |
 
---
 
## STK-04: Administradores del sistema
 
**Rol:** Personal técnico o funcional responsable de configurar, monitorear y mantener la plataforma en operación.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Control centralizado sobre configuración de tenants y usuarios. | Fallos silenciosos en integraciones con APIs externas. |
| Monitoreo en tiempo real del estado de integraciones externas. | Escalabilidad ante crecimiento de clientes (multi-tenant). |
| Capacidad de auditoría sobre todas las transacciones. | Gestión de certificados de firma digital y su renovación. |
| Herramientas de diagnóstico ante fallos. | Seguridad: accesos no autorizados, exfiltración de datos fiscales. |
 
---
 
## STK-05: Entidades tributarias (Ministerio de Hacienda)
 
**Rol:** Ente regulador que define los requisitos legales y técnicos para la facturación electrónica en Costa Rica.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Que los comprobantes emitidos cumplan el esquema XML vigente. | Emisión de comprobantes fraudulentos o manipulados. |
| Que los documentos tengan firma digital válida (XADES-EPES). | Incumplimiento del formato o protocolo de comunicación. |
| Que exista trazabilidad fiscal completa y auditable. | Imposibilidad de fiscalizar por falta de registros. |
| Que los comprobantes se conserven por el plazo legal (5 años). | |
 
---
 
## STK-06: Equipo de desarrollo
 
**Rol:** Desarrolladores e ingenieros responsables de construir, evolucionar y mantener la plataforma.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Arquitectura clara con subsistemas desacoplados. | Deuda técnica acumulada por decisiones apresuradas. |
| Stack tecnológico accesible y bien documentado. | Complejidad excesiva al integrar múltiples APIs externas. |
| Facilidad para agregar nuevas integraciones o adaptar las existentes. | Dependencia de herramientas cuya viabilidad a largo plazo es incierta (ej. decisión pendiente sobre el motor de automatización). |
| Procesos de despliegue y pruebas automatizados. | Mantenimiento de compatibilidad ante cambios de Hacienda o de redes sociales. |
 
---
 
## STK-07: Proveedores de APIs externas (Meta, TikTok)
 
**Rol:** Plataformas de redes sociales cuyos servicios se integran al sistema mediante APIs públicas.
 
| Intereses principales | Preocupaciones o restricciones |
|---|---|
| Que el consumo de sus APIs respete los términos de servicio. | Uso indebido de datos de usuarios obtenidos a través de sus APIs. |
| Que los flujos OAuth cumplan sus especificaciones de seguridad. | Abuso de cuotas de API o scraping no autorizado. |
| Que el volumen de llamadas se mantenga dentro de los rate limits. | Impacto reputacional si la integración genera spam o mala experiencia al usuario final. |
 
---
 
## Tabla resumen
 
| ID | Stakeholder | Tipo | Influencia | Drivers asociados |
|---|---|---|---|---|
| STK-01 | Dueños de PYMES | Usuario / Negocio | **Alta** | RF-01, RF-02, RF-03, RF-06, QA-02, REST-03 |
| STK-02 | Personal administrativo | Usuario operativo | **Media** | RF-03, RF-04, QA-04 |
| STK-03 | Clientes finales | Usuario externo | **Baja** | RF-02, QA-02, QA-04 |
| STK-04 | Administradores del sistema | Operador / Técnico | **Alta** | RF-04, RF-05, RF-06, QA-01, QA-03, QA-05, REST-05 |
| STK-05 | Entidades tributarias | Regulador | **Alta** | RF-01, RF-05, RF-06, QA-01, REST-01, REST-02 |
| STK-06 | Equipo de desarrollo | Constructor | **Alta** | QA-03, QA-05, REST-05, REST-06 |
| STK-07 | Proveedores APIs (Meta, TikTok) | Proveedor externo | **Media** | RF-02, QA-03, REST-04 |
## 3. Drivers Arquitectónicos

Plataforma de Facturación Electrónica Inteligente con Automatización Comercial y Redes Sociales

Los drivers arquitectónicos son los factores que moldean de forma determinante la arquitectura del sistema. No representan la totalidad de requerimientos, sino aquellos cuya omisión provocaría un fallo sistémico o un diseño fundamentalmente inadecuado (SWEBOK v3, Cap. 2 -- Software Design). A continuación se clasifican en tres categorías: requerimientos funcionales clave, atributos de calidad prioritarios y restricciones que actúan como drivers.

### 3.1 Requerimientos funcionales clave

Se incluyen únicamente los requerimientos funcionales con impacto arquitectónico directo: aquellos que obligan a tomar decisiones de estructura, componentes o integración, no de implementación interna.

| **ID** | **Requerimiento** | **Stakeholder** | **Por qué es un driver** |
|---|---|---|---|
| **RF-01** | Emisión de comprobantes electrónicos (factura, nota de crédito, nota de débito) cumpliendo el esquema XML del Ministerio de Hacienda de Costa Rica. | Dueños de PYMES, Entidades tributarias | Define el subsistema central del dominio. Obliga a un motor de facturación con generación XML, firma digital, envío al API de Hacienda y manejo de estados (aceptado/rechazado). Impone estructura de colas y reintentos. |
| **RF-02** | Integración con APIs de redes sociales (Meta Platforms, TikTok) para capturar mensajes y consultas que el motor de automatización atiende y deriva como preventas. | Dueños de PYMES, Clientes finales | Introduce un subsistema de integración social con sus propias fronteras, protocolos de autenticación OAuth y manejo de webhooks. Requiere un intermediario (el motor de automatización) que desacople la mensajería social del core de facturación, recibiendo el sistema solo el handoff de preventa. |
| **RF-03** | Automatización de la atención en redes sociales mediante un motor de workflows: responder mensajes y preguntas de usuarios, guiarlos hacia la aplicación con intención de compra y generar una **preventa** que ingresa al sistema. | Dueños de PYMES, Personal administrativo | Obliga a incorporar el motor de automatización como componente externo acotado a la **capa de captación social**, con una frontera explícita frente al dominio fiscal (sección 3.4). Define el contrato de handoff (preventa → app) y el patrón de comunicación, sin que el motor participe en la emisión fiscal. |
| **RF-04** | Gestión multiusuario con roles diferenciados (administrador, vendedor, auditor) y permisos granulares por operación. | Administradores del sistema, Personal administrativo | Obliga a un subsistema transversal de autenticación y autorización (Identity Provider). Afecta cada punto de entrada del sistema y requiere decisiones sobre protocolos (JWT, OAuth2) y almacenamiento de sesiones. |
| **RF-05** | Registro de auditoría completo e inmutable de todas las transacciones fiscales y acciones de usuarios. | Entidades tributarias, Administradores del sistema | Impone un log de auditoría append-only separado del almacenamiento transaccional. Afecta la estrategia de persistencia y puede requerir un almacén de eventos o base de datos dedicada para trazabilidad fiscal. |
| **RF-06** | Procesamiento idempotente de eventos externos (webhooks de redes sociales, respuestas de Hacienda, llamadas del motor de automatización, reintentos por timeout y acciones manuales), garantizando que un mismo evento no produzca efectos duplicados. | Dueños de PYMES, Entidades tributarias, Administradores | Es un driver porque el sistema es, por naturaleza, un receptor de eventos potencialmente duplicados. Obliga a un mecanismo transversal de claves de idempotencia / deduplicación por `event_id` y a definir qué operaciones son seguras de reintentar. Sin esto se producen efectos inaceptables: doble emisión de factura, doble notificación al cliente, doble registro de auditoría o doble cambio de estado. |

### 3.2 Atributos de calidad prioritarios

Se priorizan los cinco atributos de calidad más críticos para este sistema. La selección responde al contexto específico del dominio: un sistema fiscal con integración a servicios externos y automatización en tiempo real. Si todo es prioridad, nada lo es — por eso se limita a cinco.

| **ID** | **Atributo** | **Importancia** | **Stakeholder** | **Justificación** |
|---|---|---|---|---|
| **QA-01** | Seguridad | **Alta** | Entidades tributarias, Administradores | El sistema maneja información fiscal legalmente vinculante y datos sensibles de clientes (NIF, direcciones, montos). Una brecha comprometería la validez legal de los comprobantes y expondría a sanciones. Requiere firma digital, cifrado en tránsito/reposo y control de acceso estricto. |
| **QA-02** | Disponibilidad | **Alta** | Dueños de PYMES, Clientes finales | Las PYMES dependen del sistema para facturar en tiempo real. Una caída durante horas pico significa pérdida directa de ventas. La integración con redes sociales exige que el sistema esté disponible cuando llegan mensajes (24/7). Objetivo mínimo: 99.5% uptime mensual. |
| **QA-03** | Interoperabilidad | **Alta** | Dueños de PYMES, Administradores | El sistema debe comunicarse con al menos tres ecosistemas externos: API de Hacienda (XML/SOAP), APIs de redes sociales (REST/webhooks) y motor de automatización. Si la interoperabilidad falla, el valor diferenciador de la plataforma desaparece. |
| **QA-04** | Rendimiento | **Media-Alta** | Personal administrativo, Clientes finales | Las respuestas automáticas a consultas en redes sociales deben procesarse en segundos para no perder oportunidades comerciales. La emisión de facturas debe completarse en menos de 5 segundos incluyendo la respuesta de Hacienda. Tiempos mayores degradan la experiencia y generan doble envío. |
| **QA-05** | Modificabilidad | **Media** | Administradores, Dueños de PYMES | Las regulaciones fiscales cambian periódicamente (nuevos campos, versiones de XML, tarifas impositivas). Las APIs de redes sociales actualizan sus contratos con frecuencia. El sistema debe absorber estos cambios sin rediseño arquitectónico, lo que exige bajo acoplamiento entre subsistemas. |

### 3.3 Restricciones que actúan como drivers

Las siguientes restricciones no son negociables y condicionan directamente las decisiones arquitectónicas. Se clasifican por tipo según su origen.

| **ID** | **Restricción** | **Tipo** | **Impacto en el diseño** |
|---|---|---|---|
| **REST-01** | Los comprobantes electrónicos deben cumplir el formato XML y el protocolo de firma digital establecidos por el Ministerio de Hacienda de Costa Rica. | Regulatoria | Fuerza el uso de librerías de firma digital (XADES-EPES), certificados específicos (Firma Digital CR) y un módulo dedicado a construir y validar el XML fiscal. No hay margen de negociación sobre el formato. |
| **REST-02** | Los documentos fiscales emitidos deben conservarse por un mínimo de 5 años con integridad demostrable. | Regulatoria | Obliga a una estrategia de almacenamiento de largo plazo con respaldos, checksums de integridad y posible almacenamiento en frío. Afecta la selección de base de datos y la política de retención. |
| **REST-03** | La plataforma debe operar como servicio multi-tenant orientado a PYMES con modelo de suscripción (SaaS). | Negocio | **En un sistema fiscal, el multi-tenancy es una decisión arquitectónica crítica, no un detalle técnico.** No basta con un `tenant_id`: condiciona el aislamiento de datos fiscales entre empresas, el control de acceso por tenant, la trazabilidad y auditoría separadas por tenant, la estrategia de respaldos y recuperación por cliente, las migraciones de esquema sin afectar a todos los tenants y el cumplimiento normativo individual. Su principal riesgo arquitectónico es la **exposición cruzada de información fiscal** entre tenants, lo que obliga a centralizar la resolución de `tenant_id` en la capa de autorización (ver invariante 7 de la sección 1.6 y escenario QS-01). |
| **REST-04** | El sistema debe integrarse con APIs externas de Meta Platforms y TikTok, sujetas a sus términos de servicio, procesos de aprobación y límites de tasa. | Técnica | Las APIs de terceros imponen rate limiting, flujos OAuth específicos y revisiones de app. Obliga a implementar circuit breakers, colas de reintento, almacenamiento local de tokens y manejo de degradación cuando las APIs no están disponibles. |
| **REST-05** | El motor de automatización (herramienta a definir) se limita a la **capa de captación social**: responde mensajes/preguntas en redes sociales, guía al usuario hacia la app y genera una preventa. **Decisión de frontera ya tomada: NO participa en el dominio fiscal ni solicita la emisión de comprobantes.** | Técnica | La arquitectura define una interfaz abstracta para el motor (herramienta intercambiable, a definir en el análisis arquitectónico) y un **contrato de handoff** preventa → app como único punto de entrada del motor. El riesgo a evitar es que una herramienta externa concentre lógica de negocio fiscal. Por eso validación, emisión, cotización, autorización, auditoría e idempotencia del dominio fiscal permanecen dentro del sistema; el motor nunca cruza esa frontera. Cada llamada del motor (handoff de preventa) se autentica y audita como cualquier integración. |
| **REST-06** | El presupuesto y equipo corresponden a un proyecto académico/startup con recursos limitados. | Negocio | Restringe el uso de servicios cloud costosos, licencias comerciales y tecnologías que requieran expertise especializado escaso. Favorece stack open-source, servicios gestionados con capa gratuita y arquitectura que pueda operarse con un equipo pequeño. |

---

> **Nota metodológica:** *Esta clasificación sigue los lineamientos del SWEBOK v3 (Capítulo 2 -- Software Design), que establece que los drivers arquitectónicos comprenden los requerimientos funcionales significativos, los atributos de calidad que el sistema debe satisfacer y las restricciones del entorno de desarrollo y operación. Los stakeholders referenciados corresponden a la sección 2 del documento de arquitectura.*

> **Nota sobre el motor de automatización:** *La selección de la herramienta sigue sujeta al análisis arquitectónico, por lo que REST-05 mantiene su intercambiabilidad (cualquier motor de workflows o desarrollo propio). Lo que sí está decidido es su **alcance**: el motor es la capa de captación social y no participa del dominio fiscal.*

#### 3.4 Frontera del motor de automatización

El motor de automatización se limita a la **capa de captación en redes sociales**. Su responsabilidad termina cuando entrega una **preventa** a la aplicación; a partir de ahí, todo el dominio fiscal vive en **SmartBilling Connect**. Esta frontera impide que una herramienta externa concentre lógica de negocio fiscal crítica.

| El motor de automatización **SÍ** puede | El motor de automatización **NO** puede |
|---|---|
| Atender, responder y dar seguimiento a mensajes/preguntas de usuarios en redes sociales. | Decidir, solicitar o validar la emisión de un comprobante fiscal. |
| Guiar al usuario hacia la aplicación cuando detecta intención de compra. | Crear o modificar cotizaciones, facturas o cualquier dato fiscal. |
| Generar una **preventa** y entregarla a la app mediante el contrato de handoff. | Participar como coordinador de la transacción fiscal o cambiar estados fiscales. |
| Consultar estados públicos expuestos por la API (p. ej. para responder al usuario). | Escribir en el log de auditoría o alterar su contenido. |

**Garantías sobre las acciones del motor:** el handoff de preventa se **autentica** como una integración más (credenciales propias, sin sesión de usuario humano), se **audita** igual que cualquier acción del sistema (RF-05) y es **idempotente** (RF-06), de modo que reejecutar un workflow no genera preventas duplicadas. Si un workflow social queda a medio ejecutar, no hay impacto fiscal: el dominio fiscal solo avanza por acción interna de la app a partir de una preventa ya recibida.
---

# BLOQUE 2 — REQUERIMIENTOS DE CALIDAD


## 4. Requerimientos de calidad — Escenarios 
 
Definir atributos de calidad en abstracto no es suficiente para tomar decisiones arquitectónicas. Un enunciado como "el sistema debe ser seguro y rápido" no le dice nada al equipo de diseño: no indica cuándo ocurre el problema, qué parte del sistema lo enfrenta ni cómo se mide el éxito. Los escenarios de calidad resuelven eso: convierten cada atributo en una situación concreta, con un actor real, una condición medible y una respuesta esperada del sistema.
 
Cada escenario en esta sección sigue la estructura de seis elementos definida en la norma ISO/IEEE: fuente del estímulo, estímulo, entorno de operación, artefacto afectado, respuesta esperada del sistema y medida de respuesta. Las medidas son siempre numéricas; expresiones como "rápido" o "disponible" no califican como criterios de aceptación en un diseño arquitectónico serio.
 
Se documentan seis escenarios que cubren cinco preocupaciones de calidad distintas: seguridad, disponibilidad, rendimiento, modificabilidad e idempotencia/integridad transaccional. Estos atributos fueron priorizados en la sección 3.2 como los más críticos para el dominio de SmartBilling Connect. Las medidas se expresan como **criterios de aceptación verificables en pruebas controladas**, no como deseos absolutos: en ingeniería casi nunca se puede demostrar un "cero" en términos absolutos, por lo que se acota a lo que una suite de pruebas puede comprobar. Al cierre de la sección se analizan las tensiones entre escenarios que entran en conflicto, porque es precisamente en esos conflictos donde se toman las decisiones arquitectónicas más importantes.
 
---
 
### QS-01 — Seguridad: Acceso no autorizado a datos fiscales de un tenant
 
En una plataforma multi-tenant, el riesgo más serio no siempre viene de afuera: un usuario legítimo de un tenant podría intentar —por error o con intención— acceder a los datos de otro. Este escenario valida que el aislamiento entre tenants es real y que cualquier intento de cruzar esa frontera queda registrado.
 
| Elemento | Descripción |
|---|---|
| **Fuente** | Actor externo no autenticado, o usuario autenticado perteneciente a un tenant diferente al del recurso solicitado. |
| **Estímulo** | Intento de consultar, modificar o exportar comprobantes electrónicos o datos fiscales de un tenant ajeno, ya sea mediante llamadas directas a la API REST o mediante manipulación de parámetros de sesión. |
| **Entorno** | Sistema en operación normal, con múltiples tenants activos simultáneamente. |
| **Artefacto** | Subsistema de autenticación y autorización (Identity Provider) y capa de acceso a datos con filtros por `tenant_id`. |
| **Respuesta** | El sistema rechaza la solicitud con HTTP 403, registra el intento en el log de auditoría inmutable con timestamp, IP de origen, usuario y recurso solicitado, y no devuelve ningún dato del tenant objetivo. |
| **Medida de respuesta** | El **100 % de los intentos incluidos en la suite de pruebas de autorización** (RBAC + aislamiento de tenant) son bloqueados con HTTP 403. **Ninguna consulta sin `tenant_id` válido puede ejecutarse contra los repositorios de datos fiscales** (verificado en pruebas de integración). Toda consulta multi-tenant pasa por el mecanismo centralizado de autorización. El evento de intento queda registrado en auditoría en ≤ 500 ms. Ningún registro de un tenant ajeno aparece en la respuesta para los casos cubiertos por la suite. |
 
 > **Tensión con QS-04 — Seguridad vs. Rendimiento:** El registro de auditoría que exige QS-01 por cada intento de acceso no autorizado comparte la misma infraestructura de log append-only que QS-04 utiliza para trazabilidad fiscal. Si ese log se convierte en un cuello de botella bajo carga alta, ambos escenarios se ven afectados. La decisión de diseño implicada es la misma que se detalla en QS-04: persistir el evento de auditoría de forma transaccional en un outbox durable antes de responder y procesar el log append-only final de forma asíncrona, con reintentos, orden e idempotencia. Esto evita la falsa contradicción entre "auditoría obligatoria" y "procesamiento asíncrono".


---
 
### QS-02 — Disponibilidad: Fallo del servicio de validación de Hacienda
 
La plataforma depende de una API externa para validar los comprobantes electrónicos. Esa dependencia es inevitable por regulación, pero no puede significar que el negocio se detenga cada vez que Hacienda tenga problemas técnicos. Este escenario define cómo debe comportarse el sistema cuando ese servicio externo falla, especialmente durante las horas de mayor actividad comercial.
 
| Elemento | Descripción |
|---|---|
| **Fuente** | API del Ministerio de Hacienda de Costa Rica (servicio externo fuera del control de la plataforma). |
| **Estímulo** | El endpoint de validación de comprobantes electrónicos retorna timeout o error HTTP 5xx de forma sostenida durante un período de indisponibilidad. |
| **Entorno** | Sistema en operación durante hora pico de facturación, entre las 8:00 a.m. y las 6:00 p.m. hora Costa Rica. |
| **Artefacto** | Módulo de facturación electrónica y la cola de reintentos asociada para persistencia temporal de documentos pendientes. |
| **Respuesta** | El sistema acepta el comprobante generado localmente, lo encola para reenvío automático, comunica al usuario el estado "pendiente de validación fiscal" y continúa operando con normalidad para todas las demás funciones. Cuando la conectividad se restaura, procesa la cola en orden FIFO sin intervención manual. |
| **Medida de respuesta** | Tiempo de degradación perceptible para el usuario ≤ 2 segundos (únicamente cambia el estado visible del comprobante). Documentos encolados procesados automáticamente en ≤ 10 minutos tras la restauración del servicio externo. Disponibilidad del flujo de facturación local, sin depender de Hacienda, ≥ 99.5 % mensual. Ningún documento encolado se pierde ante reinicios del sistema en la suite de pruebas de resiliencia, garantizado por persistencia durable de la cola. |
 
 > **Tensión con QS-04 — Disponibilidad vs. Integridad fiscal:** QS-02 requiere que el sistema persista comprobantes localmente cuando Hacienda no responde. Pero QS-04 exige que cada comprobante tenga su entrada de auditoría con un estado definitivo. Durante una ventana de indisponibilidad, el estado real del comprobante ante Hacienda es desconocido, lo que crea una inconsistencia temporal entre el log interno y el registro fiscal oficial. La decisión de diseño implicada es modelar `pendiente_validacion_hacienda` como un estado fiscal legítimo y auditable: el log captura el timestamp de encolado y el de confirmación posterior, de modo que la integridad del proceso queda trazada aunque la validación no sea inmediata.


---
 
### QS-03 — Rendimiento: Respuesta automática a consulta en red social durante evento de ventas
 
Uno de los valores diferenciales de SmartBilling Connect es responder automáticamente a mensajes de clientes en redes sociales. Pero ese valor desaparece si la respuesta llega tarde: en ventas por canal digital, unos pocos segundos de demora pueden ser la diferencia entre cerrar una venta o perderla. Este escenario evalúa el comportamiento del sistema bajo la carga concentrada típica de una campaña activa.
 
| Elemento | Descripción |
|---|---|
| **Fuente** | Cliente final que envía un mensaje de consulta de precio o disponibilidad mediante Instagram Direct o WhatsApp Business, recibido por el motor de automatización vía webhook de Meta Platforms. |
| **Estímulo** | Llegada simultánea de 50 eventos webhook en un intervalo de 60 segundos durante una campaña de ventas activa. |
| **Entorno** | Plataforma bajo carga sostenida, con el motor de automatización procesando flujos concurrentes en la capa de captación social (sección 3.4). |
| **Artefacto** | Motor de automatización (capa de captación social), API pública de consulta del sistema y endpoint de handoff de preventas. |
| **Respuesta** | El motor de automatización procesa cada evento, consulta a la API del sistema si el cliente ya existe en el CRM (acceso de solo lectura permitido por la frontera de 3.4), genera la respuesta automática configurada y la envía por el canal de origen, sin requerir intervención humana en ningún paso; ante intención de compra, entrega la preventa al sistema mediante el handoff idempotente. |
| **Medida de respuesta** | Tiempo de respuesta extremo a extremo —desde la recepción del webhook hasta el envío de la respuesta al cliente— ≤ 4 segundos en el percentil 95 bajo la carga descrita. Throughput mínimo sostenido de 50 eventos por minuto sin degradación. Tasa de mensajes no procesados por timeout del motor inferior al 1 % del total recibido. |
 
 > **Tensión con QS-01 y QS-04 — Rendimiento vs. Seguridad:** El log de auditoría append-only exigido por QS-01 y QS-04 introduce una escritura obligatoria en cada operación sensible. Si esa escritura es síncrona y bloqueante dentro del camino crítico del procesamiento de webhooks, la latencia acumulada puede superar fácilmente los 4 segundos bajo carga sostenida. La decisión de diseño implicada distingue dos casos: para **operaciones fiscales**, el evento se persiste transaccionalmente en un outbox durable antes de responder y el log append-only final se procesa de forma asíncrona (no es un bloqueo síncrono del camino crítico); para **eventos comerciales de redes sociales**, es aceptable un modelo "at-least-once" con verificación post-proceso. En ambos casos el procesamiento asíncrono garantiza reintentos, orden, idempotencia y detección de duplicados.


---
 
### QS-04 — Seguridad / Trazabilidad: Registro de auditoría ante emisión masiva de comprobantes
 
La facturación electrónica tiene implicaciones legales directas. Cuando un proceso automatizado emite cientos de facturas en un lote, debe existir evidencia verificable de cada operación: quién la originó, cuándo ocurrió, cuál fue el resultado y que el documento no fue alterado después. Este escenario es especialmente relevante porque el actor que dispara el proceso no es un humano sino un proceso interno programado del sistema (no el motor de automatización, que no participa del dominio fiscal — ver 3.4).
 
| Elemento | Descripción |
|---|---|
| **Fuente** | Proceso interno programado del sistema (p. ej. job de cierre de mes) actuando como actor no humano dentro del dominio fiscal. |
| **Estímulo** | Emisión de 200 facturas electrónicas en lote durante un cierre de mes. |
| **Entorno** | Operación normal en sistema multi-tenant, con varios tenants procesando transacciones simultáneamente. |
| **Artefacto** | Log de auditoría append-only y subsistema de facturación electrónica. |
| **Respuesta** | Por cada comprobante emitido, el sistema registra en el log de auditoría la identidad del actor (usuario o integración), el tenant de origen, un timestamp con precisión de milisegundo, el número de comprobante, el estado resultante (aceptado, rechazado o pendiente) y el hash de integridad del documento XML. Ningún actor del sistema —incluyendo administradores— puede modificar ni eliminar entradas del log una vez escritas. |
| **Medida de respuesta** | Antes de que el sistema retorne la respuesta de una operación fiscal, el evento de auditoría se **persiste transaccionalmente en un outbox durable** (misma transacción que el cambio de estado del comprobante); el procesamiento hacia el log append-only final ocurre de forma asíncrona, con reintentos, orden garantizado y deduplicación. Así, el 100 % de los comprobantes emitidos en la suite de pruebas tiene su evento de auditoría persistido de forma durable antes de responder, sin que el log final sea un bloqueo síncrono en el camino crítico. La latencia adicional del registro transaccional en outbox es ≤ 80 ms por comprobante. La integridad del log es verificable mediante hashes encadenados: ninguna entrada puede alterarse sin invalidar todas las posteriores. El log se conserva por un mínimo de 5 años (REST-02). |
 
 > **Tensión con QS-05 — Trazabilidad vs. Modificabilidad:** El despliegue en caliente requerido por QS-05 implica que durante la ventana de transición pueden coexistir en producción dos versiones del módulo fiscal. El log de auditoría debe mantener coherencia entre registros generados por versiones distintas del mismo módulo, y un token válido para una versión no debería poder operar contra la otra. La decisión de diseño implicada es que cada entrada del log incluya la versión del módulo fiscal que generó el comprobante, y que el subsistema de autorización aplique control de versión en los contratos de API internos.


---
 
### QS-05 — Modificabilidad: Adaptación a cambio en el esquema XML de Hacienda
 
Las regulaciones fiscales no son estáticas. Hacienda actualiza periódicamente el esquema XML de los comprobantes electrónicos, y el sistema debe poder absorber esos cambios sin afectar a los tenants en producción ni requerir una intervención de emergencia del equipo. Este escenario verifica que el módulo fiscal está suficientemente aislado como para ser actualizado de forma independiente.
 
| Elemento | Descripción |
|---|---|
| **Fuente** | Ministerio de Hacienda de Costa Rica en su rol de regulador externo. |
| **Estímulo** | Publicación de una nueva versión del esquema XML para comprobantes electrónicos, con cambios en campos obligatorios y reglas de validación, y un plazo de adopción de 60 días calendario. |
| **Entorno** | Sistema en producción con tenants activos emitiendo comprobantes a diario. |
| **Artefacto** | Módulo de construcción y validación del XML fiscal, aislado del resto del sistema mediante una interfaz de contrato definida. |
| **Respuesta** | El equipo de desarrollo modifica exclusivamente el módulo fiscal sin tocar otros subsistemas. El cambio atraviesa el pipeline de integración continua, se valida en el ambiente de pruebas y se despliega en producción sin downtime mediante un despliegue en caliente. |
| **Medida de respuesta** | Tiempo total desde la publicación del nuevo esquema hasta el despliegue en producción validado: ≤ 10 días hábiles. Número de subsistemas ajenos al módulo fiscal que requieren modificación: cero. Cobertura de pruebas automatizadas del módulo fiscal antes del despliegue: ≥ 90 % de los casos de validación definidos por Hacienda. |
 
 > **Tensión con QS-01 y QS-04 — Modificabilidad vs. Seguridad:** La coexistencia de dos versiones del módulo fiscal durante el despliegue amplía temporalmente la superficie de ataque del sistema. Esta tensión ya fue abordada desde la perspectiva de trazabilidad en QS-04; desde la perspectiva de acceso, la decisión complementaria es que el subsistema de autorización emita tokens con alcance de versión explícito, de modo que una sesión autenticada solo pueda operar contra la versión del módulo para la que fue emitida.


---
 
### QS-06 — Idempotencia / Integridad: Evento externo duplicado

Por su naturaleza, el sistema recibe eventos que pueden llegar duplicados: webhooks de redes sociales reenviados, **preventas duplicadas por reejecución de un workflow del motor de automatización**, respuestas tardías o repetidas de Hacienda y reintentos internos por timeout en la emisión fiscal. Sin un tratamiento explícito, un duplicado puede provocar una preventa repetida, una doble notificación, una doble emisión de factura (por reintento interno) o un cambio de estado aplicado dos veces. Este escenario verifica que el sistema reconoce y neutraliza los duplicados (RF-06).

| Elemento | Descripción |
|---|---|
| **Fuente** | Sistema externo o reintento interno: webhook de Meta/TikTok reenviado, handoff de preventa repetido por reejecución del motor de automatización, respuesta duplicada de Hacienda, o reintento interno de emisión fiscal por timeout. |
| **Estímulo** | Llega un evento que ya fue procesado: la misma solicitud de emisión, la misma confirmación de Hacienda o el mismo webhook arriba dos o más veces. |
| **Entorno** | Operación normal, con reintentos activos y al-menos-una-entrega ("at-least-once") en los canales de integración. |
| **Artefacto** | Puntos de entrada de eventos (webhooks, API de integración, callbacks de Hacienda), almacén de claves de idempotencia y módulo de facturación. |
| **Respuesta** | El sistema detecta el duplicado mediante una clave de idempotencia / `event_id` y reconoce la operación como ya aplicada, retornando el resultado previo sin volver a ejecutar el efecto. No se emite una segunda factura, no se envía una segunda notificación y no se duplica el registro de auditoría. |
| **Medida de respuesta** | En la suite de pruebas de idempotencia, el 100 % de los eventos duplicados reconocidos produce **exactamente un** efecto de negocio (una factura, una notificación, una entrada de auditoría lógica). Ninguna operación fiscal marcada como idempotente genera un segundo comprobante ante reentrega. La ventana de deduplicación cubre al menos el periodo máximo de reintento configurado para cada canal. |
 
 > **Tensión con QS-03 — Idempotencia vs. Rendimiento:** La verificación de la clave de idempotencia agrega una consulta al almacén de deduplicación en el camino de cada evento. Bajo la carga de QS-03 (50 eventos/min) esto suma latencia. La decisión de diseño implicada es usar un almacén de claves de baja latencia (p. ej. índice único o caché durable) y aplicar la verificación solo a operaciones con efecto de negocio, no a consultas de solo lectura.


---


## 5. Restricciones

Las restricciones que actúan como **drivers** ya se detallaron en la sección 3.3 (REST-01 a REST-06), porque condicionan directamente decisiones arquitectónicas. Esta sección consolida la **lista completa y autoritativa** de restricciones del proyecto e incorpora la columna **Origen** (quién impone cada restricción), que complementa el análisis de impacto de 3.3. Para evitar duplicación, el detalle de impacto en el diseño permanece en 3.3; aquí se resume y se añade el origen.

| ID | Restricción | Tipo | Origen | Impacto en el diseño (resumen — ver 3.3) |
|---|---|---|---|---|
| REST-01 | Comprobantes en formato XML y firma digital del Ministerio de Hacienda de Costa Rica. | Regulatoria | Ministerio de Hacienda CR | Módulo fiscal dedicado con firma XADES-EPES y certificados Firma Digital CR. |
| REST-02 | Conservación de documentos fiscales ≥ 5 años con integridad demostrable. | Regulatoria | Ministerio de Hacienda CR | Almacenamiento de largo plazo con checksums y política de retención. |
| REST-03 | Operación SaaS multi-tenant para PYMES. | Negocio | Equipo / modelo de negocio | Aislamiento de datos fiscales por tenant; `tenant_id` centralizado en autorización (driver crítico). |
| REST-04 | Integración con APIs de Meta Platforms y TikTok sujetas a sus ToS y rate limits. | Técnica | Proveedores externos (Meta, TikTok) | Circuit breakers, colas de reintento, OAuth y manejo de degradación. |
| REST-05 | Motor de automatización a definir; limitado a la captación social, no es dueño del dominio fiscal. | Técnica | Equipo / análisis arquitectónico | Interfaz abstracta de workflows; reglas fiscales dentro del sistema (ver 3.4). |
| REST-06 | Recursos limitados de proyecto académico/startup. | Negocio | Equipo / contexto del curso | Favorece stack open-source y servicios con capa gratuita. |
| REST-07 | Cumplimiento de la Ley 8968 de Protección de Datos Personales (Costa Rica) para datos de clientes finales. | Regulatoria | PRODHAB (regulador CR) | Consentimiento, minimización y derecho de acceso/eliminación sobre datos personales; refuerza cifrado y control de acceso (QA-01). |

> **Nota:** REST-07 se incorpora en este avance porque el sistema procesa datos personales de clientes finales (contacto, identificación fiscal) provenientes de redes sociales, lo que activa la Ley 8968 además del régimen tributario.

---

## 6. Principios de diseño adoptados

El grupo se compromete a respetar los siguientes principios durante todo el diseño. Se eligieron por su relevancia directa para un sistema fiscal, multi-tenant y orientado a eventos; en la sección 12 se aportará evidencia concreta de su aplicación.

| Principio | Justificación para este sistema |
|---|---|
| **Separación de responsabilidades** | El dominio fiscal, la integración social, la orquestación y la auditoría tienen ciclos de cambio y niveles de criticidad muy distintos. Separarlos evita que un cambio en un canal social afecte la lógica fiscal y permite aislar lo regulado de lo no regulado. |
| **Diseño para el cambio (bajo acoplamiento)** | Las reglas de Hacienda y los contratos de las APIs sociales cambian con frecuencia (QA-05, QS-05). El sistema aísla el módulo fiscal y el motor de automatización tras interfaces para absorber cambios sin rediseño. |
| **Defensa en profundidad** | Al manejar datos fiscales y personales (QA-01, REST-07), la seguridad no puede depender de una sola capa: autenticación, autorización por tenant, cifrado en tránsito/reposo y auditoría inmutable se combinan. |
| **Fuente de verdad única por entidad** | Para evitar inconsistencias entre CRM, módulo fiscal, motor de automatización y Hacienda, cada dato tiene un único dueño autoritativo (sección 1.6). Hacienda es autoridad del estado fiscal; el sistema, de los datos previos. |
| **Idempotencia por diseño** | Al ser un receptor de eventos potencialmente duplicados (RF-06, QS-06), las operaciones con efecto de negocio se diseñan para producir el mismo resultado ante reentregas. |
| **Principio de menor privilegio (PoLA)** | Cada actor e integración —incluido el motor de automatización— opera con el mínimo de permisos necesarios; este se limita a la captación social y a entregar preventas, sin ningún permiso sobre el dominio fiscal (sección 3.4). |
| **KISS / YAGNI** | Dado el contexto de recursos limitados (REST-06), se evita la sobre-ingeniería: solo se introduce complejidad arquitectónica donde un driver o invariante lo justifica. |

---

# BLOQUE 3 — VISTAS ARQUITECTÓNICAS
*Hito: Avance 1 (S07) — sección 7.1 / Avance 2 (S11) — secciones 7.2 y 7.3 / Entrega final (S14) — secciones 7.4 y 7.5*

> **Nota sobre notación:** Este documento usa **C4 como notación por defecto** para las vistas arquitectónicas porque es la notación del texto base del curso (Brown, 2014). Si para alguna vista específica C4 no es la notación más adecuada dado el tipo de sistema, el grupo puede usar la notación UML equivalente (diagrama de componentes, despliegue, estado o actividad), pero debe justificar explícitamente en esa sección por qué C4 no aplica y qué notación alternativa usa. Una justificación insuficiente se evalúa como si no hubiera diagrama.

---

## 7. Vistas arquitectónicas

### 7.1 Vista de contexto
*Hito: Avance 1 (S07)*

> **Qué muestra:** El sistema como una caja negra en su entorno. Las personas y sistemas externos que interactúan con él. Las relaciones entre ellos. **No muestra** lo que hay dentro del sistema.
>
> **Notación:** C4 nivel 1 (Context Diagram). Aplica a todos los tipos de sistema — un sistema embebido, un pipeline de datos, un monolito y una plataforma SaaS todos tienen contexto externo.
>
> **Instrucciones:** Incluí el diagrama (imagen exportada o código PlantUML/Mermaid en `/diagramas/c4-contexto.puml`). Debajo del diagrama, describí cada elemento: el sistema central, cada actor externo (persona o rol) y cada sistema externo, con una oración que explique la naturaleza de la relación.

![Vista de contexto](../diagramas/c4-contexto.png)
*Figura 2 — Vista de contexto del sistema SmartBilling Connect*

| Elemento | Tipo | Descripción de la relación |
|---|---|---|
| **SmartBilling Connect** | Sistema principal | Plataforma SaaS de facturación electrónica inteligente para PYMES. Centraliza la gestión de clientes, cotizaciones, facturación electrónica y automatización de flujos comerciales provenientes de canales digitales. |
| **Dueño de PYME** | Persona / Rol | Accede al sistema mediante la interfaz web para configurar parámetros fiscales y de integración, administrar usuarios y permisos, monitorear automatizaciones y consultar métricas operativas del negocio. Interacción: HTTPS / Web UI. |
| **Vendedor / Ejecutivo** | Persona / Rol | Utiliza el sistema para registrar clientes, generar cotizaciones, convertir cotizaciones en facturas electrónicas y dar seguimiento a ventas originadas en redes sociales. Interacción: HTTPS / Web UI. |
| **Asistente Administrativo** | Persona / Rol | Opera el sistema para emitir comprobantes electrónicos, validar información fiscal de clientes, reenviar documentos y corregir errores operativos de facturación. Interacción: HTTPS / Web UI. |
| **Cliente Final** | Persona / Rol externo | Recibe comprobantes electrónicos y notificaciones comerciales, consulta cotizaciones enviadas y confirma pedidos. No accede al back-office del sistema. Interacción: correo electrónico / portal web. |
| **API Ministerio de Hacienda CR** | Sistema externo | El sistema envía comprobantes electrónicos firmados digitalmente en formato XML y consulta su estado de validación (aceptado / rechazado / en proceso). Es el punto de cumplimiento tributario obligatorio para toda factura emitida. Interacción: XML sobre HTTPS. |
| **Meta Platforms (Instagram / WhatsApp)** | Sistema externo | Canal social donde los clientes envían mensajes e interacciones. La atención y respuesta automática las gestiona el motor de automatización, no directamente el sistema; SmartBilling recibe el resultado como preventa. Interacción: REST / Webhook (vía el motor de automatización). |
| **TikTok Business API** | Sistema externo | Canal social cuyos eventos de interacción atiende el motor de automatización, que deriva las oportunidades con intención de compra al sistema como preventas. Interacción: REST / Webhook (vía el motor de automatización). |
| **Motor de Automatización** | Sistema externo | Capa de captación social: atiende mensajes y preguntas de usuarios en redes sociales, responde de forma automática, los guía hacia la aplicación y, ante intención de compra, entrega una **preventa** al sistema mediante el contrato de handoff. **No participa en la emisión ni en ningún proceso fiscal.** Interacción: REST / Eventos (handoff de preventa). |
| **Servicio de Correo Electrónico** | Sistema externo | El sistema delega en este servicio el envío de comprobantes electrónicos, cotizaciones y notificaciones a los clientes finales. Interacción: SMTP / API. |
 
#### 7.1.1 Fronteras de confianza

![Vista de Fronteras de Confianza](../diagramas/c4-contexto-confianza.png)
*Figura 3 — Fronteras de confianza del sistema SmartBilling Connect*

No todos los actores y sistemas externos tienen el mismo nivel de confianza, y esa distinción —no solo el diagrama— guía decisiones de seguridad, validación e idempotencia. Se clasifican así:

| Nivel de confianza | Elementos | Implicación arquitectónica |
|---|---|---|
| **Autoridad fiscal** | API Ministerio de Hacienda CR | Define el estado fiscal de los comprobantes (fuente de verdad del estado, sección 1.6). El sistema confía en su veredicto pero debe tolerar su indisponibilidad (QS-02) y respuestas tardías o duplicadas (QS-06). |
| **Interno confiable** | Dueño de PYME, Vendedor, Asistente Administrativo | Operan autenticados y autorizados por tenant (RBAC). Confiables, pero toda acción es auditable (RF-05) y acotada por menor privilegio. |
| **Externo de bajo control (canales)** | Meta Platforms, TikTok | Canales no confiables: disponibilidad y rate limits ajenos, payloads no validados. Toda entrada se valida y se trata como potencialmente duplicada o maliciosa. No son fuente de verdad de oportunidades. |
| **Automatización de captación social (semi-confiable)** | Motor de automatización | Acotado a la capa social: responde y guía a usuarios y entrega preventas. **No** toca el dominio fiscal (sección 3.4). Se autentica como integración, se audita y su handoff es idempotente; un fallo suyo no tiene impacto fiscal. |
| **Ejecutor sin autoridad** | Servicio de Correo Electrónico | Solo entrega mensajes; **nunca** es fuente de verdad del estado de un comprobante. Su fallo no debe bloquear ni alterar el estado fiscal. |
| **Externo no autenticado** | Cliente Final | Recibe comprobantes/notificaciones; no accede al back-office. Sin privilegios sobre datos de otros. |

> **Nota:** La agrupación visual de estos elementos por frontera de confianza se presenta en la Figura 3, que complementa la Figura 2 haciendo explícito el límite entre "lo que el sistema controla" y "lo que delega o recibe de terceros".

> **Refinamiento en la Entrega final (S14).** Al bajar a las vistas de contenedores (7.2), componentes (7.2.4) y despliegue (7.4) se revisó que no apareciera ningún actor o sistema externo nuevo ni desapareciera ninguno de los definidos en Avance 1: el contexto se mantiene con los mismos 9 elementos de frontera y sus mismos niveles de confianza. El único ajuste consciente fue reforzar en el texto que **Meta y TikTok llegan al sistema exclusivamente a través del Motor de Automatización** (nunca directo), coherente con la frontera de la sección 3.4; esto ya estaba implícito en Avance 1 y ahora es explícito en todas las vistas. La vista de contexto, por tanto, no cambió estructuralmente entre avances (ver trazabilidad completa en 7.6).

---

### 7.2 Vista de estructura interna
*Hito: Avance 2 (S11)*

> **Qué muestra:** Las piezas principales que componen el sistema, cómo están organizadas y cómo se comunican entre sí. Esta vista es la base del diseño detallado.
>
> **Notación por defecto — C4 nivel 2 (Container Diagram):** Usá esta notación si tu sistema tiene unidades desplegables separadas — APIs, aplicaciones web, bases de datos, servicios de mensajería, apps móviles, procesos batch, etc. "Contenedor" en C4 no es Docker — es cualquier unidad de ejecución o almacenamiento con una frontera propia.
>
> **Alternativa justificada:** Si el sistema es un monolito, un firmware, un sistema embebido o un pipeline de datos sin unidades desplegables separadas, podés usar un **diagrama de componentes UML** o un **diagrama de módulos**. Justificá en la subsección 7.2.1 por qué C4 contenedores no es la representación más honesta para tu sistema.
>
> **Instrucciones:** Para cada contenedor o componente principal, describí: su responsabilidad, la tecnología usada, las interfaces que expone y las dependencias que tiene. Las relaciones entre contenedores deben indicar el protocolo o mecanismo de comunicación (REST, gRPC, eventos, SQL, etc.).

#### 7.2.1 Justificación de notación

**Se usa C4 nivel 2 (Container Diagram)** porque SmartBilling Connect no es un monolito único ni un firmware/pipeline sin unidades desplegables: es una plataforma SaaS multi-tenant compuesta por **nueve unidades desplegables y de almacenamiento con frontera propia**, que se ejecutan y escalan de forma independiente y que se comunican por protocolos explícitos. C4 nivel 2 es la representación más honesta porque el valor arquitectónico del sistema está justamente en *cómo se reparten las responsabilidades entre esas unidades* y en las fronteras entre ellas (en particular, el aislamiento del dominio fiscal).

Las nueve unidades son: (1) Aplicación Web (SPA), (2) API de Aplicación, (3) Servicio de Facturación Fiscal, (4) Procesador Asíncrono (Workers), (5) Identity Provider, (6) Broker de Mensajería, (7) Base de Datos Transaccional, (8) Almacén de Auditoría append-only y (9) Almacén de Documentos Fiscales.

Esta separación no es cosmética: responde directamente a los drivers. El **Servicio de Facturación Fiscal** se aísla como contenedor propio para mantener el dominio fiscal dentro del sistema y fuera del alcance del motor de automatización (REST-05, sección 3.4) y para poder endurecer su seguridad de forma independiente (QA-01). El **Almacén de Auditoría** es un contenedor separado, append-only, porque RF-05 exige un log inmutable distinto del almacenamiento transaccional. El **Procesador Asíncrono** y el **Broker de Mensajería** materializan el patrón *outbox* + procesamiento asíncrono que resuelve la tensión seguridad/rendimiento de QS-01/QS-04 y la idempotencia de RF-06/QS-06. El **Identity Provider** externaliza autenticación/autorización multi-tenant (RF-04, REST-03). El stack se apoya en componentes gratuitos u open-source (React, ASP.NET Core, Keycloak, RabbitMQ, MinIO) y en **SQL Server** como motor de datos —cubierto por su edición gratuita (Express/Developer) o por licenciamiento existente del equipo—, lo que mantiene el costo acotado a la restricción de presupuesto REST-06.

> **Nota:** "Contenedor" en C4 no significa Docker; designa cualquier unidad de ejecución o almacenamiento con frontera propia. La vista de despliegue físico (nodos, infraestructura) corresponde a la sección 7.4.

#### 7.2.2 Diagrama

![Vista de estructura interna](../diagramas/c4-contenedores.png)
*Figura 4 — Vista de estructura interna (C4 nivel 2 — Contenedores) del sistema SmartBilling Connect. Código fuente en `/diagramas/c4-contenedores.mmd`.*

**Consistencia con la vista de contexto (7.1).** Los mismos actores y sistemas externos de la Figura 2 reaparecen aquí en los bordes, con idéntico nivel de confianza y protocolo: los usuarios internos (🟢) entran por HTTPS; el Motor de Automatización (🟠) entrega la preventa por REST/HTTPS autenticado e idempotente y **no cruza al dominio fiscal**; Meta y TikTok (🔴) solo tocan al motor por webhook REST; la API de Hacienda (🔵) intercambia XML firmado sobre HTTPS y devuelve el estado; el Servicio de Correo (⚪) recibe SMTP/API y entrega al Cliente Final. Lo que la vista de contexto trataba como una caja negra ("SmartBilling Connect") se abre aquí en sus nueve contenedores, sin añadir ni quitar relaciones externas: ninguna dependencia externa nueva aparece y ninguna del contexto desaparece.

#### 7.2.3 Descripción de elementos

| Elemento | Tipo | Responsabilidad | Tecnología | Interfaces expuestas | Dependencias |
|---|---|---|---|---|---|
| **Aplicación Web (SPA)** | Contenedor (frontend) | Interfaz de back-office para usuarios internos: gestión de clientes, cotizaciones, preventas, emisión y monitoreo. No contiene lógica fiscal. | React + TypeScript | UI web sobre HTTPS | API de Aplicación (REST), Identity Provider (login OIDC) |
| **API de Aplicación** | Contenedor (servicio) | Núcleo aplicativo (monolito modular): CRM/clientes, cotizaciones, preventas, productos y orquestación del flujo. Único punto de entrada del handoff de preventa. Escribe dominio + eventos en la tabla *outbox* en la misma transacción. | ASP.NET Core 8 (C#) | REST/JSON `/api/v1` sobre HTTPS (Bearer JWT); endpoint de handoff de preventa | BD Transaccional (SQL, ADO.NET/TLS), Servicio de Facturación Fiscal (REST interno), Identity Provider (validación JWT) |
| **Servicio de Facturación Fiscal** | Contenedor (servicio) | Dominio fiscal aislado: genera el XML, aplica firma XADES-EPES, gestiona la máquina de estados del comprobante y toda la comunicación con Hacienda. Fuente del estado interno `pendiente_validacion_hacienda`. | ASP.NET Core 8 (C#) | REST/HTTPS interno (consumido por la API); consumidor/productor AMQP | BD Transaccional (SQL), Almacén de Documentos (S3), Broker (AMQP), API Hacienda (XML/HTTPS) |
| **Procesador Asíncrono (Workers)** | Contenedor (servicio background) | Relay del *outbox* a eventos, reintentos con backoff/circuit breaker, envío de notificaciones y escritura del log de auditoría. Garantiza idempotencia por `event_id`. | .NET Worker Service (BackgroundService) | Consumidor AMQP; procesos programados | Broker (AMQP), BD Transaccional (lee outbox), Almacén de Auditoría (insert-only), Servicio de Correo (SMTP/API) |
| **Identity Provider** | Contenedor (servicio) | Autenticación y autorización multi-tenant: OIDC/OAuth2, emisión y validación de JWT, RBAC por rol y resolución de `tenant_id`. | Keycloak | OIDC / OAuth2 / JWKS sobre HTTPS | BD propia de Keycloak (interna) |
| **Broker de Mensajería** | Contenedor (infraestructura) | Transporte asíncrono de eventos de dominio; desacopla emisión fiscal, notificación y auditoría del hilo de request. Habilita reintentos y orden. | RabbitMQ | AMQP (colas/exchanges) | — |
| **Base de Datos Transaccional** | Contenedor (almacenamiento) | Persistencia transaccional multi-tenant con aislamiento por `tenant_id`: tenants, clientes, cotizaciones, facturas y tabla *outbox*. | SQL Server | SQL (ADO.NET/TLS) | — |
| **Almacén de Auditoría** | Contenedor (almacenamiento) | Log append-only e inmutable de acciones y transacciones fiscales; nadie lo modifica tras escribir (RF-05, invariante 6 de 1.6). | SQL Server (esquema append-only / insert-only) | SQL insert-only | — |
| **Almacén de Documentos Fiscales** | Contenedor (almacenamiento) | Conservación de XML/PDF de comprobantes con integridad demostrable durante ≥5 años (REST-02). | MinIO (compatible S3) | API S3 sobre HTTPS | — |
| **Motor de Automatización** *(externo)* | Sistema externo | Capa de captación social: responde/guía en redes y entrega preventas. No participa del dominio fiscal (REST-05). | Fuera del sistema | — | API de Aplicación (handoff REST) |
| **API Ministerio de Hacienda CR** *(externo)* | Sistema externo | Autoridad fiscal: valida el comprobante y define su estado aceptado/rechazado. | Fuera del sistema | XML sobre HTTPS | — |
| **Servicio de Correo Electrónico** *(externo)* | Sistema externo | Entrega comprobantes y notificaciones al cliente final; nunca es fuente de verdad del estado fiscal. | Fuera del sistema | SMTP / API | — |

---

#### 7.2.4 Vista de componentes (C4 nivel 3)

> **Qué muestra:** El interior de un contenedor de la vista 7.2, descompuesto en sus componentes principales (agrupaciones de código con una responsabilidad clara), las interfaces que exponen/consumen y la tecnología con que se implementan. Es el puente entre la vista de contenedores (7.2.2) y el diseño detallado de clases (sección 10).

> **Notación — C4 nivel 3 (Component Diagram).** Se abren **dos subsistemas**, elegidos por ser los que concentran los escenarios de calidad más exigentes y las fronteras arquitectónicas del sistema: el **Servicio de Facturación Fiscal** (dominio regulado aislado — ADR-001) y el **Procesador Asíncrono** (materialización del outbox y de la idempotencia — ADR-002). Se eligieron estos dos por encima de la API de Aplicación porque son los que cargan con QS-01, QS-02, QS-04 y QS-06; la API de Aplicación es un monolito modular de responsabilidades más convencionales (CRM, cotizaciones) cuyo detalle relevante —el handoff— ya quedó cubierto en la vista de comportamiento (7.3, Flujos 2 y 4).

##### 7.2.4.1 Componentes del Servicio de Facturación Fiscal

![Vista de componentes — Servicio de Facturación Fiscal](../diagramas/c4-componentes-fiscal.png)
*Figura 5 — Vista de componentes (C4 nivel 3) del Servicio de Facturación Fiscal. Código fuente en `/diagramas/c4-componentes-fiscal.mmd`.*

El servicio expone **dos puntos de entrada** que convergen en el mismo orquestador, de modo que la validación, la idempotencia y la máquina de estados se aplican por igual sin importar por dónde entre la solicitud:

| Componente | Rol (B/C/E) | Responsabilidad | Interfaz |
|---|---|---|---|
| `FiscalInvoiceController` | Boundary | Punto de entrada REST interno consumido por la API de Aplicación; traduce HTTP ↔ dominio. | REST/HTTPS interno |
| `FiscalEventConsumer` | Boundary | Consumidor AMQP del evento `emitir_comprobante` relevado desde el outbox; delega en el mismo servicio que el controller. | AMQP (RabbitMQ) |
| `FiscalInvoiceService` | Control | Orquestador: valida, coordina construcción/firma/envío, transiciona estado y encola auditoría de forma transaccional. | — (núcleo) |
| `ComprobanteStateMachine` | Control | Concentra las transiciones válidas del comprobante; evita lógica de estado dispersa. | — |
| `IdempotencyGuard` | Control | Deduplica por `event_id`: un reintento interno no genera un segundo comprobante (QS-06). | — |
| `IXmlComprobanteBuilder` → `XmlComprobanteBuilderV44` | Control | Construye el XML conforme al esquema vigente; **intercambiable** ante cambios de Hacienda (QS-05). | Interfaz de construcción |
| `ISignatureProvider` → `XadesEpesSignatureProvider` | Control | Aplica la firma XAdES-EPES sobre el XML. | Interfaz de firma |
| `IHaciendaClient` → `HaciendaHttpClient` | Boundary | Cliente hacia Hacienda con **reintentos + circuit breaker** (QS-02). | XML/HTTPS externo |
| `IComprobanteRepository` → `ComprobanteSqlRepository` | Entity | Persistencia del estado del comprobante con **concurrencia optimista**. | SQL (ADO.NET/TLS) |
| `IOutboxWriter` → `OutboxWriter` | Entity | Escribe el evento de auditoría en la tabla outbox dentro de la misma transacción (ADR-002). | SQL (misma tx) |

**Consistencia hacia arriba (7.2) y hacia abajo (10).** Cada dependencia externa del diagrama coincide exactamente con las que la Figura 4 asigna al contenedor "Servicio de Facturación Fiscal" (BD Transaccional, Broker, Almacén de Documentos y API de Hacienda): la vista de componentes no introduce ninguna dependencia que no existiera ya a nivel de contenedor. Hacia abajo, estos mismos componentes son los que la sección 10.1 detalla a nivel de clases y contratos (Figura 14).

##### 7.2.4.2 Componentes del Procesador Asíncrono (Workers)

![Vista de componentes — Procesador Asíncrono](../diagramas/c4-componentes-workers.png)
*Figura 6 — Vista de componentes (C4 nivel 3) del Procesador Asíncrono (Workers). Código fuente en `/diagramas/c4-componentes-workers.mmd`.*

Este contenedor tiene dos rutas de trabajo: **relevar** el outbox hacia el broker y **consumir** eventos para notificar y auditar. Ambas comparten la garantía de idempotencia y la política de reintentos.

| Componente | Rol (B/C/E) | Responsabilidad |
|---|---|---|
| `OutboxRelay` | Control | *Poller* que lee el outbox no publicado (claim por lote con bloqueo de fila) y lo relaya al broker. |
| `EventDispatcher` | Control | Enruta cada evento consumido a su handler (competing consumers). |
| `IdempotencyGuard` | Control | Deduplica por `event_id`: exactamente un efecto de negocio por evento (QS-06). |
| `RetryPolicy + CircuitBreaker` | Control | Backoff exponencial y reintento FIFO ante fallos aguas abajo (QS-02); re-encola `emitir_comprobante`. |
| `NotificationDispatcher` | Control | Arma y envía el comprobante/notificación al cliente por el Servicio de Correo. |
| `AuditWriter` | Entity | Escribe la entrada append-only e inmutable en el Almacén de Auditoría (RF-05, QS-04). |
| `OutboxRepository` | Entity | Marca los eventos del outbox como publicados tras confirmarse la publicación. |

**Consistencia con 7.2.** Las dependencias del diagrama —lee el outbox de la BD Transaccional, publica/consume en el Broker, escribe en el Almacén de Auditoría, envía por el Servicio de Correo y re-encola hacia el Servicio Fiscal— son exactamente las que la Figura 4 asigna al contenedor "Procesador Asíncrono", sin agregados ni omisiones. El modelo de concurrencia de estos componentes (claim de outbox, competing consumers, concurrencia optimista) se analiza en detalle en la sección 7.5.

---

### 7.3 Vista de comportamiento
*Hito: Avance 2 (S11) — al menos 2 flujos; Entrega final (S14) — flujos completos*

> **Qué muestra:** Cómo fluye la información y el control a través del sistema para los casos de uso más importantes. Complementa la vista estática de la sección 7.2.
>
> **Notación:** Diagramas de secuencia UML. **Esta sección es obligatoria para todos los tipos de sistema.**
>
> **Instrucciones:** Incluí un diagrama de secuencia por cada flujo crítico. Para el Avance 2, incluí al menos los 2 flujos más importantes. Para la Entrega final, cubrí el camino feliz Y al menos un camino de error o excepción por flujo. Cada diagrama debe tener título, los participantes claramente identificados y las llamadas etiquetadas con el método o mensaje.

Para la Entrega final se documentan **cinco flujos**, cada uno con su camino feliz y al menos un camino de error o excepción. Los dos primeros (introducidos en Avance 2) son los más importantes porque cubren el driver central del dominio (RF-01 — emisión fiscal) y el driver diferenciador del producto (RF-02/RF-06 — captación social con handoff idempotente). Los tres restantes, añadidos en la Entrega final, cierran el comportamiento extremo a extremo: la **recuperación** ante la caída de Hacienda (que en el Flujo 1 solo se dejaba encolada), la **conversión comercial** completa preventa → cotización → factura (que enlaza los Flujos 2 y 1) y la **corrección** de un comprobante rechazado junto con la invariante de inmutabilidad. En conjunto ejercitan todos los contenedores y fronteras de confianza de 7.1 y 7.2.

| # | Flujo | Camino de error cubierto | Escenarios de calidad |
|---|---|---|---|
| 1 | Emisión de factura electrónica | Timeout/5xx de Hacienda → `pendiente_validacion_hacienda` | QS-01, QS-02, QS-04, QS-06 |
| 2 | Handoff de preventa desde el Motor de Automatización | Clave de idempotencia duplicada → 200 sin efecto nuevo | QS-01, QS-03, QS-06 |
| 3 | Recuperación asíncrona tras indisponibilidad de Hacienda | Hacienda sigue caída → circuito abierto, backoff, sin pérdida | QS-02, QS-06 |
| 4 | Conversión de preventa a cotización y factura | Preventa ya convertida (409) / cotización vencida (422) | QS-01, QS-06 |
| 5 | Corrección tras rechazo de Hacienda | Intento de editar un comprobante aceptado → 409 (inmutable) | RF-05, QS-06 |

#### Flujo 1 — Emisión de factura electrónica

![Diagrama de secuencia — Emisión de factura electrónica](../diagramas/secuencia-emision-factura.png)
*Figura 7 — Secuencia: emisión de factura electrónica, camino feliz y degradación ante indisponibilidad de Hacienda. Código fuente en `/diagramas/secuencia-emision-factura.mmd`.*

**Descripción:** El flujo inicia cuando un usuario interno (Asistente Administrativo o Vendedor) solicita emitir una factura desde una cotización aprobada. La SPA envía la solicitud a la API de Aplicación con su JWT; la API valida token, rol y `tenant_id` contra el Identity Provider (ningún dato fiscal se toca sin esa validación — QS-01). La decisión de diseño clave ocurre en el paso 5: la factura (estado `emitida_local`) y el evento outbox `emitir_comprobante` se escriben **en la misma transacción**, de modo que el evento de emisión queda persistido de forma durable *antes* de responder 202 al usuario (patrón *outbox*, QS-04); la emisión fiscal nunca bloquea el hilo de request. El Procesador Asíncrono releva el outbox al Broker y el Servicio de Facturación Fiscal —único contenedor que habla con Hacienda— verifica idempotencia por `event_id` (un reintento interno no genera segunda factura — QS-06), genera el XML, aplica la firma XADES-EPES, archiva el documento en el Almacén de Documentos y lo envía a Hacienda. **Manejo de error:** si Hacienda responde timeout o 5xx, el comprobante pasa al estado fiscal legítimo `pendiente_validacion_hacienda` y entra a la cola de reintentos con backoff y circuit breaker; el usuario solo percibe el cambio de estado (≤ 2 s de degradación) y al restaurarse el servicio la cola se procesa en orden FIFO sin intervención manual ni pérdida de documentos (QS-02). En ambas ramas, el resultado termina en el Almacén de Auditoría (entrada append-only con actor, tenant, timestamp, número de comprobante, estado y hash del XML) y el comprobante se entrega al cliente final por el Servicio de Correo, que nunca es fuente de verdad del estado fiscal.

**Escenarios de calidad que este flujo valida:** QS-01 (autorización por tenant en el punto de entrada), QS-02 (degradación y recuperación ante fallo de Hacienda), QS-04 (auditoría persistida en outbox antes de responder, ≤ 80 ms), QS-06 (idempotencia por `event_id` ante reintentos internos de emisión).

---

#### Flujo 2 — Handoff de preventa desde el Motor de Automatización

![Diagrama de secuencia — Handoff de preventa](../diagramas/secuencia-handoff-preventa.png)
*Figura 8 — Secuencia: handoff idempotente de preventa desde el Motor de Automatización. Código fuente en `/diagramas/secuencia-handoff-preventa.mmd`.*

**Descripción:** El flujo inicia fuera del sistema: un cliente final envía un mensaje con intención de compra por Instagram/WhatsApp o TikTok (canales no confiables), el canal lo entrega por webhook al Motor de Automatización, y este responde y guía al cliente de forma automática dentro de su capa social (QS-03). Cuando detecta intención de compra, el motor ejecuta el **handoff**: un POST al endpoint de preventas de la API de Aplicación, autenticado con credencial de integración propia (OAuth2 client, sin sesión de usuario humano) y acompañado de una `Idempotency-Key`. El Identity Provider emite un token con alcance de integración que **no tiene acceso al dominio fiscal** (REST-05, sección 3.4). La API consulta la clave de idempotencia y decide: si la clave es nueva, crea la preventa, registra la clave y escribe el evento outbox `preventa_recibida` en una única transacción, respondiendo 201; **manejo del caso de error/duplicado:** si la clave ya fue procesada —reejecución de un workflow del motor, reintento por timeout— responde 200 con el resultado previo sin ejecutar ningún efecto nuevo, garantizando exactamente un efecto de negocio por evento (QS-06). El Procesador Asíncrono releva el outbox, publica el evento y escribe la entrada de auditoría append-only con la identidad de la integración (RF-05). El dominio fiscal no avanza en ningún punto de este flujo: la preventa queda visible en la SPA y solo una acción interna de un vendedor la convierte en cotización o factura, lo que garantiza que un fallo del motor no tenga impacto fiscal.

**Escenarios de calidad que este flujo valida:** QS-06 (deduplicación por clave de idempotencia con exactamente un efecto de negocio), QS-03 (la capa social responde fuera del camino crítico del sistema; el handoff es asíncrono respecto de la conversación), QS-01 (credencial de integración autenticada y acotada por alcance), además de los drivers RF-05 (auditoría del actor no humano) y REST-05 (el motor no participa del dominio fiscal).

---

#### Flujo 3 — Recuperación asíncrona tras indisponibilidad de Hacienda

![Diagrama de secuencia — Recuperación tras caída de Hacienda](../diagramas/secuencia-recuperacion-hacienda.png)
*Figura 9 — Secuencia: recuperación asíncrona y reintento FIFO idempotente tras la restauración de Hacienda, con el camino de error de indisponibilidad sostenida. Código fuente en `/diagramas/secuencia-recuperacion-hacienda.mmd`.*

**Descripción:** Este flujo completa el camino de error que el Flujo 1 dejaba abierto. El punto de partida es un conjunto de comprobantes en estado `pendiente_validacion_hacienda`, encolados en orden FIFO tras el timeout o 5xx de Hacienda. El Procesador Asíncrono mantiene un **circuit breaker**: mientras el circuito está abierto no golpea a Hacienda de forma continua, sino que espera con **backoff exponencial y jitter** y lanza sondas espaciadas. **Camino de error (indisponibilidad sostenida):** si la sonda vuelve a fallar, el circuito permanece abierto y los comprobantes **no se pierden** —la cola es durable—, simplemente se difiere el siguiente intento. **Camino de recuperación:** cuando una sonda tiene éxito, el circuito pasa a semiabierto y luego a cerrado, y se drena la cola en orden FIFO. Por cada comprobante, el Servicio Fiscal verifica idempotencia por `event_id` (si ya había sido aceptado en un intento previo no lo reenvía — QS-06), reenvía el XML firmado, actualiza el estado definitivo con concurrencia optimista y encola la auditoría; finalmente el Worker escribe la entrada append-only y notifica al cliente. Todo el proceso ocurre **sin intervención manual y sin comprobantes duplicados**.

**Escenarios de calidad que este flujo valida:** QS-02 (recuperación automática tras la restauración del servicio de Hacienda, sin pérdida de documentos) y QS-06 (idempotencia de los reintentos: exactamente un comprobante por evento aun tras múltiples reenvíos).

---

#### Flujo 4 — Conversión de preventa a cotización y factura

![Diagrama de secuencia — Preventa a factura](../diagramas/secuencia-preventa-a-factura.png)
*Figura 10 — Secuencia: conversión comercial de preventa → cotización → factura, con caminos de error de negocio (preventa ya convertida, cotización vencida). Código fuente en `/diagramas/secuencia-preventa-a-factura.mmd`.*

**Descripción:** Este flujo enlaza el diferenciador del producto (Flujo 2) con el núcleo del dominio (Flujo 1), mostrando cómo una preventa recibida por handoff se transforma en una factura fiscal **solo por acción interna** de un usuario autorizado —nunca por el Motor de Automatización (REST-05)—. El Vendedor abre la preventa y solicita convertirla en cotización; la API valida rol y `tenant_id` contra el Identity Provider y, **en una única transacción**, crea la cotización y marca la preventa como convertida. **Camino de error 1 (preventa ya convertida):** si la preventa ya tiene cotización —doble clic, reintento— la API responde `409 Conflict` sin crear un duplicado. Tras la aprobación del cliente, el Vendedor emite la factura desde la cotización: si está vigente, la API crea la factura (`emitida_local`) y el evento outbox `emitir_comprobante`, respondiendo `202` y **entregando el control al Flujo 1** para la emisión fiscal end-to-end. **Camino de error 2 (cotización inválida):** si la cotización está vencida o ya fue facturada, responde `422 Unprocessable Entity`. Así, la frontera entre lo comercial (reversible, editable) y lo fiscal (regulado, con consecuencia legal) queda explícita en el flujo.

**Escenarios de calidad que este flujo valida:** QS-01 (autorización por rol y tenant en cada transición de estado comercial) y QS-06 (la conversión de una preventa produce exactamente una cotización aunque la acción se reintente).

---

#### Flujo 5 — Corrección tras rechazo de Hacienda e invariante de inmutabilidad

![Diagrama de secuencia — Corrección tras rechazo](../diagramas/secuencia-correccion-rechazo.png)
*Figura 11 — Secuencia: corrección mediante nota de crédito/débito tras un rechazo de Hacienda, y bloqueo del intento de modificar un comprobante aceptado. Código fuente en `/diagramas/secuencia-correccion-rechazo.mmd`.*

**Descripción:** Este flujo cierra el ciclo de vida del comprobante (ver la máquina de estados de la Figura 1) para el caso de rechazo. El Asistente Administrativo revisa un comprobante en estado `Rechazado` y su código de motivo. **Camino de corrección válido:** inicia una corrección que **no modifica el documento original** —irreversible por diseño— sino que emite una **nota de crédito/débito** que lo referencia; esta nota se crea con su propio `event_id` en una transacción con outbox y reutiliza el Flujo 1 para su emisión y validación ante Hacienda. **Camino de error (invariante de inmutabilidad):** si un usuario intenta editar o eliminar una factura ya `Aceptada`, la API valida la invariante de la sección 1.6 y responde `409 Conflict` explicando que un comprobante aceptado es irreversible y solo se corrige con una nota; además, **el intento rechazado también se audita** (append-only, RF-05), de modo que queda trazado incluso lo que no se permitió ejecutar.

**Escenarios de calidad que este flujo valida:** RF-05 (auditoría de acciones, incluidos los intentos bloqueados) y QS-06 (la nota de corrección se emite exactamente una vez por evento), apoyándose en la invariante de inmutabilidad del comprobante aceptado (sección 1.6).

---

### 7.4 Vista de despliegue
*Hito: Entrega final (S14)*

> **Qué muestra:** Dónde y cómo se despliega el sistema físicamente — servidores, contenedores Docker, servicios cloud, dispositivos edge, bases de datos, balanceadores, etc.
>
> **Notación:** C4 Deployment Diagram o diagrama de despliegue UML. Obligatorio si el sistema tiene componentes distribuidos en múltiples nodos. Opcional pero recomendado para sistemas monolíticos desplegados en cloud.
>
> **Instrucciones:** Mostrá los nodos de infraestructura, qué artefactos de software corren en cada nodo, y las conexiones de red entre ellos con el protocolo indicado. Si usás servicios cloud, nombralos específicamente (ej. AWS RDS, Google Cloud Run, Azure Service Bus).

#### 7.4.1 Estrategia de despliegue y su justificación

La topología se elige en coherencia con el estilo arquitectónico (sección 8: *service-based* con núcleo de monolito modular) y, sobre todo, con la restricción **REST-06** (equipo de 3 personas, presupuesto limitado). Por eso el sistema se despliega como **contenedores Docker orquestados con Docker Compose sobre dos VMs cloud** (una de aplicación y una de datos), y **no** sobre Kubernetes ni sobre un stack de PaaS gestionado: Kubernetes excede la capacidad operativa del equipo (mismo argumento que descarta microservicios en la sección 8.2) y el PaaS gestionado introduce costo recurrente que el presupuesto no soporta. Docker Compose es la representación más honesta porque los "contenedores" de la vista 7.2 (que en C4 son unidades lógicas) se materializan aquí como contenedores Docker reales, uno por unidad desplegable, con un mapeo casi 1:1.

La separación en **dos nodos** no es cosmética: aísla el plano de cómputo (servicios sin estado, reiniciables y escalables verticalmente) del plano de datos (con estado y volúmenes persistentes), de modo que un reinicio o redeploy de la VM de Aplicación no arriesga la integridad de la BD transaccional, la auditoría append-only ni los documentos fiscales que deben conservarse ≥ 5 años (REST-02).

#### 7.4.2 Diagrama

![Vista de despliegue](../diagramas/despliegue.png)
*Figura 12 — Vista de despliegue (VM cloud + Docker Compose) del sistema SmartBilling Connect. Código fuente en `/diagramas/despliegue.mmd`.*

#### 7.4.3 Descripción de nodos

| Nodo | Descripción | Artefactos desplegados | Conectividad |
|---|---|---|---|
| **VM de Aplicación** | VM Linux (≈2 vCPU / 4 GB) con Docker Engine. Aloja los servicios sin estado y la puerta de entrada. Escala verticalmente; los servicios pueden replicarse con más réplicas de contenedor. | `nginx` (reverse proxy + terminación TLS), SPA (build estático servido por nginx), API de Aplicación (ASP.NET Core 8), Servicio de Facturación Fiscal (ASP.NET Core 8), Procesador Asíncrono (.NET Worker Service), Keycloak, RabbitMQ (con volumen durable) | Entrante: HTTPS/443 desde navegadores y desde el Motor de Automatización (handoff). Interno: HTTP a SPA/API, OIDC/JWKS a Keycloak, AMQP/5672 a RabbitMQ. Hacia VM de Datos: TDS/1433 (TLS) y S3/9000 (HTTPS). Saliente: HTTPS/443 a Hacienda, SMTP/587 al correo. |
| **VM de Datos** | VM Linux (≈2 vCPU / 8 GB) con Docker Engine y **volúmenes persistentes**. Aloja todo el estado del sistema. Se respalda de forma independiente. | SQL Server (Express/Developer) con la BD transaccional, el esquema de auditoría append-only y la BD interna de Keycloak; MinIO (almacén compatible S3, retención ≥ 5 años) | Entrante: TDS/1433 (TLS) desde API, Servicio Fiscal, Workers y Keycloak; API S3/9000 (HTTPS) desde el Servicio Fiscal. Sin exposición a Internet: solo accesible desde la VM de Aplicación (red privada). |
| **Dispositivos de usuario** *(fuera del sistema)* | Navegadores de los usuarios internos (Dueño de PYME, Vendedor, Asistente). | SPA cargada en el navegador | HTTPS/443 hacia el `nginx` de la VM de Aplicación. |
| **API Ministerio de Hacienda CR** *(externo)* | Autoridad fiscal. | — | Recibe XML firmado sobre HTTPS/443 desde el Servicio Fiscal. |
| **Motor de Automatización (+ Meta/TikTok)** *(externo)* | Capa de captación social. | — | HTTPS/443 hacia la API (handoff de preventa, autenticado e idempotente). |
| **Servicio de Correo (SMTP)** *(externo)* | Entrega de comprobantes y notificaciones. | — | SMTP/587 (TLS) desde el Procesador Asíncrono. |

**Consideraciones de despliegue relacionadas con los drivers.** (a) *Seguridad (QA-01):* solo la VM de Aplicación tiene interfaz pública; la VM de Datos vive en red privada y nunca se expone a Internet, y todo tráfico externo pasa por la terminación TLS de nginx. (b) *Disponibilidad (QA-02):* RabbitMQ usa un volumen durable para que los eventos encolados sobrevivan a un reinicio del contenedor (medida de QS-02), y el plano de datos, al estar aislado, no se ve afectado por redeploys del plano de cómputo. (c) *Costo (REST-06):* todo el stack corre sobre software gratuito/OSS y dos VMs modestas, sin servicios gestionados de pago. (d) *Evolución:* si en el futuro un servicio (p. ej. el Servicio Fiscal) necesitara escalar de forma independiente, este esquema permite moverlo a su propia VM o a más réplicas sin rediseñar la arquitectura lógica —los contenedores ya están aislados y se comunican por protocolos explícitos—.

---

### 7.5 Vista de concurrencia *(sección opcional)*
*Hito: Entrega final (S14) — obligatoria si el sistema maneja concurrencia*

> **Cuándo incluirla:** Si tu sistema tiene múltiples procesos o hilos ejecutándose simultáneamente, maneja eventos asincrónicos, tiene condiciones de carrera posibles, o requiere sincronización entre componentes. Si el sistema es completamente secuencial y single-threaded, omití esta sección y justificá por qué no aplica.
>
> **Notación:** Diagrama de estado UML o diagrama de actividad UML con swimlanes.
>
> **Instrucciones:** Describí el modelo de concurrencia del sistema: qué procesos o hilos existen, cómo se sincronizan, qué recursos comparten, y cómo se evitan condiciones de carrera o deadlocks. Referenciá los escenarios de calidad de la sección 4 que este modelo satisface.

**¿Aplica esta sección? Sí.** SmartBilling Connect es un sistema concurrente por diseño: el Procesador Asíncrono corre como uno o más workers (`BackgroundService`) que compiten por procesar el outbox y los eventos del broker en paralelo con los hilos de request de la API; varios usuarios de un mismo tenant pueden operar simultáneamente sobre los mismos comprobantes; y el sistema recibe eventos externos potencialmente duplicados (RF-06). Existen, por tanto, recursos compartidos (tabla outbox, filas de `Comprobante`, colas del broker) y posibles condiciones de carrera. Omitir esta sección sería deshonesto; el modelo de concurrencia es precisamente lo que hace correctos a los patrones outbox e idempotencia.

#### 7.5.1 Diagrama del modelo de concurrencia

![Modelo de concurrencia](../diagramas/concurrencia-outbox.png)
*Figura 13 — Modelo de concurrencia: relay de outbox, competing consumers y control de concurrencia (actividad con swimlanes). Código fuente en `/diagramas/concurrencia-outbox.mmd`. La máquina de estados del comprobante que estos procesos hacen avanzar está en la Figura 1.*

#### 7.5.2 Procesos, recursos compartidos y sincronización

| Aspecto | Cómo se resuelve | Driver / escenario |
|---|---|---|
| **Hilos/procesos concurrentes** | Hilos de request de la API y del Servicio Fiscal + N workers del Procesador Asíncrono (competing consumers) + consumidores del broker. Cada unidad es sin estado; el estado vive en la BD y el broker. | QA-04, REST-06 |
| **Publicación fiable de eventos** | Patrón *Transactional Outbox* (ADR-002): el cambio de negocio y el evento se escriben en **una sola transacción**; ningún evento se publica si la transacción no commitea, y ninguno se pierde si el proceso cae tras el commit. | QS-04, QS-02 |
| **Relay concurrente del outbox** | Los workers reclaman filas por lote con **bloqueo de fila** (`UPDATE TOP(k) … WITH (UPDLOCK, READPAST)`): dos workers nunca toman la misma fila y un worker no espera por filas ya reclamadas por otro (sin contención ni doble publicación). | QS-06 |
| **Entrega a consumidores** | RabbitMQ con **colas durables** y *competing consumers* con *ack* manual y *prefetch*: cada mensaje lo procesa exactamente un consumidor; si este falla antes del *ack*, el mensaje se re-entrega. | QS-02, QS-06 |
| **Idempotencia de efectos** | `IdempotencyGuard` deduplica por `event_id` / `Idempotency-Key` antes de aplicar cualquier efecto de negocio: reentregas y duplicados producen **exactamente un** comprobante, notificación o entrada de auditoría. | QS-06, RF-06 |
| **Actualización concurrente de comprobantes** | **Concurrencia optimista** (columna `rowversion`) sobre `Comprobante`: si dos flujos intentan transicionar el mismo comprobante, el segundo detecta el conflicto de versión y reintenta sobre el estado fresco, evitando transiciones inconsistentes. | QS-01, integridad fiscal |
| **Orden y reintentos ante fallos** | `RetryPolicy + CircuitBreaker` con backoff exponencial y reproceso **FIFO**; ante fallo aguas abajo se hace *nack*/re-encola sin romper el orden ni perder el mensaje. | QS-02 |
| **Prevención de deadlocks** | Las transacciones son cortas y de alcance mínimo (escribir dominio + outbox y commitear); el trabajo lento (Hacienda, correo, S3) ocurre **fuera** de la transacción, en los consumidores. Al no anidar transacciones largas ni tomar múltiples bloqueos en distinto orden, no hay ciclos de espera. | QA-04 |

**Cómo se evitan las condiciones de carrera clave.** (1) *Doble emisión:* aunque dos workers relayen o dos eventos duplicados lleguen, la deduplicación por `event_id` más la máquina de estados garantizan un solo comprobante (QS-06). (2) *Doble toma de outbox:* el bloqueo de fila con `READPAST` reparte el trabajo sin solapamiento. (3) *Transición de estado inconsistente:* la concurrencia optimista con `rowversion` hace que solo una transición gane y la otra reintente. Este modelo satisface QS-02 (recuperación sin pérdida), QS-04 (auditoría sin penalizar el camino crítico) y QS-06 (exactamente un efecto por evento).

---

### 7.6 Evolución del diseño entre avances

> Esta subsección documenta cómo evolucionaron las vistas arquitectónicas entre los tres hitos (Propuesta S03 → Avance 1 S07 → Avance 2 S11 → Entrega final S14), qué se mantuvo estable y por qué, y qué se agregó en cada paso. Complementa el *Historial de versiones* del encabezado con el detalle de las decisiones de diseño.

#### 7.6.1 Trazabilidad de vistas por hito

| Vista | Propuesta (0.1) | Avance 1 (0.2) | Avance 2 (0.3) | Entrega final (1.0) |
|---|---|---|---|---|
| 7.1 Contexto | Idea inicial del sistema | **Creada** (C4 nivel 1 + fronteras de confianza) | Sin cambios estructurales | Refinada: explícito que Meta/TikTok solo llegan vía el Motor |
| 7.2 Contenedores | — | — | **Creada** (C4 nivel 2, 9 contenedores + stack) | Consolidada su consistencia con contexto y componentes |
| 7.2.4 Componentes | — | — | — | **Creada** (C4 nivel 3: Servicio Fiscal y Procesador Asíncrono) |
| 7.3 Comportamiento | — | — | **Creada** (2 flujos: emisión y handoff) | Completada a **5 flujos** con caminos de error |
| 7.4 Despliegue | — | — | — | **Creada** (VM cloud + Docker Compose, 2 nodos) |
| 7.5 Concurrencia | — | — | — | **Creada** (outbox, competing consumers, concurrencia optimista) |

#### 7.6.2 Decisiones que cambiaron o se refinaron

| Cambio | De → A | Motivación |
|---|---|---|
| Frontera del Motor de Automatización | Implícita (Avance 1) → explícita y repetida en todas las vistas (7.1, 7.2, 7.3, ciclo de vida) | Evitar que se leyera al Motor como participante fiscal; endurece REST-05 y la frontera de la sección 3.4. |
| Aislamiento del dominio fiscal | Módulo dentro del sistema (Propuesta) → **contenedor propio** (Avance 2, ADR-001) → **componentes detallados** (Entrega final, 7.2.4) | Convertir una convención en una barrera arquitectónica verificable (QA-01, QS-05). |
| Comunicación entre partes | Llamadas directas (implícito) → **outbox + broker asíncrono** (Avance 2, ADR-002) → **modelo de concurrencia formalizado** (Entrega final, 7.5) | Resolver la tensión seguridad/rendimiento (QS-01/QS-04) y la idempotencia (RF-06/QS-06) sin bloquear el request. |
| Comportamiento cubierto | 2 flujos felices + degradación (Avance 2) → **5 flujos con recuperación, conversión y corrección** (Entrega final) | Cubrir el camino de error extremo a extremo, no solo dejarlo encolado. |
| Despliegue | No definido → **Docker Compose sobre 2 VMs** (Entrega final) | Coherencia con REST-06: la opción operable por 3 personas, descartando K8s y PaaS de pago. |

#### 7.6.3 Qué se mantuvo estable (y por qué es una buena señal)

Lo más importante de la evolución es lo que **no** cambió: los actores y sistemas externos de la vista de contexto (7.1) son los mismos desde Avance 1, y los nueve contenedores de Avance 2 se conservan sin altas ni bajas en la Entrega final. Las vistas nuevas (componentes, despliegue, concurrencia) **descomponen** lo ya definido en lugar de contradecirlo: cada dependencia de la vista de componentes existe ya a nivel de contenedor, y cada nodo de despliegue aloja exactamente los contenedores de 7.2. Esa estabilidad —agregar detalle sin reabrir decisiones— es evidencia de que los drivers de la sección 3 fueron suficientes para fijar la arquitectura temprano, en línea con los principios KISS/YAGNI de la sección 6.

---

# BLOQUE 4 — DECISIONES ARQUITECTÓNICAS
*Hito: Avance 2 (S11)*

---

## 8. Estilo arquitectónico

> El estilo arquitectónico de SmartBilling Connect responde a una tensión central identificada en los drivers de la sección 3: el sistema exige fronteras fuertes entre el dominio comercial, el dominio fiscal (con consecuencia legal directa), el procesamiento asíncrono y la identidad/acceso (RF-01 a RF-06, QA-05), e interoperabilidad con al menos tres ecosistemas externos (Hacienda, los canales sociales vía el motor de automatización y el servicio de correo) — pero debe ser construido y operado por un equipo de 3 personas con presupuesto limitado (REST-06). La selección del estilo prioriza satisfacer los atributos de calidad QA-01 a QA-05 dentro de esas restricciones, y se evalúa contra dos alternativas que fueron descartadas con sus respectivos trade-offs. El estilo adoptado es el que materializa la vista de contenedores de la sección 7.2.

### 8.1 Estilo(s) adoptado(s)

| Estilo | Aplicación en el sistema | Justificación |
|---|---|---|
| Arquitectura basada en servicios (*service-based*) con núcleo de monolito modular | Estructura general del sistema, materializada en la vista de contenedores (sección 7.2). El dominio comercial completo (CRM, cotizaciones, preventas, productos y orquestación del flujo) se concentra en la **API de Aplicación**, un monolito modular con módulos de fronteras explícitas. Solo se separan como unidades desplegables propias los servicios que un driver justifica: el **Servicio de Facturación Fiscal** (aislamiento del dominio regulado — ADR-001, REST-05, QS-05), el **Procesador Asíncrono** (desacople del camino crítico — ADR-002) y el **Identity Provider** (Keycloak — RF-04, REST-03). Es el punto medio entre monolito y microservicios: pocos servicios de grano grueso, una única base de datos transaccional compartida (con tabla outbox) y sin proliferación de infraestructura distribuida. | Responde a la tensión entre subsistemas desacoplados (RF-01 a RF-05, QA-05) y equipo reducido (REST-06). Un monolito puro no fuerza la frontera fiscal que exigen REST-05 y QS-05 — quedaría como convención, no como barrera arquitectónica (ver alternativas de ADR-001) —, mientras que microservicios completos exceden la capacidad operativa del equipo (sección 8.2). Con cuatro servicios de grano grueso, el pipeline CI/CD y el monitoreo siguen siendo operables por 3 personas, y el dominio fiscal puede endurecerse (QA-01) y desplegarse (QS-05) de forma independiente del resto. |
| Comunicación basada en eventos con broker y outbox transaccional (complementario) | Comunicación asíncrona entre contenedores: la API de Aplicación y el Servicio de Facturación Fiscal escriben los eventos de dominio en la tabla *outbox* dentro de la misma transacción que el cambio de estado; el Procesador Asíncrono los releva a RabbitMQ y desde ahí se procesan la emisión fiscal, las notificaciones y el log de auditoría, con reintentos, orden garantizado y deduplicación por `event_id` (ADR-002). | Responde a QA-02 (Disponibilidad) y QA-04 (Rendimiento): un fallo o lentitud de Hacienda no bloquea el hilo de request (QS-02), y la auditoría obligatoria no penaliza el camino crítico — solo la escritura en outbox, ≤ 80 ms (QS-01, QS-04). El broker durable garantiza que ningún evento encolado se pierda ante reinicios (medida de QS-02) y habilita la idempotencia de RF-06/QS-06. |

### 8.2 Alternativas consideradas y rechazadas

| Alternativa | Por qué se consideró | Por qué se rechazó |
|---|---|---|
| Microservicios (descomposición fina) | SmartBilling Connect tiene subsistemas con fronteras claras (dominio comercial, facturación fiscal, identidad, auditoría, notificaciones). Cada uno podría desplegarse como servicio independiente con su propia base de datos, permitiendo escalamiento y despliegue totalmente independientes. | Complejidad operacional vs. equipo (REST-06): descomponer en 8-10 servicios finos con base de datos por servicio exigiría API gateway, service discovery, distributed tracing y orquestación de contenedores (Kubernetes), inoperable para 3 personas y por encima del presupuesto. Consistencia de datos fiscales (QA-01, REST-01): la emisión exige que la factura y su evento de auditoría se confirmen en una única transacción local (ADR-002); con los datos repartidos entre servicios eso obligaría a sagas o two-phase commit. Escala prematura: los volúmenes iniciales (cientos de facturas/día) no justifican la distribución fina. El estilo adoptado ya captura el beneficio clave — aislar el dominio fiscal — sin ese costo. |
| Monolito en capas (N-Tier) desplegado como unidad única | Es el patrón más conocido, simple de implementar y con baja curva de aprendizaje. Consistente con KISS y la restricción de equipo (REST-06): un solo artefacto, un pipeline, un proceso a monitorear. | Sin frontera física del dominio fiscal: la frontera de REST-05 y el despliegue independiente del módulo fiscal que exige QS-05 (≤ 10 días hábiles, cero subsistemas ajenos modificados) quedarían como convención de código, no como algo forzado por el diseño (ADR-001). Sin fronteras de dominio (QA-05): en capas horizontales, la lógica fiscal y la comercial coexisten en la misma capa de negocio sin separación. Solo comunicación síncrona: el procesamiento asíncrono con outbox, broker y workers que exigen QS-02 y QS-04 no tiene un lugar natural en un modelo request/response en capas. |

### 8.3 Análisis de trade-offs del estilo elegido

> Todo estilo arquitectónico tiene compromisos. Los siguientes trade-offs son específicos de SmartBilling Connect y su contexto de drivers.

| Trade-off | Qué se gana | Qué se sacrifica | Escenario afectado |
|---|---|---|---|
| Pocos servicios de grano grueso vs. escalamiento fino por módulo | Cuatro unidades de ejecución operables por 3 personas (REST-06): pipeline CI/CD y monitoreo acotados. El Servicio de Facturación Fiscal y los Workers escalan y se despliegan de forma independiente del núcleo (QS-05). | Dentro de la API de Aplicación, CRM, cotizaciones y preventas escalan juntos: un pico de preventas obliga a replicar el núcleo completo, no solo ese módulo. | QA-02 (Disponibilidad) y QA-04 (Rendimiento) — se mitiga con réplicas del núcleo detrás de un balanceador; los picos de mensajería social los absorbe el motor de automatización (externo) y el handoff llega ya filtrado como preventas. |
| Consistencia transaccional local vs. visibilidad inmediata en el log final | La factura y su evento de auditoría/outbox se confirman en una única transacción de BD: nunca existe un comprobante sin su evento durable (RF-05, QS-04), sin la complejidad de sagas. | El log append-only final es eventualmente consistente: existe una ventana de segundos entre el commit transaccional y la entrada visible en el Almacén de Auditoría. | QS-04 (Trazabilidad) tensiona con QA-04 (Rendimiento) — resuelto con el outbox (ADR-002); la ventana queda trazada con los timestamps de encolado y de confirmación, por lo que no se confunde con falta de auditoría. |
| Fronteras lógicas dentro del núcleo vs. fronteras físicas | Los módulos del núcleo (API de Aplicación) comparten proceso y transacciones locales: sin overhead de serialización ni latencia de red entre CRM, cotizaciones y preventas. | Las fronteras entre módulos del núcleo son convenciones de equipo, no barreras del compilador ni de la red. Un desarrollador puede acceder directamente a tablas de otro módulo, violando el encapsulamiento. | QA-05 (Modificabilidad) — se mitiga con revisión de código, tests de dependencias entre módulos y convenciones de estructura de carpetas. El módulo de mayor riesgo — el fiscal — ya está extraído como servicio propio (ADR-001), donde la frontera sí es física. |
| Broker externo (RabbitMQ) vs. bus de eventos in-process | Eventos durables ante caídas y reinicios, reintentos con backoff, dead-letter queues y visibilidad operativa de las colas. Sin esto, la medida de QS-02 ("ningún documento encolado se pierde ante reinicios") no es demostrable. | Una pieza más de infraestructura que instalar, monitorear y actualizar (tensión con REST-06), y latencia adicional frente a eventos en memoria. | QS-02 (Disponibilidad) y QS-06 (Idempotencia) se favorecen sobre la simplicidad operativa — se mitiga porque RabbitMQ es open-source (REST-06) y se opera como un único contenedor con configuración estándar. |
---

## 9. Registro de decisiones — ADRs

---

### ADR-001 — Aislamiento del dominio fiscal en un servicio dedicado (Servicio de Facturación Fiscal)

> Documento completo: [`/decisiones/ADR-001-aislamiento-dominio-fiscal.md`](../decisiones/ADR-001-aislamiento-dominio-fiscal.md)

| Campo | Detalle |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-07-04 |
| **Autores** | Edgar Jacob, Brandon Garita, Alejandro Mora |
| **Drivers / escenarios que responde** | REST-05, QA-01, RF-01, QS-05 |

---

### ADR-002 — Patrón Transactional Outbox con procesamiento asíncrono para auditoría e idempotencia

> Documento completo: [`/decisiones/ADR-002-outbox-transaccional-asincrono.md`](../decisiones/ADR-002-outbox-transaccional-asincrono.md)

| Campo | Detalle |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-07-04 |
| **Autores** | Edgar Jacob, Brandon Garita, Alejandro Mora |
| **Drivers / escenarios que responde** | RF-05, RF-06, QS-01, QS-03, QS-04, QS-06 |

---

### ADR-003 — Aislamiento multi-tenant mediante `tenant_id` centralizado en la capa de autorización

> Documento completo: [`/decisiones/ADR-003-estrategia-multi-tenancy.md`](../decisiones/ADR-003-estrategia-multi-tenancy.md)

| Campo | Detalle |
|---|---|
| **Estado** | Aceptada |
| **Fecha** | 2026-07-04 |
| **Autores** | Edgar Jacob, Brandon Garita, Alejandro Mora |
| **Drivers / escenarios que responde** | REST-03, REST-06, QS-01, QA-01 |

---




# BLOQUE 5 — DISEÑO DETALLADO
*Hito: Avance 2 (S11) — primer componente / Entrega final (S14) — componentes restantes*

## 10. Diseño detallado de componentes

### 10.1 Componente 1 — Servicio de Facturación Fiscal

**Responsabilidad:** Genera el XML del comprobante electrónico, lo firma digitalmente (XADES-EPES), gestiona su máquina de estados y coordina el envío y la consulta de estado ante el Ministerio de Hacienda. Lo elegimos como el componente más crítico porque ahí vive toda la lógica que tiene consecuencia legal directa (RF-01). Además es la pieza que quedó aislada a propósito frente al motor de automatización (REST-05, ADR-001) y la que carga con los escenarios de calidad más exigentes del proyecto: QS-01, QS-02, QS-04, QS-05 y QS-06.

**Trazabilidad:** Sección 1.4, casos de uso *"Emitir facturas electrónicas, validar información fiscal de clientes, reenviar comprobantes electrónicos, consultar estados tributarios, corregir errores operativos de facturación"* (Asistente administrativo) y *"convertir cotizaciones en facturas electrónicas"* (Vendedor / Ejecutivo comercial). Ambos apuntan al elemento **Servicio de Facturación Fiscal** descrito en la vista de estructura interna (sección 7.2.3).

#### 10.1.1 Diagrama de clases de diseño

![Diagrama de clases — Servicio de Facturación Fiscal](../diagramas/clases-servicio-facturacion-fiscal.png)
*Figura 14 — Diagrama de clases de diseño: Servicio de Facturación Fiscal. Código fuente en `/diagramas/clases-servicio-facturacion-fiscal.mmd`.*

Separamos el punto de entrada (`FiscalInvoiceController`, boundary) de la orquestación (`FiscalInvoiceService`, control) y de los detalles de infraestructura, cada uno detrás de su propia interfaz: `IXmlComprobanteBuilder` para construir el XML (intercambiable según la versión del esquema, pensando en QS-05), `ISignatureProvider` para la firma digital, `IHaciendaClient` para hablar con Hacienda (con su política de reintentos y circuit breaker) y `IComprobanteRepository` / `IOutboxWriter` para la parte de persistencia y outbox que se explica en ADR-002. `ComprobanteStateMachine` concentra las transiciones válidas del comprobante (Generado → Firmado → Enviado → Aceptado / Rechazado / PendienteValidacionHacienda) para que esa lógica no termine repartida por todo el servicio. Gracias a esta separación por interfaces, si Hacienda cambia el esquema el año que viene, en principio bastaría con reemplazar `XmlComprobanteBuilderV44` sin tocar `FiscalInvoiceService` ni el resto — que es más o menos lo que promete QS-05.

---

#### 10.1.2 Contratos de interfaz

---

| Método / Endpoint | Precondición | Postcondición | Excepciones |
|---|---|---|---|
| `Task<ComprobanteResult> FiscalInvoiceService.EmitirComprobanteAsync(ComprobanteRequest request, CancellationToken ct)` | `request.TenantId` corresponde a un tenant activo y resuelto por la capa de autorización (ADR-003); `request.Lineas` contiene al menos una línea con montos válidos (> 0). | El comprobante queda persistido con estado `Firmado` o `PendienteValidacionHacienda`; su evento de auditoría queda encolado de forma transaccional en el outbox (ADR-002) antes de retornar. | `ValidationException` si `request` no cumple las reglas de negocio mínimas; `TenantMismatchException` si el `TenantId` del request no coincide con el del contexto de autorización; `FiscalSignatureException` si la firma digital falla por certificado inválido o vencido. |
| `Task<ComprobanteStatusDto> FiscalInvoiceService.ConsultarEstadoAsync(Guid comprobanteId, Guid tenantId)` | `comprobanteId` corresponde a un comprobante existente perteneciente a `tenantId`. | Retorna el estado actual del comprobante sin exponer datos de otro tenant. | `NotFoundException` si el comprobante no existe para ese `tenantId` (nunca revela si existe para otro tenant — QS-01). |
| `string IXmlComprobanteBuilder.ConstruirXml(ComprobanteRequest request)` | `request` ya fue validado por `FiscalInvoiceService.ValidarRequest`. | Retorna un XML bien formado y conforme al esquema vigente del Ministerio de Hacienda. | `SchemaValidationException` si el XML resultante no valida contra el esquema XSD vigente. |
| `byte[] ISignatureProvider.Firmar(string xmlContent, X509Certificate2 certificado)` | `certificado` es válido, no ha expirado y corresponde al tenant emisor. | Retorna el XML firmado digitalmente conforme al estándar XADES-EPES, listo para envío. | `CertificateExpiredException`; `InvalidCertificateException` si el certificado no corresponde al emisor declarado. |
| `Task<HaciendaResponse> IHaciendaClient.EnviarComprobanteAsync(byte[] xmlFirmado, CancellationToken ct)` | `xmlFirmado` fue producido por un `ISignatureProvider` válido. | Retorna la respuesta de Hacienda (aceptado/rechazado) o agota la política de reintentos y propaga el fallo. | `TimeoutException` / `HttpRequestException` ante indisponibilidad de Hacienda (capturadas explícitamente para activar el camino `PendienteValidacionHacienda`, QS-02). |
| `ComprobanteEstado ComprobanteStateMachine.Transicionar(ComprobanteEstado actual, ComprobanteEvento evento)` | La transición solicitada (`actual` + `evento`) existe en la tabla de transiciones válidas. | Retorna el nuevo estado válido; nunca dos actores concurrentes pueden aplicar transiciones inconsistentes sobre el mismo comprobante (control de concurrencia optimista en el repositorio). | `InvalidStateTransitionException` si la transición solicitada no es válida desde el estado actual. |

#### 10.1.3 Análisis de robustez

| Objeto | Tipo (Boundary / Control / Entity) | Responsabilidad |
|---|---|---|
| `FiscalInvoiceController` | Boundary | Punto de entrada REST interno consumido únicamente por la API de Aplicación; traduce el request HTTP a la llamada del servicio y el resultado a una respuesta HTTP. |
| `ComprobanteRequest` / `ComprobanteResult` | Boundary (DTO) | Estructuras de datos que cruzan la frontera del componente hacia la API de Aplicación. |
| `HaciendaHttpClient` | Boundary | Interfaz hacia el sistema externo (API del Ministerio de Hacienda); traduce llamadas del dominio a HTTP/XML y viceversa. |
| `FiscalInvoiceService` | Control | Orquesta el flujo completo de emisión: valida, construye XML, firma, transiciona estado, persiste y encola auditoría. |
| `ComprobanteStateMachine` | Control | Aplica las reglas de negocio que determinan qué transiciones de estado son válidas. |
| `XmlComprobanteBuilderV44` | Control | Transforma los datos de la solicitud en el XML fiscal conforme al esquema vigente. |
| `XadesEpesSignatureProvider` | Control | Aplica la operación criptográfica de firma digital sobre el XML construido. |
| `Comprobante` | Entity | Representa el dato persistente central del dominio fiscal (estado, XML, hash de integridad, clave numérica). |
| `ComprobanteSqlRepository` | Entity | Objeto de acceso a datos que lee y escribe el estado persistente de `Comprobante`. |
| `OutboxWriter` | Entity | Objeto de acceso a datos que persiste el evento de auditoría en la tabla outbox dentro de la misma transacción (ADR-002). |

#### 10.1.4 Diagrama de secuencia — flujo principal

![Secuencia — Emisión de comprobante fiscal](../diagramas/secuencia-emision-comprobante-fiscal.png)
*Figura 15 — Secuencia: emisión de comprobante fiscal, camino feliz y camino de error (timeout de Hacienda). Código fuente en `/diagramas/secuencia-emision-comprobante-fiscal.mmd`.*

**Descripción:** La API de Aplicación llama a `FiscalInvoiceController`, que delega en `FiscalInvoiceService`. Este diagrama detalla el interior del componente; en el flujo extremo a extremo de la Figura 7, la solicitud de emisión llega al servicio como evento AMQP (`emitir_comprobante`) relevado desde el outbox — ese consumidor AMQP delega en el mismo `FiscalInvoiceService` que el endpoint REST interno mostrado aquí, por lo que ambas entradas comparten idéntica validación, idempotencia y máquina de estados. El servicio valida la solicitud, construye el XML, lo firma, pasa el comprobante a estado `Firmado` y lo guarda junto con su evento de auditoría en la misma transacción (el patrón outbox de ADR-002), devolviendo `202 Accepted` sin esperar a Hacienda. Por otro lado, el servicio envía el comprobante a Hacienda: si todo sale bien, Hacienda responde a tiempo y el comprobante pasa a `Aceptado`. Si Hacienda no responde o falla (timeout o HTTP 5xx sostenido, el caso que cubre QS-02), el comprobante pasa a `PendienteValidacionHacienda` y queda en cola para reintentarse en orden FIFO, sin que el usuario note más de los 2 segundos de degradación que permite ese mismo escenario.

**Escenarios de calidad que este flujo valida:** QS-01 (autorización por tenant antes de cualquier operación), QS-02 (degradación controlada si Hacienda falla), QS-04 (auditoría transaccional vía outbox), QS-05 (el módulo de XML queda aislado detrás de una interfaz) y QS-06 (idempotencia de los reintentos).

---

### 10.2 Componente 2 — [Nombre del componente] *(Entrega final — S14)*

**Responsabilidad:** [Una oración que describe qué hace este componente y por qué es crítico para el sistema]

**Trazabilidad:** [Referencia a los casos de uso de la sección 1.4 que este componente soporta] → [Referencia al elemento en la vista de estructura interna, sección 7.2]

#### 10.2.1 Diagrama de clases de diseño

> **Instrucciones:** Este no es un diagrama de clases de análisis ni un modelo de dominio. Es el diseño: incluí métodos con firmas completas (nombre, parámetros, tipo de retorno), modificadores de acceso, relaciones de dependencia reales y las interfaces que el componente expone y consume. Mostrá cómo se aplican los patrones de diseño (sección 11) dentro de este componente.

![Diagrama de clases — Componente 2](../diagramas/clases-componente2.png)
*Figura N — Diagrama de clases de diseño: [Nombre del componente]*

#### 10.2.2 Contratos de interfaz

> **Instrucciones:** Para cada método o endpoint público del componente, documentá su contrato formal. Un contrato no es solo la firma — es la especificación de qué garantiza el método y qué exige de quien lo llama.

| Método / Endpoint | Precondición | Postcondición | Excepciones |
|---|---|---|---|
| `[firma del método]` | [Qué debe ser verdad antes de llamarlo] | [Qué garantiza que será verdad después] | [Qué errores puede lanzar y bajo qué condición] |

#### 10.2.3 Análisis de robustez

> **Instrucciones:** Usá el análisis de robustez para verificar que el diseño del componente cubre correctamente la interacción entre la interfaz externa (boundary), la lógica de control (control) y los datos (entity). Identificá los objetos de cada tipo que participan en los flujos principales de este componente.

| Objeto | Tipo (Boundary / Control / Entity) | Responsabilidad |
|---|---|---|
| [Nombre] | | |

#### 10.2.4 Diagrama de secuencia — flujo principal

> **Instrucciones:** Mostrá el flujo de mensajes entre los objetos identificados en el análisis de robustez para el caso de uso principal que este componente soporta. Incluí el camino feliz y al menos un camino de error significativo.

![Secuencia — Componente 2, flujo principal](../diagramas/secuencia-comp2-principal.png)
*Figura N — Secuencia: [Nombre del flujo principal de Componente 2]*

![Secuencia — Componente 2, camino de error](../diagramas/secuencia-comp2-error.png)
*Figura N — Secuencia: [Nombre del camino de error de Componente 2]*

---

### 10.3 Componente 3 — [Nombre del componente] *(Entrega final — S14)*

**Responsabilidad:** [Una oración que describe qué hace este componente y por qué es crítico para el sistema]

**Trazabilidad:** [Referencia a los casos de uso de la sección 1.4 que este componente soporta] → [Referencia al elemento en la vista de estructura interna, sección 7.2]

#### 10.3.1 Diagrama de clases de diseño

> **Instrucciones:** Este no es un diagrama de clases de análisis ni un modelo de dominio. Es el diseño: incluí métodos con firmas completas (nombre, parámetros, tipo de retorno), modificadores de acceso, relaciones de dependencia reales y las interfaces que el componente expone y consume. Mostrá cómo se aplican los patrones de diseño (sección 11) dentro de este componente.

![Diagrama de clases — Componente 3](../diagramas/clases-componente3.png)
*Figura N — Diagrama de clases de diseño: [Nombre del componente]*

#### 10.3.2 Contratos de interfaz

> **Instrucciones:** Para cada método o endpoint público del componente, documentá su contrato formal. Un contrato no es solo la firma — es la especificación de qué garantiza el método y qué exige de quien lo llama.

| Método / Endpoint | Precondición | Postcondición | Excepciones |
|---|---|---|---|
| `[firma del método]` | [Qué debe ser verdad antes de llamarlo] | [Qué garantiza que será verdad después] | [Qué errores puede lanzar y bajo qué condición] |

#### 10.3.3 Análisis de robustez

> **Instrucciones:** Usá el análisis de robustez para verificar que el diseño del componente cubre correctamente la interacción entre la interfaz externa (boundary), la lógica de control (control) y los datos (entity). Identificá los objetos de cada tipo que participan en los flujos principales de este componente.

| Objeto | Tipo (Boundary / Control / Entity) | Responsabilidad |
|---|---|---|
| [Nombre] | | |

#### 10.3.4 Diagrama de secuencia — flujo principal

> **Instrucciones:** Mostrá el flujo de mensajes entre los objetos identificados en el análisis de robustez para el caso de uso principal que este componente soporta. Incluí el camino feliz y al menos un camino de error significativo.

![Secuencia — Componente 3, flujo principal](../diagramas/secuencia-comp3-principal.png)
*Figura N — Secuencia: [Nombre del flujo principal de Componente 3]*

![Secuencia — Componente 3, camino de error](../diagramas/secuencia-comp3-error.png)
*Figura N — Secuencia: [Nombre del camino de error de Componente 3]*

---

## 11. Patrones de diseño aplicados

> **Instrucciones:** Documentá los patrones de diseño que aplicaste en el sistema. Se requieren **mínimo 3 patrones**. Para cada patrón, no describas el patrón en general — describí cómo lo aplicaste en este sistema específico. La justificación debe responder: ¿qué problema concreto de este sistema resuelve este patrón?, ¿qué alternativa consideraste y por qué el patrón fue mejor? El diagrama debe mostrar la aplicación real, no el diagrama genérico del libro.

---

### Patrón 1 — [Nombre del patrón, ej. "Strategy"]

| Campo | Detalle |
|---|---|
| **Categoría** | Creacional / Estructural / Comportamiento |
| **Ubicación en el sistema** | [En qué componente o capa se aplica — referencia a sección 10] |
| **Problema que resuelve** | [Descripción del problema concreto en este sistema que motivó el uso del patrón] |
| **Alternativa considerada** | [Qué otra solución se evaluó] |
| **Por qué el patrón y no la alternativa** | [Justificación técnica, no "porque es buena práctica"] |

![Aplicación del patrón Strategy en [Componente]](../diagramas/patron-strategy.png)
*Figura N — Aplicación del patrón [Nombre] en [Componente/contexto]*

---

### Patrón 2 — [Nombre del patrón]

*(Repetir estructura)*

---

### Patrón 3 — [Nombre del patrón]

*(Repetir estructura. Agregar Patrón 4, 5, etc. según aplique)*

---

## 12. Principios y técnicas habilitadoras — evidencia
 
> Para cada principio declarado en la sección 6, se aporta evidencia concreta con referencia a clase, interfaz, ADR o diagrama que demuestra su aplicación en SmartBilling Connect. Los principios evaluados son los siete comprometidos en la sección 6, sin omisiones ni adiciones.
 
| Principio | Evidencia concreta | Referencia |
|---|---|---|
| **Separación de responsabilidades** | El dominio fiscal está físicamente aislado del dominio comercial: el **Servicio de Facturación Fiscal** es un contenedor desplegable independiente de la **API de Aplicación** (§7.2.3). Esto no es una convención de carpetas sino una frontera de proceso y de red: la API de Aplicación llama al Servicio Fiscal por REST/HTTPS interno y nunca accede directamente a sus tablas. El **Procesador Asíncrono** (Workers) vive en un tercer proceso que se ocupa exclusivamente de relevar el outbox, reintentar envíos a Hacienda, escribir en el Almacén de Auditoría y despachar notificaciones. El **Identity Provider** (Keycloak) corre como cuarto contenedor con su propia base de datos. Cada unidad tiene un ciclo de cambio y un nivel de criticidad distinto: el Servicio Fiscal puede endurecerse o desplegarse sin tocar el CRM; los Workers pueden escalar sin afectar la API. | ADR-001 (Aislamiento del dominio fiscal), Diagrama C4 nivel 2 (§7.2.2, Figura 4), Tabla de contenedores (§7.2.3) |
| **Diseño para el cambio (bajo acoplamiento)** | El Servicio de Facturación Fiscal encapsula el esquema XML de Hacienda detrás de la interfaz `IXmlComprobanteBuilder` (§10.1.2). Si Hacienda publica una versión 4.4, se implementa un `XmlComprobanteBuilderV44` sin modificar `FiscalInvoiceService` ni ningún otro contenedor — el formato XML es un detalle interno del servicio. La firma digital está detrás de `ISignatureProvider`; la comunicación con Hacienda, detrás de `IHaciendaClient` con su política de reintentos y circuit breaker. En la frontera externa, el motor de automatización (sistema externo) entrega preventas por un contrato de handoff REST con `Idempotency-Key`; si se cambia la herramienta de automatización, la API de Aplicación no cambia porque solo conoce el contrato del endpoint, no al motor. | QS-05 (≤ 10 días hábiles, cero subsistemas ajenos), Contratos de interfaz (§10.1.2), Diagrama de secuencia de emisión (Figura 5) |
| **Defensa en profundidad** | La seguridad opera en 4 capas independientes, cada una en un contenedor o mecanismo distinto: (1) **Keycloak** (Identity Provider) autentica y emite JWT con `tenant_id` y roles — corre como contenedor propio, aislado; (2) **Middleware de autorización** en la API de Aplicación y en el Servicio Fiscal valida RBAC y verifica que el recurso pertenezca al tenant del token antes de cada operación; (3) **Cifrado** en tránsito (TLS entre contenedores, ADO.NET/TLS hacia SQL Server) y en reposo para datos sensibles (certificados de firma, datos personales — REST-07); (4) **Almacén de Auditoría** append-only e insert-only (contenedor separado, §7.2.3) donde ningún actor — ni administradores — puede modificar ni eliminar entradas una vez escritas (invariante 6 de §1.6). Un fallo en la capa (1) no expone datos porque la capa (2) sigue verificando el tenant en cada query; un fallo en (2) tiene como segunda barrera las políticas de RLS a nivel de SQL Server. | ADR-003 (Multi-tenancy centralizada), QS-01 (tres capas deben fallar simultáneamente), §7.1.1 (Fronteras de confianza) |
| **Fuente de verdad única por entidad** | La tabla de fuentes de verdad de §1.6 se implementa de forma concreta: el estado fiscal del comprobante (`Aceptado`/`Rechazado`) lo determina exclusivamente **Hacienda** — el Servicio de Facturación Fiscal refleja y conserva ese estado pero nunca lo fija por su cuenta (`ComprobanteStateMachine` solo permite la transición `Enviado → Aceptado` o `Enviado → Rechazado` como resultado de una respuesta de Hacienda, no como acción interna). El estado `PendienteValidacionHacienda` es legítimo y auditable: marca la ventana entre el envío y la respuesta, con timestamps de encolado y confirmación (QS-02). La **preventa** la genera el motor de automatización y, una vez entregada por handoff, el motor no puede modificarla — solo el usuario interno la gestiona dentro de la API de Aplicación. El **registro de auditoría** es su propia fuente de verdad inmutable en un contenedor separado. | §1.6 (Tabla de fuentes de verdad e invariantes), §10.1.2 (`ComprobanteStateMachine.Transicionar`), QS-02, Flujo 1 de §7.3 (Figura 5) |
| **Idempotencia por diseño** | Toda operación con efecto de negocio que recibe eventos potencialmente duplicados implementa deduplicación por clave: (1) El **handoff de preventa** del motor usa una `Idempotency-Key` en el header HTTP; la API de Aplicación verifica la clave contra un almacén antes de crear la preventa — si ya existe, retorna 200 con el resultado previo sin ejecutar efecto (Flujo 2, Figura 6). (2) El **Servicio de Facturación Fiscal** verifica `event_id` antes de procesar un evento `emitir_comprobante` relevado desde el outbox — un reintento del Procesador Asíncrono no genera una segunda factura (§10.1.2, contrato de `EmitirComprobanteAsync`). (3) Los **callbacks de Hacienda** se deduplicaan por ID de comprobante: una respuesta duplicada de aceptación no cambia el estado de un comprobante ya aceptado (transición inválida en `ComprobanteStateMachine`). | RF-06 (driver de idempotencia), QS-06 (100 % eventos duplicados producen exactamente un efecto), ADR-002 (outbox con deduplicación por `event_id`) |
| **Principio de menor privilegio (PoLA)** | El motor de automatización se autentica con credencial de integración OAuth2 propia (no sesión de usuario humano) y recibe un token con alcance que **no incluye acceso al dominio fiscal** (§3.4, REST-05). Concretamente: puede hacer POST al endpoint de handoff de preventa y GET a estados públicos, pero no puede invocar ningún endpoint del Servicio de Facturación Fiscal ni escribir en la tabla de comprobantes. Esto se verifica en el flujo 2 (Figura 6). Dentro del sistema, los roles RBAC (administrador, vendedor, asistente) limitan las operaciones por usuario: un vendedor puede crear cotizaciones y convertirlas en facturas, pero no puede modificar parámetros fiscales ni acceder a la configuración de tenants — eso es exclusivo del administrador (RF-04, §1.4). El Procesador Asíncrono tiene permiso insert-only sobre el Almacén de Auditoría y nunca update ni delete. | §3.4 (Tabla: el motor SÍ/NO puede), Flujo 2 (§7.3, Figura 6), ADR-003 (resolución de `tenant_id` en autorización) |
| **KISS / YAGNI** | Se eligió una arquitectura service-based con 4 servicios de grano grueso en lugar de microservicios finos (§8.2): esto da la frontera física que el dominio fiscal necesita sin proliferar infraestructura distribuida. La base de datos transaccional es una única instancia de SQL Server compartida (con aislamiento lógico por `tenant_id`), no una BD por servicio. El Identity Provider es Keycloak (producto maduro, gratuito, no desarrollo propio). El broker es RabbitMQ (open-source, un solo contenedor, configuración estándar). No se adoptó event sourcing ni CQRS porque el volumen de eventos (cientos/día) no lo justifica (§15.1): la tabla de auditoría append-only cubre la trazabilidad fiscal sin la complejidad de reconstruir estado desde eventos. Cada decisión de complejidad fue evaluada contra REST-06 (equipo de 3 personas). | ADR-001 (justificación service-based vs. microservicios), §8.2 (alternativas rechazadas), §15.1 (postura frente a event sourcing) |
 
### 12.1 Tensiones entre principios y cómo se resolvieron
 
> Cuando dos principios entran en conflicto, se debe tomar una decisión explícita. Las siguientes tensiones se identificaron durante el diseño de SmartBilling Connect.
 
| Tensión | Principio A | Principio B | Cómo se resolvió | ADR/Referencia |
|---|---|---|---|---|
| **KISS vs. Separación de responsabilidades** | KISS dice que menos unidades desplegables es mejor: un solo proceso sería lo más simple de operar para un equipo de 3 (REST-06). | Separación de responsabilidades dice que el dominio fiscal debe tener una frontera física, no solo lógica, para impedir que el motor de automatización o el código comercial crucen al ámbito regulado. | Se priorizó la separación pero acotada: solo 4 servicios de grano grueso, no 8-10 microservicios (§8.2). El Servicio de Facturación Fiscal se extrajo como contenedor propio porque REST-05 y QS-05 lo exigen como barrera arquitectónica, no como convención. KISS se aplicó dentro de cada servicio (implementaciones directas, sin sobre-abstracción) y en la infraestructura (un solo broker, una sola BD transaccional, un solo Identity Provider). | ADR-001, §8.1, §8.2 |
| **Fuente de verdad única vs. Disponibilidad** | Fuente de verdad única (§1.6) dice que el estado fiscal lo determina Hacienda: sin respuesta de Hacienda, el estado es indefinido. | Disponibilidad (QA-02, QS-02) exige que el sistema siga facturando aunque Hacienda no responda durante 30+ minutos. | Se modeló `PendienteValidacionHacienda` como un estado fiscal legítimo y auditable — no es un hack ni un estado de error, sino la representación honesta de "enviado, esperando veredicto". El log de auditoría captura el timestamp de encolado y el de confirmación posterior, preservando la trazabilidad sin bloquear la operación. Hacienda sigue siendo la autoridad del estado final, pero el sistema puede operar mientras espera. | QS-02, §1.6 (tabla de fuentes de verdad), Flujo 1 §7.3 (camino de degradación) |
| **Defensa en profundidad vs. KISS** | Defensa en profundidad exige 4 capas de seguridad (Keycloak, RBAC, cifrado, auditoría inmutable), cada una con configuración y mantenimiento. | KISS dice que cada capa añadida es complejidad que el equipo de 3 personas debe mantener (REST-06). | Se implementaron las 4 capas usando componentes que minimizan el esfuerzo operativo: Keycloak es un producto gestionado (no un desarrollo propio de autenticación), el middleware de autorización es reutilizable en la API y en el Servicio Fiscal (mismo código, dos despliegues), el cifrado usa TLS estándar y funciones nativas de SQL Server (no un servicio externo), y la auditoría es un subscriber del outbox que escribe en un esquema insert-only (sin infraestructura adicional más allá del Procesador Asíncrono que ya existe). La profundidad se mantiene, pero cada capa es la implementación más simple que cumple el requisito. | ADR-002, ADR-003, §7.2.3 (contenedores) |
| **Idempotencia por diseño vs. Rendimiento** | Idempotencia (RF-06, QS-06) exige verificar una clave de deduplicación antes de cada operación con efecto de negocio, lo que añade una consulta al almacén de claves en cada evento. | Rendimiento (QA-04, QS-03) exige procesar 50 eventos/minuto en ≤ 4 s P95 sin degradación. | Se aplicó idempotencia solo a operaciones con efecto de negocio (handoff de preventa, emisión de comprobante, callbacks de Hacienda), no a consultas de solo lectura. La verificación usa un índice único sobre la clave de idempotencia en SQL Server, con costo marginal por consulta (lookup por índice, no full scan). Para el handoff de preventa, la verificación y la creación ocurren en la misma transacción, sin round-trip adicional. El overhead medido es despreciable frente al costo de las operaciones de red (Hacienda: cientos de ms; handoff: decenas de ms). | RF-06, QS-06, QS-03, §10.1.2 (contrato de `EmitirComprobanteAsync`) |
 
---

# BLOQUE 6 — CALIDAD Y TRAZABILIDAD
*Hito: Entrega final (S14)*

---

## 13. Análisis de calidad del diseño
### 13.1 Validación de escenarios de calidad contra el diseño final
 
> Para cada escenario QS-XX de la sección 4, se argumenta cómo el diseño final lo satisface. Las medidas requeridas son las definidas en la sección 4; la columna "Cómo el diseño lo satisface" referencia las decisiones concretas (contenedores, componentes, ADRs, patrones) que lo habilitan. Se incluye el riesgo residual donde corresponde.
 
| Escenario | Medida requerida (§4) | Cómo el diseño lo satisface | Decisiones que lo habilitan | Riesgo residual |
|---|---|---|---|---|
| **QS-01 — Seguridad: acceso no autorizado a datos de otro tenant** | 100 % de intentos bloqueados con HTTP 403 en la suite de pruebas de autorización. Ninguna consulta sin `tenant_id` válido llega a repositorios fiscales. Evento de auditoría registrado en ≤ 500 ms. | Tres capas independientes deben fallar simultáneamente para que el acceso cruce tenants: (1) **Keycloak** valida el JWT y extrae `tenant_id` y roles; si el token es inválido o ausente, rechaza con 401 antes de tocar cualquier servicio. (2) **Middleware de autorización** en la API de Aplicación y el Servicio Fiscal verifica que el recurso solicitado pertenezca al `tenant_id` del token; si no coincide, rechaza con 403. (3) **SQL Server** aplica filtro obligatorio por `tenant_id` en cada query a través de un contexto de tenant inyectado por middleware — ninguna consulta puede omitir ese filtro porque el repositorio (`ComprobanteSqlRepository`) lo recibe como dependencia obligatoria, no como parámetro opcional. El intento fallido se persiste en el outbox dentro de la misma transacción y el Procesador Asíncrono lo escribe en el Almacén de Auditoría append-only. | ADR-003 (tenant_id centralizado), ADR-001 (frontera física del Servicio Fiscal), §7.1.1 (fronteras de confianza), §1.6 (invariante 7: ninguna consulta sin tenant_id válido) | Si un desarrollador bypassea el middleware y construye una query SQL manual sin filtro de tenant, la capa (3) debería atrapar el caso, pero depende de la disciplina en el uso del repositorio. Se mitiga con revisión de código y tests de integración que verifican que toda ruta de acceso pasa por el contexto de tenant. |
| **QS-02 — Disponibilidad: fallo del servicio de Hacienda** | Degradación perceptible ≤ 2 s. Documentos encolados procesados en ≤ 10 min tras restauración. Disponibilidad del flujo local ≥ 99.5 % mensual. Ningún documento encolado se pierde ante reinicios. | Cuando Hacienda retorna timeout o 5xx, el Servicio de Facturación Fiscal transiciona el comprobante a `PendienteValidacionHacienda` (estado fiscal legítimo y auditable, §1.6) y publica el evento de reintento al broker. El **Procesador Asíncrono** reintenta con backoff exponencial y circuit breaker: si Hacienda sigue caído, el circuito se abre por 5 minutos y deja de intentar, evitando saturación. Al restaurarse, la cola FIFO se procesa en orden sin intervención manual. El usuario solo percibe el cambio de estado en la SPA (≤ 2 s de degradación visual). La durabilidad la garantiza **RabbitMQ** (cola durable con persistencia en disco): un reinicio del sistema no pierde mensajes encolados. El resto del sistema (CRM, preventas, cotizaciones) no se ve afectado porque la emisión fiscal es asíncrona (outbox → broker → Servicio Fiscal). | ADR-002 (outbox transaccional), ADR-001 (Servicio Fiscal aislado), Flujo 1 §7.3 (Figura 5, camino de degradación), §14.1 (reintentos y circuit breaker) | La ventana de inconsistencia temporal: durante la indisponibilidad, el comprobante existe internamente pero Hacienda no lo ha validado. Se mitiga con el estado `PendienteValidacionHacienda` que hace esa ventana visible y auditable, no silenciosa. |
| **QS-03 — Rendimiento: 50 webhooks en 60 s durante campaña de ventas** | Respuesta extremo a extremo ≤ 4 s P95 (webhook → respuesta al cliente). Throughput ≥ 50 eventos/min. Tasa de no procesados < 1 %. | El procesamiento de webhooks ocurre **fuera del sistema**: el motor de automatización (externo) recibe los webhooks de Meta/TikTok, responde al cliente y guía hacia la app. SmartBilling Connect solo recibe el resultado como **handoff de preventa** — un POST REST con `Idempotency-Key`. Esto desacopla la latencia social de la operación interna: el P95 de 4 s se mide en la capa del motor, donde no hay transacción fiscal ni escritura de outbox en el camino crítico. Dentro del sistema, el handoff es una operación ligera (validar clave, crear preventa, escribir outbox, responder 201) que no compite con la emisión fiscal. La API de Aplicación no procesa webhooks de redes sociales directamente — esa es responsabilidad del motor, que escala de forma independiente. | REST-05 y §3.4 (frontera del motor), Flujo 2 §7.3 (Figura 6), §8.1 (el motor absorbe los picos de mensajería social) | Si el motor de automatización se satura, las respuestas a clientes se degradan fuera del control del sistema. Se mitiga documentando requisitos de capacidad del motor como parte del contrato operativo, pero el riesgo es inherente a una dependencia externa. |
| **QS-04 — Trazabilidad: auditoría ante emisión masiva (200 facturas en lote)** | Evento de auditoría persistido en outbox durable antes de responder (misma transacción). Latencia adicional del outbox ≤ 80 ms por comprobante. 100 % de comprobantes con evento de auditoría en la suite de pruebas. Integridad del log verificable por hashes encadenados. Retención ≥ 5 años. | El patrón **Transactional Outbox** (ADR-002) es la pieza central: `FiscalInvoiceService.EmitirComprobanteAsync` escribe el comprobante (estado `Firmado`) y su evento de auditoría en la tabla outbox **dentro de la misma transacción de SQL Server** — si la transacción falla, ni el comprobante ni el evento existen; si confirma, ambos están durablemente persistidos. El `OutboxWriter` (§10.1.3) es el responsable de esa escritura transaccional. El Procesador Asíncrono releva el outbox al broker y desde ahí escribe en el **Almacén de Auditoría** (contenedor separado, insert-only, §7.2.3). El log final es eventualmente consistente, pero el evento durable ya existe en el outbox antes de que el sistema retorne respuesta. Los hashes encadenados en el Almacén de Auditoría hacen que alterar una entrada invalide todas las posteriores. El actor que originó cada comprobante (usuario humano o proceso interno de cierre de mes) queda registrado con su identidad, tenant y timestamp con precisión de milisegundo. | ADR-002 (outbox transaccional), §10.1.2 (contrato de `EmitirComprobanteAsync`), §10.1.3 (`OutboxWriter`), §7.2.3 (Almacén de Auditoría append-only) | Ventana de segundos entre el commit transaccional y la entrada visible en el Almacén de Auditoría final. Durante esa ventana, la evidencia existe en el outbox pero no es consultable desde la interfaz de auditoría. Se mitiga registrando timestamps de encolado y de confirmación para que la ventana quede trazada. |
| **QS-05 — Modificabilidad: nuevo esquema XML de Hacienda** | Tiempo total publicación → despliegue validado ≤ 10 días hábiles. Subsistemas ajenos al módulo fiscal que requieren modificación: cero. Cobertura de pruebas ≥ 90 %. | El **Servicio de Facturación Fiscal** es un contenedor desplegable independiente (ADR-001): tiene su propio pipeline CI/CD, su propia imagen Docker y su propio ciclo de releases. El cambio de esquema XML se localiza exclusivamente en la implementación de `IXmlComprobanteBuilder` (§10.1.2): se crea `XmlComprobanteBuilderV44`, se actualizan las validaciones y se ajustan los tests unitarios del servicio. La interfaz `IXmlComprobanteBuilder` no cambia — `FiscalInvoiceService` la consume sin conocer la versión del esquema (inversión de dependencia). Ningún otro contenedor se modifica: la API de Aplicación llama al Servicio Fiscal por la misma interfaz REST, los Workers relevan los mismos eventos del outbox, Keycloak no participa del flujo XML. El despliegue del Servicio Fiscal se hace de forma independiente, en caliente, sin downtime para los demás contenedores. | ADR-001 (aislamiento físico del dominio fiscal), §10.1.2 (interfaz `IXmlComprobanteBuilder`), §8.1 (justificación de service-based) | Durante la ventana de despliegue en caliente, pueden coexistir dos versiones del Servicio Fiscal. Las entradas de auditoría incluyen la versión del esquema XML para mantener trazabilidad entre versiones. El riesgo es que Hacienda cambie el esquema de forma incompatible hacia atrás; se mitiga porque `IXmlComprobanteBuilder` permite mantener dos implementaciones activas simultáneamente con routing por versión. |
| **QS-06 — Idempotencia: evento externo duplicado** | 100 % de eventos duplicados reconocidos producen exactamente un efecto de negocio. Ninguna operación fiscal marcada como idempotente genera segundo comprobante. Ventana de deduplicación ≥ periodo máximo de reintento por canal. | Tres puntos de entrada implementan deduplicación por clave: (1) **Handoff de preventa**: la API de Aplicación verifica la `Idempotency-Key` del header contra un índice único en SQL Server; si la clave existe, retorna 200 con el resultado previo sin crear segunda preventa (Flujo 2, Figura 6). (2) **Emisión fiscal**: el Servicio de Facturación Fiscal verifica `event_id` del evento AMQP antes de procesar `emitir_comprobante`; un reintento del Procesador Asíncrono no genera segunda factura (§10.1.2). (3) **Callbacks de Hacienda**: `ComprobanteStateMachine` rechaza transiciones inválidas — una respuesta duplicada de "Aceptado" sobre un comprobante ya aceptado no produce ningún efecto (transición `Aceptado → Aceptado` no existe en la tabla de transiciones). En los tres casos, la respuesta al duplicado es idéntica a la original: el sistema se comporta como si el evento hubiera llegado exactamente una vez. | ADR-002 (outbox con deduplicación), RF-06, §10.1.2 (`ComprobanteStateMachine.Transicionar` con `InvalidStateTransitionException`), Flujo 2 §7.3 | La ventana de deduplicación tiene un límite temporal: las claves de idempotencia se conservan por el periodo máximo de reintento configurado (ej. 72 h para webhooks, 7 días para preventas). Un duplicado que llegue después de ese periodo podría procesarse como nuevo. Se mitiga porque los reintentos de las fuentes externas (Meta, Hacienda, motor) tienen timeouts muy inferiores a esa ventana. |
 
### 13.2 Trade-offs entre atributos de calidad
 
> Los conflictos reales entre atributos de calidad donde se tomó una decisión explícita. Cada trade-off referencia el escenario de la sección 4 donde se manifiesta.
 
| Atributo A | Atributo B | Tensión | Decisión tomada | Consecuencia aceptada |
|---|---|---|---|---|
| **Seguridad (QA-01)** | **Rendimiento (QA-04)** | Cada request pasa por 3 capas de validación (JWT en Keycloak, RBAC en middleware, filtro de tenant en SQL Server) antes de ejecutar la operación de negocio. Cada capa añade latencia. La tensión se intensifica en QS-03 (50 eventos/min) y QS-04 (200 facturas en lote). | Se priorizó seguridad. El overhead de validación se minimizó: JWT se verifica localmente contra la clave pública de Keycloak (sin llamada de red por request), los permisos RBAC se cachean en memoria por sesión, y el filtro de `tenant_id` se inyecta como condición WHERE en cada query (costo marginal en SQL Server con índice). | Se acepta un overhead de ~10-20 ms por request por las 3 capas. No afecta el objetivo de ≤ 4 s P95 en QS-03 (el handoff de preventa es una operación ligera) ni de ≤ 80 ms de outbox en QS-04 (el outbox es una escritura transaccional, no un request externo). |
| **Disponibilidad (QA-02)** | **Consistencia / Integridad fiscal (QA-01)** | QS-02 requiere que el sistema siga facturando cuando Hacienda no responde. Pero QS-04 e invariante 5 de §1.6 exigen que toda factura tenga su entrada de auditoría y un estado fiscal válido. Durante la indisponibilidad, el estado real ante Hacienda es desconocido. | Se priorizó consistencia para transacciones fiscales mediante el patrón outbox: la factura y su evento de auditoría se confirman en la misma transacción de SQL Server (ADR-002). Si la BD falla, la factura no se emite. Para la ventana de indisponibilidad de Hacienda, se modeló `PendienteValidacionHacienda` como estado fiscal legítimo — no es consistencia eventual sino una representación fiel de la realidad: "enviado, esperando veredicto". | Si SQL Server tiene problemas de escritura, la factura no se emite hasta que se resuelvan — la disponibilidad del flujo de emisión depende de la BD. Se acepta una disponibilidad ligeramente menor a cambio de integridad fiscal absoluta. Para operaciones no fiscales (notificaciones, logs informativos), sí se usa consistencia eventual vía eventos. |
| **Modificabilidad (QA-05)** | **Complejidad operativa (REST-06)** | QS-05 exige que el Servicio Fiscal se despliegue de forma independiente, lo que implica mantener un servicio separado con su propio pipeline, su propia imagen Docker y su propio monitoreo — más carga operativa que un módulo dentro de un monolito. | Se priorizó modificabilidad para el dominio fiscal porque REST-05 y QS-05 lo justifican: la frontera del dominio regulado no puede ser una convención de carpetas (§8.2, alternativa N-Tier rechazada). Se compensó la complejidad operativa con KISS en el resto: el dominio comercial (CRM, cotizaciones, preventas) permanece en un monolito modular (API de Aplicación) que no requiere despliegue independiente. El total es 4 servicios, no 10 microservicios. | Se acepta el costo operativo de 4 servicios + 1 broker + 1 Identity Provider. Es más que un monolito puro, pero significativamente menos que microservicios. RabbitMQ y Keycloak son productos maduros con configuración estándar; el equipo no los desarrolla, solo los opera. |
| **Idempotencia (RF-06)** | **Rendimiento (QA-04)** | QS-06 exige verificar clave de idempotencia en cada operación con efecto de negocio. QS-03 exige procesar preventas con baja latencia (el handoff es parte de la cadena de 4 s P95). La verificación añade una consulta al almacén de claves en cada evento. | Se aplicó idempotencia selectivamente: solo a operaciones con efecto de negocio (handoff, emisión fiscal, callbacks de Hacienda), no a consultas de solo lectura. La verificación usa un índice único en SQL Server (lookup O(log n), no full scan). Para el handoff, la verificación y la creación ocurren en la misma transacción (un solo round-trip a BD). | El overhead por verificación de idempotencia es del orden de 1-5 ms (lookup por índice), despreciable frente a la latencia de red del handoff (~50 ms) o de Hacienda (~200-2000 ms). No se justifica sacrificar idempotencia por milisegundos. |
 
### 13.3 Estimación de cohesión y acoplamiento
 
> Se evalúan los 3 contenedores de servicio diseñados por el equipo (los componentes .NET del sistema), que corresponden a las unidades de ejecución de la vista de contenedores (§7.2.3). No se evalúa el motor de automatización porque es un sistema externo (§3.4) ni Keycloak porque es un producto de terceros.
 
| Contenedor | Cohesión | Justificación | Acoplamiento | Justificación |
|---|---|---|---|---|
| **Servicio de Facturación Fiscal** (§10.1) | **Alta** (funcional) | Todas las clases del servicio (`FiscalInvoiceService`, `IXmlComprobanteBuilder` / `XmlComprobanteBuilderV44`, `XadesEpesSignatureProvider`, `IHaciendaClient` / `HaciendaHttpClient`, `ComprobanteStateMachine`, `ComprobanteSqlRepository`, `OutboxWriter`) colaboran para cumplir una sola responsabilidad: el ciclo de vida del comprobante electrónico desde la generación XML hasta la confirmación de Hacienda. No hay clases que hagan cosas no relacionadas con el dominio fiscal. Cada clase interna tiene una sola razón para cambiar: `XmlComprobanteBuilderV44` cambia por esquema XML, `XadesEpesSignatureProvider` cambia por estándar de firma, `HaciendaHttpClient` cambia por protocolo de Hacienda. | **Bajo** | Dependencias eferentes mínimas: (1) SQL Server para persistencia de comprobantes y outbox (acceso por `ComprobanteSqlRepository` y `OutboxWriter`), (2) MinIO para archivado del XML/PDF, (3) API de Hacienda vía `IHaciendaClient`. No depende de la API de Aplicación, ni de los Workers, ni de Keycloak directamente — recibe requests REST autenticados cuyo JWT ya fue validado en la capa anterior. No conoce la existencia del CRM, las cotizaciones ni las preventas. Expone la interfaz REST interna que la API de Aplicación consume sin conocer su implementación. |
| **API de Aplicación** (§7.2.3) | **Media-Alta** (funcional con múltiples subdominios) | Concentra el dominio comercial completo: CRM/clientes, cotizaciones, preventas, productos y orquestación del flujo. Internamente es un monolito modular con módulos de fronteras lógicas. La cohesión no es máxima porque agrupa subdominios distintos (CRM vs. cotizaciones vs. preventas), pero todos comparten el mismo contexto comercial — no se mezcla lógica fiscal ni de identidad. | **Medio** | Tiene el acoplamiento más alto de los tres servicios: (1) llama al Servicio de Facturación Fiscal por REST interno cuando un usuario convierte una cotización en factura, (2) depende de Keycloak para validar JWT en cada request, (3) escribe en SQL Server (tabla transaccional + outbox), (4) es el punto de entrada del handoff de preventa del motor externo. Este acoplamiento es inherente a su rol de núcleo orquestador — se mitiga porque cada dependencia es contra un contrato (interfaz REST del Servicio Fiscal, protocolo OIDC de Keycloak, esquema de BD compartido), no contra una implementación interna. |
| **Procesador Asíncrono (Workers)** (§7.2.3) | **Alta** (funcional) | Toda su lógica se centra en una sola responsabilidad: relevar eventos del outbox y procesarlos. Cada worker implementa una sola función: uno releva el outbox a RabbitMQ, otro procesa la escritura en el Almacén de Auditoría (insert-only), otro despacha notificaciones por correo, otro gestiona reintentos hacia Hacienda. Ningún worker contiene lógica de negocio fiscal ni comercial — son ejecutores de efectos secundarios. | **Medio** | Dependencias eferentes diversas pero controladas: (1) lee el outbox de SQL Server, (2) publica en RabbitMQ, (3) escribe en el Almacén de Auditoría (insert-only), (4) invoca el Servicio de Correo (SMTP/API), (5) puede reinvocar al Servicio Fiscal para reintentos de envío a Hacienda. Cada dependencia es contra un contrato estable (esquema de outbox, protocolo AMQP, esquema insert-only, API de correo). El acoplamiento eferente es moderado pero cada conexión es de baja volatilidad — estos contratos cambian con poca frecuencia. |
 
**Resumen de métricas:**
 
| Métrica | Servicio de Facturación Fiscal | API de Aplicación | Procesador Asíncrono (Workers) |
|---|---|---|---|
| Cohesión | Alta (funcional) | Media-Alta (funcional, múltiples subdominios comerciales) | Alta (funcional) |
| Acoplamiento aferente (quién depende de mí) | Alto — la API de Aplicación y los Workers lo consumen | Alto — todos los usuarios y el motor externo entran por aquí | Bajo — nadie depende directamente de él; es consumidor, no proveedor |
| Acoplamiento eferente (de quién dependo) | Bajo — SQL Server, MinIO, API Hacienda | Medio — Keycloak, Servicio Fiscal, SQL Server | Medio — SQL Server (outbox), RabbitMQ, Almacén de Auditoría, Servicio de Correo |
| Inestabilidad (eferente / total) | Baja (0.25) — módulo estable | Media (0.5) — equilibrado | Alta (0.7) — módulo que absorbe cambios operativos |
 
> **Interpretación:** El Servicio de Facturación Fiscal es el contenedor más estable del sistema (inestabilidad 0.25), lo cual es correcto porque contiene la lógica de dominio fiscal que no debe cambiar frecuentemente — solo cambia por actualizaciones regulatorias de Hacienda (QS-05), que están diseñadas para ser absorbidas internamente sin propagar cambios. El Procesador Asíncrono es el más inestable (0.7): como ejecutor de efectos secundarios, es el primero que se modifica cuando se agrega un nuevo canal de notificación, se cambia la política de reintentos o se ajusta el procesamiento del outbox. Esto es coherente con el Principio de Abstracciones Estables (SAP, Martin 2017): el módulo estable (Servicio Fiscal) expone interfaces abstractas (`IXmlComprobanteBuilder`, `ISignatureProvider`, `IHaciendaClient`), mientras que el módulo inestable (Workers) implementa flujos concretos de procesamiento que cambian con más frecuencia.
 
---

# BLOQUE 7 — SECCIONES ESPECÍFICAS POR TIPO DE SISTEMA
*Hito: Entrega final (S14) — incluir solo las que aplican al sistema*

> **Instrucciones:** Incluí solo las secciones que corresponden al tipo de sistema que diseñaste. Si tu sistema no es distribuido, no incluís la sección 14.1. Si no tiene componentes de IA, no incluís la 14.4. Justificá al inicio de este bloque cuáles secciones incluís y por qué.

**Secciones incluidas en este proyecto:** [Listá las que aplican con una oración de justificación]

---

## 14.1 Sistemas distribuidos / cloud *(si aplica)*

> **Instrucciones:** Documentá las decisiones de diseño específicas para la distribución. Incluí la estrategia de consistencia (fuerte, eventual, causal), el modelo CAP aplicado, la estrategia de tolerancia a particiones y el modelo de despliegue en nube con los servicios específicos usados.

### Estrategia de consistencia
[Completar — qué modelo de consistencia usa el sistema y por qué es el adecuado para el dominio]

### Modelo CAP aplicado
[Completar — entre CP y AP, cuál favorece el sistema y bajo qué condiciones. Justificar con los escenarios de calidad]

### Manejo de fallos y resiliencia
[Completar — circuit breakers, retries, timeouts, fallbacks. Para cada mecanismo: dónde aplica y por qué]

---

## 14.2 Sistemas concurrentes / tiempo real *(si aplica)*

> **Instrucciones:** Documentá el modelo de concurrencia del sistema. Identificá los recursos compartidos, los mecanismos de sincronización y cómo se evitan condiciones de carrera y deadlocks.

### Modelo de concurrencia
[Completar — hilos, procesos, actores, eventos asincrónicos — qué modelo usa el sistema]

### Recursos compartidos y sincronización
| Recurso compartido | Mecanismo de sincronización | Riesgo de condición de carrera | Mitigación |
|---|---|---|---|
| [Recurso] | [Mutex, semáforo, lock, etc.] | [Sí/No — descripción] | [Cómo se previene] |

---

## 14.3 Sistemas IoT / edge *(si aplica)*

> **Instrucciones:** Documentá el diseño del flujo de datos desde los dispositivos hasta la nube o el sistema central. Incluí el protocolo de comunicación, la estrategia ante conectividad intermitente y el modelo de procesamiento en edge vs. nube.

### Topología edge-cloud
[Completar con diagrama de despliegue físico si no está cubierto en 7.4]

### Estrategia ante conectividad intermitente
[Completar — qué hace el dispositivo cuando pierde conectividad, cómo sincroniza cuando la recupera, qué datos se pierden y cuáles no]

### Protocolo de comunicación
[Completar — MQTT, CoAP, AMQP, HTTP/REST — justificación de la elección]

---

## 14.4 Sistemas con IA Generativa / Agentes *(si aplica)*

> **Instrucciones:** El diseño de sistemas con IA generativa o agentes tiene desafíos específicos que no aparecen en sistemas tradicionales. Documentá las decisiones de diseño para cada uno de los siguientes aspectos. La IA no puede ser una API call sin diseño — se evalúa que el grupo pensó en la arquitectura del sistema de IA, no solo en su uso.

### Diseño del orquestador
[Completar — cómo se coordinan los agentes o llamadas al modelo, qué decide el orquestador y qué los agentes individuales]

### Gestión de contexto y memoria
[Completar — cómo se mantiene el contexto entre interacciones, qué se persiste, qué se descarta, con qué estrategia de windowing]

### Estrategia de fallback
[Completar — qué hace el sistema cuando el modelo falla, produce respuestas inaceptables o supera el límite de latencia]

### Trazabilidad de decisiones del modelo
[Completar — cómo el sistema registra qué decisión tomó el modelo, con qué contexto y con qué resultado, para auditoría y mejora]

### Evaluación de calidad de respuestas
[Completar — cómo el sistema detecta alucinaciones, respuestas fuera de dominio o respuestas de baja calidad, y qué hace al respecto]

---

## 14.5 Sistemas con seguridad crítica *(si aplica)*

> **Instrucciones:** Si el sistema maneja datos sensibles, tiene requerimientos regulatorios o es un objetivo de ataque probable, documentá el modelo de seguridad desde el diseño.

### Modelo de amenazas (STRIDE simplificado)
| Amenaza | Componente en riesgo | Mitigación en el diseño |
|---|---|---|
| Spoofing | | |
| Tampering | | |
| Repudiation | | |
| Information Disclosure | | |
| Denial of Service | | |
| Elevation of Privilege | | |

### Controles por capa
[Completar — qué controles de seguridad existen en cada capa del sistema]

---

# BLOQUE 8 — TENDENCIAS Y EVOLUCIÓN
*Hito: Entrega final (S14)*

---

## 15. Tendencias y evolución del diseño

> **Instrucciones:** Este bloque no es una sección teórica sobre tendencias — es una reflexión fundamentada sobre cómo el diseño actual se posiciona frente a las tendencias estudiadas en el curso. Para cada tendencia relevante, documentá una decisión consciente: adoptaron elementos de ella, la consideraron y la rechazaron, o la dejaron como punto de extensión futuro. Cada postura debe estar justificada con criterios de diseño, no con preferencias personales.

### 15.1 Postura frente a tendencias relevantes

| Tendencia | Postura del diseño | Justificación |
|---|---|---|
| Microservicios | Adoptada / Parcialmente adoptada / Rechazada / Punto de extensión | [Por qué — conectado con los drivers y trade-offs del sistema] |
| Cloud-native / 12-factor | | |
| Diseño dirigido por el dominio (DDD) | | |
| IA Generativa / Agentes | | |
| [Otras tendencias relevantes para el dominio] | | |

### 15.2 Puntos de extensión del diseño

> **Instrucciones:** Identificá las partes del diseño que están preparadas para crecer o cambiar sin romper el sistema. Un buen diseño anticipa el cambio sin sobrediseñar. Para cada punto de extensión, indicá qué cambio habilitaría y qué decisión de diseño lo hace posible.

| Punto de extensión | Cambio que habilita | Decisión de diseño que lo soporta |
|---|---|---|
| [ej. Interfaz INotificationService desacoplada de la implementación] | [Agregar un nuevo canal de notificación sin modificar la lógica de negocio] | [ADR-003: abstracción de notificaciones detrás de interfaz] |

---

# APÉNDICES

---

## 16. Glosario

> **Instrucciones:** Definí los términos del dominio y los términos técnicos específicos de este sistema que un lector externo podría no conocer. El glosario es el "ubiquitous language" del proyecto — si el equipo usa términos del dominio de forma consistente en todo el documento, este glosario los define.

| Término | Definición |
|---|---|
| [Término] | [Definición en el contexto de este sistema] |

---

## 17. Referencias

> **Instrucciones:** Listá todas las fuentes usadas en el documento — libros del curso, artículos, documentación técnica, estándares. Usá formato APA o IEEE de forma consistente.

- Brown, S. (2014). *Software Architecture for Developers*. Leanpub.
- Budgen, D. (2003). *Software Design* (2.ª ed.). Addison-Wesley.
- Gomaa, H. (2011). *Software Modeling and Design*. Cambridge University Press.
- Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1995). *Design Patterns*. Addison-Wesley.
- [Agregar referencias adicionales usadas en el proyecto]

---

*Documento generado bajo el template estándar PSWE-04 — Universidad Cenfotec — Maestría Profesional en Ingeniería del Software*
