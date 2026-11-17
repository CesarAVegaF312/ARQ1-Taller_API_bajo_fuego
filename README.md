# Taller · API bajo fuego

Taller del **Módulo 4 (Estilos arquitectónicos · REST)** del Diplomado en Diseño y Arquitectura de Software, Universidad de La Sabana.

La API de pedidos de Inventarium funciona en la demo: cada petición, enviada una vez y con buena red, responde bien. En este taller la auditan, la ponen frente a una red móvil que pierde respuestas y rediseñan su contrato hasta que el mismo caos deje de cobrar dos veces. Al final deciden cómo exponer esa misma capacidad a cinco consumidores que no se parecen entre sí.

> Una API puede responder bien a cada petición y aun así cobrar dos veces. Lo que lo evita no está en el código de cada endpoint: está en el contrato.

---

## Objetivo general

Diagnosticar una API HTTP con el modelo de madurez de Richardson y la semántica de los métodos (seguridad e idempotencia), medir el costo operativo de sus defectos bajo una red no confiable y rediseñar el contrato con recursos, códigos de estado, claves de idempotencia y caché condicional, justificando cada decisión con evidencia.

## Al terminar, el grupo puede

- Clasificar un endpoint en el modelo de Richardson y explicar por qué varias URI no hacen, por sí solas, un nivel 1.
- Distinguir un método **seguro** de uno **idempotente** y predecir qué pasa cuando un cliente reintenta.
- Diseñar un `POST` reintentable con `Idempotency-Key` y explicar por qué la clave la genera el cliente, una vez por intención.
- Usar `ETag`, `If-None-Match` y `Cache-Control` para que una revalidación cueste encabezados y no cuerpo, y `If-Match` para no pisar cambios ajenos.
- Elegir el estilo de integración por consumidor (REST, gRPC, GraphQL, eventos, MCP) y nombrar lo que cada elección cuesta.

---

## Duración y formato

- **Sesión virtual sincrónica de 2 horas.**
- **Grupos de 3 o 4 personas**, cada uno en una sala de grupos. Una persona comparte pantalla con la página; las demás proponen peticiones y redactan.
- **No hay que instalar nada.** El servidor y la red son una simulación dentro de una sola página HTML.

## Complemento interactivo

- **Archivo:** [`api_bajo_fuego.html`](api_bajo_fuego.html) — se abre con doble clic, sin servidor ni dependencias (solo carga tipografías de Google Fonts si hay conexión).
- Todo lo que el grupo hace (hallazgos, decisiones, resultados, elecciones de consumidores) se guarda en el navegador. Si cierran la pestaña y la vuelven a abrir en el mismo equipo, siguen donde iban. Los datos de la consola (pedidos creados, pagos) se reinician al recargar.
- La simulación es determinista: con la misma **semilla** y los mismos parámetros, todos los grupos ven los mismos números. Eso permite comparar entre grupos.

| # | Ronda | Qué hay en la página |
|---|-------|----------------------|
| 01 | Auditoría | Consola HTTP contra la API v0, historial de peticiones y formulario de hallazgos con evidencia. |
| 02 | La red miente | Simulador de tráfico de clientes móviles: latencia, respuestas perdidas, reintentos y la carta «pasa un crawler». Marcador con pedidos y cobros duplicados, dinero cobrado de más, pedidos cancelados por un GET y KB transferidos. |
| 03 | Rediseño | Seis decisiones de contrato que cambian el servidor de verdad, la trampa de la clave nueva por reintento, nivel de Richardson calculado, consola con rutas v1 y tabla antes/después con el mismo caos. |
| 04 | Cinco consumidores | La misma capacidad para una app móvil, un socio B2B, un panel interno, un sistema de envíos y un agente de IA. Elección y justificación por consumidor, consecuencias reveladas y la herramienta MCP que saldría del contrato v1 del grupo. |
| 05 | Entrega | Markdown con todo lo anterior, listo para copiar o descargar. |

---

## Agenda

| Ronda | Min | Qué hacen |
|-------|----:|-----------|
| Encuadre | 10 | Abren la página, leen el contexto y anotan la semilla de la clase que el docente escribe en el chat (por defecto `2026`). |
| 01 · Auditoría | 25 | Exploran la v0 con la consola. Registran cada defecto con nivel de Richardson y una petición del historial como evidencia. |
| 02 · La red miente | 15 | Lanzan el tráfico con los parámetros que fija el docente. Para cada número en rojo escriben qué decisión lo habría evitado. |
| 03 · Rediseño | 35 | Activan decisiones, las prueban en la consola (201, 304, 409, 412, 422, repetición con la misma clave) y repiten el caos. Activan la trampa y explican lo que pasa. Llenan la plantilla del contrato. |
| 04 · Cinco consumidores | 20 | Eligen cómo exponer la capacidad a cada consumidor, justifican en una línea y revelan las consecuencias. Leen la herramienta MCP generada desde su contrato. |
| Cierre | 15 | Exportan el Markdown y completan las plantillas. |

**Total: 120 minutos.**

---

## La API v0

| Endpoint | Lo que hace |
|----------|-------------|
| `GET /getProductos` | Devuelve los 120 productos (~42 KB) en cada llamada. |
| `GET /getProducto?id=SKU-…` | Un producto. |
| `POST /crearPedido` | Crea un pedido. Cuerpo: `cliente`, `items[{sku,cantidad}]`, `direccion`. |
| `GET /pedido?id=…` | Un pedido. |
| `POST /pagarPedido` | Registra un pago. Cuerpo: `id`, `monto`, `medio`. |
| `GET /cancelarPedido?id=…` | Cancela un pedido. |
| `POST /actualizarPedido` | Cambia dirección, notas, ítems o cancela, según el campo `accion`. |

Datos de prueba en la consola: pedidos `1001` (pendiente), `1002` (pagado) y `1003` (pendiente); productos `SKU-1001` a `SKU-1120`.

## Las decisiones del contrato v1

| Decisión | Qué cambia en el servidor |
|----------|---------------------------|
| Recursos con sustantivos | Rutas `/productos`, `/pedidos/{id}`, `/pedidos/{id}/pagos`; paginación con `?limite` y `?cursor`. Habilita las demás. |
| Códigos y `problem+json` | 201 + `Location`, 404, 400 frente a 422, 409, 405 con `Allow`. Errores según RFC 9457. |
| `Idempotency-Key` en los POST | Misma clave y mismo cuerpo: respuesta original sin ejecutar de nuevo. Otro cuerpo: 422. Sin clave: 400. |
| `ETag` y caché condicional | Catálogo con `public, max-age=60`; 304 sin cuerpo; pedidos con `private, no-cache`; `If-Match` → 412. |
| Cancelar sin GET | `POST /pedidos/{id}/cancelacion` como recurso de proceso. |
| PATCH parcial | `PATCH /pedidos/{id}` con `application/merge-patch+json`. |

**La trampa:** un interruptor simula un cliente que genera una `Idempotency-Key` nueva en cada reintento. Actívenlo, repitan el mismo caos y expliquen con sus números lo que pasa.

---

## Entregable

Un solo envío por grupo en la plataforma, con tres archivos:

1. **`api-bajo-fuego-<grupo>.md`** — exportado desde la sección 05 de la página. Contiene hallazgos, decisiones, tabla antes/después, matriz de consumidores y las peticiones citadas como evidencia.
2. **`contrato_api_v1.md`** — a partir de [`plantillas/plantilla_contrato_api.md`](plantillas/plantilla_contrato_api.md).
3. **`defectos.md`** — a partir de [`plantillas/plantilla_defectos.md`](plantillas/plantilla_defectos.md): los hallazgos de la ronda 1 con impacto y la decisión que cierra cada uno.

## Rúbrica (5,0)

| Criterio | Peso | Qué se espera |
|----------|-----:|---------------|
| Hallazgos con evidencia | 1,2 | Al menos ocho defectos distintos, cada uno con una petición citada y su respuesta. Nivel de Richardson por endpoint con una línea de justificación. |
| Contrato v1 | 1,3 | Plantilla completa y coherente con las decisiones activadas: recursos, métodos, códigos por operación, encabezados, idempotencia y caché. Las decisiones no tomadas se justifican. |
| Antes y después | 1,0 | Tabla con los parámetros de la clase. Para cada métrica que cambió, qué decisión la movió. La trampa explicada con sus números. |
| Cinco consumidores | 1,0 | Una opción y una justificación por consumidor. Para al menos dos consumidores, comentario de la consecuencia revelada (si coincidía con lo que esperaban o no, y por qué). Lectura de la herramienta MCP generada: qué parte de la descripción depende de su contrato. |
| Claridad | 0,5 | Markdown legible, sin relleno. Cada afirmación remite a una petición, un número o una decisión. |

---

## Conceptos que se usan

- **Seguro:** el método no modifica estado observable (`GET`, `HEAD`). Cualquier intermediario puede reintentarlo, cachearlo o seguirlo como enlace.
- **Idempotente:** ejecutarlo N veces deja el mismo estado que ejecutarlo una vez (`PUT`, `DELETE`, y los seguros). `POST` no lo es; `PATCH` puede diseñarse para serlo.
- **Idempotency-Key:** encabezado que el cliente genera **una vez por intención** y repite en cada reintento. El servidor guarda la clave con la respuesta producida y la devuelve sin ejecutar de nuevo.
- **Caché condicional:** `ETag` identifica la versión de una representación; `If-None-Match` pide «solo si cambió» y recibe `304 Not Modified` sin cuerpo; `If-Match` exige la versión esperada antes de modificar y recibe `412` si no coincide.
- **`problem+json`:** formato estándar para el cuerpo de los errores HTTP (RFC 9457).

## Relación con el resto del curso

- La presentación de REST del módulo 4 (`03-rest.html`, en la carpeta de presentaciones del curso) cubre las seis restricciones, el modelo de Richardson, la tabla de seguridad e idempotencia y la caché condicional; este taller las pone a prueba.
- El evento `PedidoPagado` de la ronda 4 conecta con el patrón Outbox de la sesión de microservicios y con el módulo 7 (comunicación asíncrona).
- El adaptador MCP de la ronda 4 es un adaptador de entrada en el sentido de la arquitectura hexagonal del módulo 5.

## Recursos

- Fielding, R. (2000). *Architectural Styles and the Design of Network-based Software Architectures*, cap. 5.
- RFC 9110 — HTTP Semantics (métodos seguros e idempotentes, códigos de estado, solicitudes condicionales).
- RFC 9111 — HTTP Caching.
- RFC 9457 — Problem Details for HTTP APIs.
- RFC 7396 — JSON Merge Patch.
- IETF HTTPAPI WG — *The Idempotency-Key HTTP Header Field* (borrador `draft-ietf-httpapi-idempotency-key-header`).
- Fowler, M. (2010). *Richardson Maturity Model*. martinfowler.com.
- Especificación de Model Context Protocol — modelcontextprotocol.io.
