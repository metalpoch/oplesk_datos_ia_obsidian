---
tipo: anexo
modulo: "Anexos"
capa: referencia
herramienta: "contrato de datos"
resultado: "el esquema que todos los temas usan igual"
tiempo-lectura: 8 min
prerrequisitos: []
tags: [anexo, esquema, datos, contrato]
---

# Esquema de datos

> [!important] Para qué existe esta nota
> Cuarenta notas hablan del mismo dataset. Si dos lo nombran distinto, el alumno llega a pandas y su código no encuentra la columna. **Acá está la única versión válida.** Si una nota contradice esta tabla, la nota está mal.

## 📄 El archivo principal: `ventas.csv`

Una tabla de ventas de una tienda de barrio, exportada "como salió del sistema".

| Columna | Tipo en el CSV | Ejemplo limpio | Suciedad deliberada |
|---|---|---|---|
| `id_venta` | texto | `V-000123` | filas duplicadas con el mismo id |
| `fecha` | texto | `15/03/2024` | dos formatos mezclados (`dd/mm/aaaa` e ISO `aaaa-mm-dd`), una fecha imposible (`31/02/2024`) y una ambigua (`03/04/2024`) |
| `producto` | texto | `Pan de queso` | acentos y ñ (el archivo viene en dos codificaciones) |
| `categoria` | texto | `Bebidas` | `bebidas`, `BEBIDAS`, `Bebidas ` con espacio final |
| `cantidad` | texto | `2` | valores vacíos |
| `precio_unit` | texto | `3.50` | en algunas filas viene con coma: `3,50` |
| `total` | texto | `7.00` | nulos donde no hay precio |
| `metodo_pago` | texto | `efectivo` | nulos |
| `cliente_id` | texto | `C-0456` | nulos, y ids huérfanos |

### La invariante que hace verificables los ejercicios

> [!warning] `total = cantidad × precio_unit` (redondeado a 2 decimales)
> Vale para **todas las filas que tienen precio y cantidad**. Es lo que permite que el alumno verifique su trabajo: si suma `total` y le da distinto que `sum(cantidad * precio_unit)`, hay filas sucias en el medio y eso es exactamente el hallazgo que buscamos.
>
> Si esta invariante no se cumple, los ejercicios de SQL y de pandas dan cifras distintas y la discusión se va en cuál está bien.

**Nunca cambies el nombre de una columna sin buscar quién la usa** (`Ctrl+Shift+F` en el vault).

## 🗂️ Los otros archivos

| Archivo | Columnas | Tema | Qué tiene de especial |
|---|---|---|---|
| `clima_caracas.csv` | `fecha`, `temp_max_c`, `lluvia_mm` | `01 · HTTP con httpx` | 90 días de Caracas, una fila por día, en ISO; la copia incluida cubre los mismos 90 días que las ventas |
| `precios_competencia.csv` | `producto`, `precio`, `stock` | `02 · Scraping responsable con BeautifulSoup` | 20 productos; 2 sin precio (producto agotado) |
| `ventas_latin1.csv` | las mismas que `ventas.csv` | `03 · Pandas, limpieza difícil` | guardado en `latin-1`, con la ñ; sirve para provocar `UnicodeDecodeError` |
| `ventas_limpias.csv` | las de `ventas.csv`, tipadas | `02 · Pandas, limpiar y reportar` · `01 · DuckDB y Parquet` | la salida de la limpieza: `fecha` es `datetime`, `total` es numérico |
| `ventas.parquet` | las de `ventas_limpias.csv` | `01 · DuckDB y Parquet` · `01 · Streamlit, primera app` | mismo contenido, formato columnar |

## 🚫 Nombres que NO se usan

Estos aparecían en borradores anteriores. Si los ves en una nota, está desactualizada:

| Mal | Bien | Por qué |
|---|---|---|
| `monto` | `total` | El importe de la línea. `monto` queda solo como palabra en prosa ("el monto de la venta") |
| `venta_id` | `id_venta` | Prefijo `id_` para todas las claves |
| `precio` | `precio_unit` | Es el precio **por unidad**; multiplicado por `cantidad` da `total` |
| `cant` | `cantidad` | |

> [!tip] `monto_total` sí aparece en el curso, a propósito
> En el tema de integración el alumno renombra una columna y el pipeline se rompe. `monto_total` es el nombre **viejo** de `total`, y aparece en la Chuleta de errores y en la nota de integración como el ejemplo del cambio. No es una inconsistencia: es la lección.

## 🔍 Cómo saber si dos notas se contradicen

1. Busca `precio_unit`, `id_venta` o `total` en todo el vault (`Ctrl+Shift+F`).
2. Si una nota usa `precio`, `venta_id` o `monto` **como nombre de columna**, corrígela.
3. Si aparece en prosa ("el monto de la venta", "los precios de la competencia"), está bien.

---

## 🔗 Relacionado

- [[01 · PostgreSQL, DDL y carga]] — donde el esquema se vuelve tabla
- [[01 · Pandas, leer y perfilar]] — donde el alumno descubre que todo entró como texto
- [[Chuleta de errores]] — qué dice el motor cuando el nombre no coincide
