---
tipo: anexo
modulo: "Anexos"
capa: familiaridad
herramienta: "pandas y Polars (tabla de traducción)"
resultado: "ninguno: es una chuleta de consulta"
tiempo-lectura: 6 min
prerrequisitos: []
tags: [anexo, pandas, polars, chuleta]
---

# Equivalencias Pandas y Polars

![[pandas.svg|64]] ![[polars.svg|64]]

> [!info] Esto no se lee de corrido
> Es una chuleta: busca la operación que necesitas, copia la línea y sigue. Si quieres el contexto y el porqué, está en [[Polars en 25 minutos]].

## 📊 La tabla

| Operación | pandas | polars | Cuándo importa la diferencia |
|---|---|---|---|
| Leer CSV | `pd.read_csv("ventas.csv")` | `pl.read_csv("ventas.csv")` | En 2.0 `read_csv` es `scan_csv().collect()`; para archivos grandes usa `scan_csv` y `.collect()` al final |
| Filtrar filas | `df[df["total"] > 100]` | `df.filter(pl.col("total") > 100)` | En pandas, combinar condiciones pide paréntesis: `df[(a) & (b)]`; en polars se encadenan `.filter()` |
| Seleccionar columnas | `df[['fecha', 'total']]` | `df.select("fecha", "total")` | El doble corchete de pandas es el error de tipeo más común al pasar a polars |
| Crear columna | `df.assign(total=df["precio"] * df["cant"])` | `df.with_columns((pl.col("precio") * pl.col("cant")).alias("total"))` | En polars sin `.alias()` la columna se llama como la expresión, y te enteras tarde |
| Agregar por grupo | `df.groupby("categoria").agg(total=("total", "sum"))` | `df.group_by("categoria").agg(pl.col("total").sum().alias("total"))` | `groupby` vs `group_by`; pandas acepta nombres en `agg`, polars exige `.alias()` |
| Ordenar | `df.sort_values("total", ascending=False)` | `df.sort("total", descending=True)` | El parámetro se llama distinto y el valor por defecto es ascendente en los dos |
| Join | `df.merge(clima, on="fecha", how="left")` | `df.join(clima, on="fecha", how="left")` | Desde Polars 2.0 el join **no preserva el orden**: agrega `maintain_order="left"` si dependes de él |
| Unión de tablas | `pd.concat([a, b], ignore_index=True)` | `pl.concat([a, b])` | En polars el índice no existe; y `how="horizontal"` ahora exige la misma cantidad de filas |
| Valores nulos | `df["total"].isna()` | `df["total"].is_null()` | En polars `NaN` y `null` son cosas distintas: para floats hay `is_nan()` aparte |
| Tipos de fecha | `pd.to_datetime(df["fecha"], errors="coerce")` | `df.with_columns(pl.col("fecha").str.to_date("%d/%m/%Y", strict=False))` | Desde 2.0 no se puede `cast` de texto a fecha: `str.to_date` es el único camino |
| Renombrar | `df.rename(columns={"monto": "total"})` | `df.rename({"monto": "total"})` | Si te olvidas `columns=` en pandas, te renombra el índice y no falla |
| Contar únicos | `df["categoria"].nunique()` | `df["categoria"].n_unique()` | Un guion bajo de diferencia: `nunique` vs `n_unique` |

## 🔁 El mismo pipeline, en los dos motores

Leer, castear la fecha, unir el clima, filtrar y agregar. La misma tarea, dos veces:

```python
# pandas
import pandas as pd
v = pd.read_csv("ventas.csv")
v["fecha"] = pd.to_datetime(v["fecha"], errors="coerce", dayfirst=True)
c = pd.read_csv("clima.csv", parse_dates=["fecha"])
tabla = (v.merge(c[['fecha', 'temp']], on="fecha", how="left")
          .query("total > 0")
          .groupby("categoria", as_index=False)
          .agg(total=("total", "sum"), temp=("temp", "mean"))
          .sort_values("total", ascending=False))
```

```python
# polars
import polars as pl
v = pl.read_csv("ventas.csv").with_columns(
        pl.col("fecha").str.to_date("%d/%m/%Y", strict=False))
c = pl.read_csv("clima.csv").with_columns(pl.col("fecha").str.to_date("%Y-%m-%d"))
tabla = (v.join(c.select("fecha", "temp"), on="fecha", how="left", maintain_order="left")
          .filter(pl.col("total") > 0)
          .group_by("categoria")
          .agg(pl.col("total").sum().alias("total"), pl.col("temp").mean().alias("temp"))
          .sort("total", descending=True))
```

Compara el resultado con dos cifras, no con el archivo entero: cantidad de filas y suma de `total`. Si esas dos coinciden, la traducción está bien.

## ↔️ Lo que existe en uno y no en el otro

- **Índice**: pandas tiene `set_index`, `.loc` y `.iloc`; polars no tiene índice, se filtra por columna.
- **`apply` por fila**: en pandas es natural y lento; en polars se escribe como expresión, y si necesitas Python puro usas `map_elements` (más lento, mismo consejo).
- **Pivot**: `df.pivot_table(...)` en pandas es `df.pivot(on=, index=, values=, aggregate_function=)` en polars.
- **Window**: `groupby().transform()` en pandas es `pl.col(...).over(...)` en polars.
- **Plan de ejecución**: en polars puedes verlo con `.explain()`; pandas ejecuta al momento y no hay plan que mostrar.
- **Gráficos**: pandas trae `.plot()` con matplotlib; polars no grafica, se pasa a pandas o a Plotly/Altair.

## ⏱️ Cuándo la diferencia importa de verdad

- **Cuando el orden de las filas es parte del resultado** (un `.head()`, un `zip`, dos tablas alineadas por posición). En pandas el orden se conserva casi siempre; en Polars 2.0 no está garantizado.
- **Cuando hay texto sucio**: los tipos se infieren distinto, y una columna de montos con coma puede quedar como texto en un motor y como número en el otro.
- **Cuando comparas resultados entre motores**: la misma consulta puede devolver el mismo total con distinto orden y distintos tipos. Compara cifras, no `df.equals()`.
- **Cuando el volumen es chico**: con 50 k filas las dos APIs tardan lo mismo; elige por comodidad, no por velocidad.

## ⚠️ Tres trampas de traducción

1. **El join sin orden.** Un port de 1.x a 2.x compila y corre: solo devuelve las filas en otro orden. Si después haces `.head()` o alineas por posición, el resultado cambia sin error.
2. **El casteo de fecha.** `pl.col("fecha").cast(pl.Date)` falla con `InvalidOperationError` desde 2.0. Se usa `.str.to_date(formato, strict=False)` y con `strict=False` las que no parsean quedan en `null`, no explotan.
3. **El `None` que no es `NaN`.** En pandas, `isna()` cubre los dos. En polars, si la columna es float, `is_null()` no ve los `NaN`: revisa las dos cosas antes de contar faltantes.

## 📌 Versiones fijadas

Al escribir esto: **pandas 3.1.1** y **Polars 2.1.0**. Las dos tuvieron cambios que rompen código: pandas 3 infiere texto como dtype `str` (ya no `object`) y Polars 2 cambió el orden de los joins y sacó los casteos de texto a fecha. Fíjalas en `requirements.txt`.

## 🔄 Cómo traducir un snippet ajeno

1. Pregunta primero de qué motor es: si dice `.groupby(` o `df[...]`, es pandas.
2. Traduce la **operación**, no la sintaxis: "agrupar y sumar" es lo mismo en los dos.
3. Cambia `groupby` → `group_by`, `sort_values` → `sort`, `isna` → `is_null`, `nunique` → `n_unique`.
4. Si el snippet hace un join y después usa el orden, agrega `maintain_order="left"`.
5. Verifica con una cifra: el total antes y después tiene que dar igual.

## ✅ Checklist antes de decir "ya lo traduje"

- [ ] Los dos motores devuelven la misma cantidad de filas
- [ ] El total de la columna de montos coincide en los dos
- [ ] Las fechas quedaron como fecha, no como texto
- [ ] Si hay join, sé si dependo del orden o no

## 🔗 Ver también

- [[Polars en 25 minutos]] — el porqué, la comparación cronometrada y la advertencia de la 2.0
- La versión pandas a fondo: [[01 · Pandas, leer y perfilar]] · [[02 · Pandas, limpiar y reportar]]
- [Viniendo de pandas](https://docs.pola.rs/user-guide/migration/pandas/) — la tabla oficial, más completa que esta
- [[Chuleta de errores]] — cuando la traducción falla y no sabes por qué
