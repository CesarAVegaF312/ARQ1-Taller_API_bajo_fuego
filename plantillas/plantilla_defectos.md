# Defectos de la API v0

> Plantilla. Un bloque por defecto. Cada defecto debe citar una petición del historial de la consola (el número `#n`) y copiar la parte de la respuesta que lo demuestra.
> Severidad: **Alta** (pierde dinero o datos), **Media** (rompe a clientes o intermediarios), **Baja** (dificulta operar o depurar).

**Grupo:** <nombre e integrantes>

## Resumen

| ID | Endpoint | Defecto | Severidad | Evidencia | Decisión v1 que lo cierra |
|----|----------|---------|-----------|-----------|---------------------------|
| D-01 | | | | #<n> | |
| D-02 | | | | #<n> | |

---

## D-01 · <título corto>

- **Endpoint:** `<MÉTODO> <ruta>`
- **Categoría:** <Verbo en la URI · GET con efecto · POST reintentable sin clave · Error escondido en un 200 · 500 por un error del cliente · Fuga de detalle interno · Validación ausente · Endpoint túnel · Sin caché ni validador · Colección sin límite · Crear sin 201 ni Location · Otro>
- **Nivel de Richardson del endpoint:** <0/1/2> — <por qué>
- **Severidad:** <Alta/Media/Baja>

**Evidencia (#<n>):**

```http
<MÉTODO> <ruta>
<encabezados>

<cuerpo>

HTTP/1.1 <código> <texto>
<encabezados relevantes>

<parte del cuerpo que demuestra el defecto>
```

**Impacto:** <qué le pasa al negocio o al cliente cuando esto ocurre en producción; si la ronda 2 lo midió, pongan el número>

**Corrección:** <cómo queda en la v1 y qué código, método o encabezado lo resuelve>

**Comprobación en la v1 (#<n>):** <número de la petición en la consola de la ronda 3 que muestra el comportamiento corregido>

---

## D-02 · <título corto>

<copiar el bloque anterior>
