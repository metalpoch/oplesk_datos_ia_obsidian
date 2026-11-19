---
tipo: anexo
modulo: "Anexos"
capa: familiaridad
herramienta: Polars
resultado: "ninguno: es una muestra. Si te interesa, el cronómetro propio es el ejercicio"
tiempo-lectura: 15 min
prerrequisitos: []
tags: [anexo, polars, pandas, motores]
---

# Polars en 25 minutos

![[polars.svg|64]] ![[pandas.svg|64]]

> [!info] Esto es material de consulta
> Es una **muestra**: la misma tarea que ya sabes hacer en pandas, con otro motor, para que sepas que existe, para qué sirve y dónde seguir. No reemplaza nada del curso. Si te engancha, al final está por dónde seguir.

## 🎁 Qué te llevas de acá

1. **Qué resuelve** — las mismas operaciones de tabla, con un motor columnar, en paralelo y con evaluación perezosa.
2. **Cuándo NO usarla** — cuando el resto de tu código, tu equipo o tus tutoriales son pandas.
3. **Un "hola mundo"** — cinco líneas y ya estás haciendo lo mismo que en pandas.
4. **El error más común** — desde la 2.0 los joins no te devuelven las filas en el orden que esperabas, y no avisa.
5. **Para profundizar** — la documentación oficial, que es de las mejores del ecosistema.

## 🧭 Por qué pandas es la base y Polars la alternativa

Pandas se usa mucho más, y no es una impresión: según el relevamiento del curso (descargas de PyPI en 30 días, consultadas el 16-nov-2026), **pandas tiene ~688 M de descargas y Polars ~65 M**: unas **10,5 veces** más. En estrellas de GitHub están casi empatados, que es la trampa clásica: popularidad en GitHub no es base instalada.

El matiz honesto: las descargas no son usuarios. A pandas lo arrastra medio ecosistema como dependencia, y Polars se instala más veces de forma directa. La diferencia real es menor a 10×, pero **no se invierte**.

Por eso el curso te enseña pandas: lo vas a encontrar en cada tutorial, en cada respuesta de StackOverflow, en cada notebook heredado y en casi todo lo que te escriba un agente. Polars entra acá para que sepas que existe. **Qué hace distinto, sin humo:** guarda en formato columnar (Arrow), usa todos los núcleos de tu máquina y puede armar un plan antes de ejecutar (API `lazy`). Pandas opera con NumPy, en un solo hilo, y ejecuta a medida que lees.

## ⏱️ La comparación honesta

> [!warning] Con 50.000 filas no vas a ver diferencia
> El dataset del curso tiene 50 k filas. En los dos motores eso tarda menos de lo que tardas en pestañear: si cronometras con 50 k y Polars "gana", estás midiendo ruido. **Para que el cronómetro diga algo hay que generar 10 millones de filas.**

Y no te doy mis números: dependen de tu laptop y de si pandas está usando Arrow para texto. Mide los tuyos:

```python
import time, numpy as np, pandas as pd, polars as pl

N = 10_000_000
rng = np.random.default_rng(42)
datos = {
    "categoria": rng.choice(np.array(["bebidas", "panaderia", "abarrotes", "limpieza"]), N),
    "total": rng.gamma(2.0, 1.5, N).round(2),
}
t0 = time.perf_counter(); res_p = pd.DataFrame(datos).groupby("categoria")["total"].sum(); tp = time.perf_counter() - t0
t0 = time.perf_counter(); res_r = pl.DataFrame(datos).group_by("categoria").agg(pl.col("total").sum()); tr = time.perf_counter() - t0

print(f"pandas {tp:.2f}s · polars {tr:.2f}s · ratio {tp/tr:.1f}x")
assert len(res_p) == res_r.height, "no agruparon igual: comparar tiempos no significa nada"
```

Ese `assert` no es decoración: primero que cuadre, después que corra.

## 📋 Cinco operaciones que se hacen distinto

| Operación | pandas | polars |
|---|---|---|
| Leer CSV | `pd.read_csv("ventas.csv")` | `pl.read_csv("ventas.csv")` |
| Filtrar filas | `df[df["total"] > 100]` | `df.filter(pl.col("total") > 100)` |
| Agregar por grupo | `df.groupby("categoria")["total"].sum()` | `df.group_by("categoria").agg(pl.col("total").sum())` |
| Join | `df.merge(clima, on="fecha", how="left")` | `df.join(clima, on="fecha", how="left")` |
| Elegir columnas | `df[['fecha', 'total']]` | `df.select("fecha", "total")` |

La diferencia de fondo no es la sintaxis: en Polars escribes una **expresión** (`pl.col(...)`) y el motor decide cómo ejecutarla. Por eso `group_by` no se llama `groupby` y `select` reemplaza al doble corchete. El mismo cálculo completo, con nombre de salida en cada motor:

```python
# pandas
resumen = (dfp.groupby("categoria", as_index=False)
               .agg(total=("total", "sum"), tickets=("total", "size"))
               .sort_values("total", ascending=False))
```

```python
# polars
resumen = (dfr.group_by("categoria")
              .agg(pl.col("total").sum().alias("total"), pl.col("total").len().alias("tickets"))
              .sort("total", descending=True))
```

## ⚠️ La advertencia de versión: la 2.0 cambió el orden

Esto es lo que rompe material viejo: **desde la 2.0 los joins no te devuelven las filas en el orden que esperabas**. La causa no es el join: la API `lazy` pasó a usar el motor de streaming por defecto, y ese motor **no garantiza el orden de las filas** en las operaciones que no lo necesitan (joins, `group_by`, `unpivot`). No hay error ni warning: hay otro orden.

```python
izq = pl.LazyFrame({"k": [0, 1, 2], "l": ["a", "b", "c"]})
der = pl.LazyFrame({"k": [2, 1, 0], "r": ["x", "y", "z"]})

izq.join(der, on="k", how="left").collect()                           # ya no respeta izq
izq.join(der, on="k", how="left", maintain_order="left").collect()    # sí
```

Tres formas de no comerte el cambio, de la más fina a la más bruta:

- `maintain_order="left"` en el join, cuando el orden importa de verdad.
- `sort(...)` explícito antes de cualquier `.head()`, `iloc` o `zip` que asuma orden.
- `pl.Config.set_engine_affinity("in-memory")` (o `POLARS_ENGINE_AFFINITY=in-memory`) para volver al comportamiento anterior en todo el proceso.

Y una segunda trampa de la misma versión, que te va a tocar con los datos sucios de la tienda: **ya no se puede castear texto a fecha**. `pl.col("fecha").cast(pl.Date)` ahora falla con `InvalidOperationError`; se usa `pl.col("fecha").str.to_date("%d/%m/%Y", strict=False)`. Fija la versión en `requirements.txt` (`polars==2.1.0`, la vigente al escribir esto) y, si lees un tutorial de la 1.x, desconfía de los joins y de los casteos de fecha.

## 🚫 Cuándo NO usar Polars

- **Cuando el resto del stack es pandas**: scikit-learn, Seaborn, Plotly y casi todo el ecosistema comen DataFrames de pandas; convertir de ida y vuelta cuesta más de lo que ganas.
- **Cuando el dato es chico**: 50 k filas no justifican aprender otra API.
- **Cuando el orden de las filas tiene que ser el que era**, y no quieres acordarte de `maintain_order` en cada join.
- **Cuando copias un snippet**: lo que te devuelve un agente es pandas el 90 % de las veces, y el equipo mantiene lo que entiende.

## ✅ Cómo saber que lo probaste

- [ ] Corriste el script de 10 M de filas y anotaste tus dos tiempos
- [ ] El `assert` de grupos pasó: los dos motores agruparon igual
- [ ] Hiciste un join y explicas por qué ahora hace falta `maintain_order`
- [ ] Puedes decir en una frase cuándo elegirías pandas aunque Polars sea más rápido

## 🔗 Dónde seguir

- Anexo del vault: [[Equivalencias Pandas y Polars]] — la chuleta para tener al lado
- El otro motor del curso: [[01 · DuckDB y Parquet]] — mismo principio columnar, con SQL
- [Guía de actualización a la 2.0](https://docs.pola.rs/releases/upgrade/2/) — lee la sección de orden de filas antes de portar código
- [Viniendo de pandas](https://docs.pola.rs/user-guide/migration/pandas/) — la tabla de traducción oficial
- **[Documentación oficial de Polars](https://docs.pola.rs/)** — empieza por el *user guide*: es corto y está bien escrito
