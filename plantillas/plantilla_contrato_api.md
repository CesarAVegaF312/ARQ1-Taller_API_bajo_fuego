# Contrato API v1 · Pedidos de Inventarium

> Plantilla. Reemplacen todo lo que está entre `< >`. Borren las filas que no apliquen y agreguen las que falten.
> El contrato debe coincidir con las decisiones que activaron en la ronda 3. Si una decisión no la tomaron, díganlo en la sección 7 y expliquen por qué.

**Grupo:** <nombre e integrantes>
**Versión:** 1.0.0
**Nivel de Richardson declarado:** <0 · 1 · 2> — <una línea de justificación>

---

## 1. Convenciones

| Tema | Decisión |
|------|----------|
| URL base | `https://api.inventarium.example` |
| Formato | `application/json; charset=utf-8` |
| Errores | <`application/problem+json` (RFC 9457) / otro> |
| Moneda | Enteros en COP, sin decimales |
| Fechas | ISO 8601 en UTC |
| Paginación | <`?limite` (1–200, por defecto 50) y `?cursor`; encabezado `Link rel="next"`> |
| Versionado | <en la ruta / en un encabezado / ninguno> y por qué |
| Idempotencia | <qué operaciones exigen `Idempotency-Key`, cuánto tiempo se guarda la clave> |

## 2. Recursos

| Recurso | URI | Identidad | Notas |
|---------|-----|-----------|-------|
| Producto | `/productos/{sku}` | `sku` | Solo lectura |
| Colección de productos | `/productos` | — | Filtro `?categoria` |
| Pedido | `/pedidos/{id}` | `id` | |
| Pago | `/pedidos/{id}/pagos/{pagoId}` | `pagoId` | |
| <Cancelación> | <`/pedidos/{id}/cancelacion`> | <una por pedido> | <recurso de proceso> |

## 3. Operaciones

Una ficha por operación. Copien el bloque las veces que haga falta.

### `<MÉTODO> <URI>`

- **Propósito:** <una línea>
- **Seguro / idempotente:** <sí/no> / <sí/no — y cómo, si lo es por diseño>
- **Encabezados de la petición:** <`Content-Type`, `Idempotency-Key`, `If-None-Match`, `If-Match`…>
- **Cuerpo de la petición:**

```json
{ }
```

- **Respuestas:**

| Código | Cuándo | Encabezados | Cuerpo |
|-------:|--------|-------------|--------|
| 200 | | | |
| 201 | | `Location` | |
| 304 | | `ETag`, `Cache-Control` | (vacío) |
| 400 | | | problem+json |
| 404 | | | problem+json |
| 409 | | | problem+json |
| 412 | | | problem+json |
| 422 | | | problem+json con `errores[]` |

- **Caché:** <`Cache-Control` que devuelve y por qué ese tiempo de frescura>
- **Evidencia en la consola:** <número de petición del historial que muestra este comportamiento>

## 4. Errores

Ejemplo de cuerpo de error del contrato:

```json
{
  "type": "https://api.inventarium.example/problemas/<slug>",
  "title": "<título corto>",
  "status": 422,
  "detail": "<qué pasó, en una línea>",
  "errores": [{ "campo": "<campo>", "problema": "<problema>" }]
}
```

| `type` (slug) | Código | Cuándo ocurre |
|---------------|-------:|---------------|
| | | |

## 5. Idempotencia

- **Operaciones que exigen la clave:** <…>
- **Quién genera la clave y cuándo:** <el cliente, una vez por intención; explicar con el caso del pago>
- **Qué hace el servidor si la clave se repite con el mismo cuerpo:** <…>
- **Qué hace si la clave se repite con otro cuerpo:** <…>
- **Cuánto tiempo guarda la clave:** <…>
- **Qué demostró la trampa de la ronda 3:** <números antes y después>

## 6. Caché y concurrencia

| Recurso | `Cache-Control` | `ETag` | Revalidación | Por qué esa frescura |
|---------|-----------------|--------|--------------|----------------------|
| Catálogo | | | | <¿cuántos segundos de precio desactualizado acepta el negocio? Esto lo decide el dueño de producto> |
| Pedido | | | | |

- **Control de concurrencia en modificaciones:** <`If-Match` → 412 / ninguno>

## 7. Decisiones no tomadas

| Decisión | Por qué no | Riesgo que asumen |
|----------|-----------|-------------------|
| Nivel 3 (hipermedia) | | |
| | | |

## 8. Migración desde la v0

- **Convivencia:** <cuánto tiempo siguen vivas las rutas v0>
- **Cómo se avisa a los clientes:** <…>
- **Equivalencias:**

| v0 | v1 |
|----|----|
| `GET /getProductos` | |
| `GET /getProducto?id=` | |
| `POST /crearPedido` | |
| `GET /pedido?id=` | |
| `POST /pagarPedido` | |
| `GET /cancelarPedido?id=` | |
| `POST /actualizarPedido` | |

## 9. Consumidores (ronda 4)

| Consumidor | Cómo se le expone la capacidad | Qué parte de este contrato reutiliza | Qué agrega |
|------------|--------------------------------|--------------------------------------|------------|
| App móvil | | | |
| Socio B2B | | | |
| Panel de reportes | | | |
| Sistema de envíos | | | |
| Agente de IA | | | |
