---
tipo: tema
modulo: "06 Pandas"
capa: dominio inicial
herramienta: pandas
resultado: "ventas_diarias_clima.csv con una fila por fecha"
tiempo-lectura: 15 min
prerrequisitos: ["Pandas, limpiar y reportar"]
tags: [tema, pandas, groupby, merge]
---

# Pandas: groupby y merge con red de seguridad

![[pandas.svg|64]]

> [!tip] En una frase, antes de empezar
> **`groupby` junta las filas que comparten un valor y devuelve un número por grupo; `merge` pega dos tablas usando una columna en común, como un `VLOOKUP` de Excel.** Las dos te llevan de muchas filas a una respuesta.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y escribe `#duda` donde algo no te cierre; esas dudas son el arranque.
>
> **Lo que NO está acá:** la solución de la tabla. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

Tienes las ventas en un archivo y el clima en otro. En Excel harías un `VLOOKUP` de la fecha, arrastrarías la fórmula hasta la fila 50.000 y esperarías. Y cuando termina, el total de ingresos no coincide con el del archivo original: el clima tenía dos filas para el 15/03 y la venta de ese día se contó dos veces. Excel no avisó nada, porque el `VLOOKUP` siempre devuelve algo.

Del otro lado, la pregunta "¿cuánto vendimos por día?" no tiene una celda donde vivir: hay 365 filas que sumar por fecha, y a mano es una tabla dinámica nueva cada vez.

**Eso es lo que resuelven estas dos operaciones:** `groupby` convierte muchas filas en un resumen por grupo, y `merge` cruza dos tablas por una columna común. Y a diferencia del `VLOOKUP`, el `merge` puede avisarte si la llave estaba repetida: para eso está `validate`.

> [!note] Qué no es
> Pegar dos tablas una al lado de la otra sin mirar la llave no es un `merge`: eso es `concat`, y si el orden de las filas no coincide, pegas la temperatura del día equivocado. Y un `merge` sin `validate` no falla: duplica en silencio.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Construir una tabla diaria con ingresos, tickets, temperatura y lluvia, una fila por fecha |
| **Después** | Demostrar con un conteo que el merge no duplicó ni perdió filas |

La pregunta es de negocio: ¿se vende más cuando hace calor? Para responderla, dos tablas tienen que quedar en una, y la serie diaria no puede mentir.

## 🧠 Conceptos

1. **`groupby` con agregaciones nombradas** — una fila por grupo, cada columna con su operación y su nombre.
2. **`merge` con `validate=`** — la red que falla si la llave de la derecha no es única; un merge sin `validate` duplica en silencio.
3. **Granularidad verificada** — una fila por fecha se demuestra con `is_unique` y un conteo antes/después, no con la vista.

## 📖 Teoría

### 1. Groupby: una fila por fecha, dos números por fila

```python
diario = (ventas.groupby("fecha", as_index=False)
          .agg(ingresos=("total", "sum"),
               tickets=("id_venta", "nunique")))
print(diario.head())
```

`groupby("fecha")` junta las filas que comparten fecha. `.agg()` con nombres te deja elegir la operación de cada columna: `sum` para el dinero, `nunique` para tickets (un ticket puede tener varias líneas; si cada fila ya es un ticket, `count` alcanza). `as_index=False` deja `fecha` como columna y no como índice. El resultado tiene una fila por fecha porque `groupby` agrupa por esa llave.

**Por qué importa:** toda serie diaria, todo gráfico y toda comparación con el clima empiezan por una tabla en el grano correcto. Si el grano está mal, todo lo que viene después está mal.

### 2. Merge: la llave y la red de seguridad

```python
diario = diario.merge(clima, on="fecha", how="left", validate="m:1")
```

`how="left"` conserva todas las fechas de `diario`, tengan clima o no. `validate="m:1"` declara que muchas filas de la izquierda pueden coincidir con una de la derecha, y falla si `clima` repite una fecha. Si el clima viene por hora, primero se agrega a un registro por día (temperatura media, lluvia sumada) y después se cruza. Sin `validate`, una fecha repetida en el clima multiplica las filas de ventas: el ingreso del día se cuenta dos veces y la serie miente.

**Por qué importa:** el merge es el lugar donde una tabla limpia se vuelve una tabla falsa. `validate` convierte una duplicación silenciosa en un error que puedes ver.

### 3. Granularidad: una fila por fecha, demostrada

```python
antes = len(diario)
con_clima = diario.merge(clima, on="fecha", how="left", validate="m:1")
assert len(con_clima) == antes
assert con_clima["fecha"].is_unique
```

Un `assert` es una prueba que queda en el script y corre cada vez. `is_unique` verifica el grano final; la comparación de largos verifica que el `left join` no agregó ni perdió filas. Si usaras `how="inner"`, el largo cambiaría de forma legítima por las fechas sin clima: en ese caso compara contra el conjunto esperado, no contra el total original.

**Por qué importa:** "no dupliqué" no es una opinión, es un número que se muestra. El assert es la diferencia entre creer que la tabla está bien y saberlo.

Después de los asserts, conviene armar la misma tabla en Polars y cronometrar las dos versiones[^polars].

[^polars]: El cierre de este tema son 25-30 minutos de contraste: la misma tarea, resuelta en Pandas y en Polars, con **dos cronómetros** en pantalla. Vas a ver la diferencia de tiempo y te vas con una pregunta abierta: *¿por qué?*. Aquí no se responde: la respuesta llega en [[01 · DuckDB y Parquet]], y el mapa de equivalencias en [[Polars en 25 minutos]].

## ⚠️ El error más común

> [!warning] `pandas.errors.MergeError: Merge keys are not unique in right dataset`
> ```
> >>> diario.merge(clima, on="fecha", how="left", validate="m:1")
> pandas.errors.MergeError: Merge keys are not unique in right dataset; not a many-to-one merge
> ```
> **Qué significa:** la tabla de la derecha, `clima`, tiene más de una fila para alguna fecha. `m:1` espera una sola. El final del mensaje cambia con el `validate`: con `m:1` dice `not a many-to-one merge`; con `one_to_one` dice `not a one-to-one merge`. La causa es la misma. **Cómo se lee:** mira la derecha antes de tocar el merge: `clima["fecha"].duplicated().sum()` y las filas repetidas con `keep=False`. El 80 % de las veces es clima por hora o dos estaciones para el mismo día. **Cómo se arregla:** decide el grano del clima antes de cruzar: `clima.groupby("fecha", as_index=False).agg(temp=("temp", "mean"), lluvia=("lluvia", "sum"))`, y recién ahí mergea con `validate="m:1"`. Si la izquierda también debe ser única, `validate="1:1"` es más estricto.
>
> **El merge sin `validate`:** no falla, duplica. Si el 15/03 tiene 2 filas de clima y una fila diaria con $1.000, el merge devuelve 2 filas y el ingreso de ese día aparece dos veces: el total del período queda inflado. Compara `len()` antes y después: ese número es el delator.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Arma la tabla diaria con `groupby` y agregaciones nombradas: ingresos y tickets. Imprime `head()` y `len()`, y explica por qué el largo coincide con la cantidad de fechas únicas. (8 min)
2. Carga el clima, revisa su grano con `duplicated()` sobre `fecha`, agrégalo si hace falta y mergea con `validate="m:1"`. Guarda `ventas_diarias_clima.csv`. (9 min)
3. Agrega los asserts de granularidad y de conteo antes/después. Después quita `validate` en una copia, provoca la duplicación y muestra cómo cambia `len()`; vuelve a ponerlo. (8 min)

### Reto — opcional

1. Compara el ingreso total de la tabla diaria con el de las ventas crudas: tienen que coincidir al centavo. Si no coinciden, encuentra el paso que los separó.
2. Repite la tabla diaria en Polars, cronometra las dos versiones y escribe una frase con lo que viste. El *por qué* no va todavía: se responde más adelante, en [[01 · DuckDB y Parquet]].

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `ventas_diarias_clima.csv` tiene una fila por fecha y las columnas de ingresos, tickets, temperatura y lluvia
- [ ] El largo antes y después del `left join` es el mismo
- [ ] El `assert` de `is_unique` está en el script, no solo en tu cabeza
- [ ] Puedes explicar qué significa `m:1` y por qué el merge falló o pasó
- [ ] Puedes explicar en una frase por qué usaste pandas y no una hoja de cálculo

## 🔗 Para profundizar

- [pandas: groupby](https://pandas.pydata.org/docs/user_guide/groupby.html) — agregaciones nombradas y `as_index`
- [pandas: merging](https://pandas.pydata.org/docs/user_guide/merging.html) — tipos de join y validación
- [pandas: `merge`](https://pandas.pydata.org/docs/reference/api/pandas.merge.html) — el parámetro `validate` con sus combinaciones

## 🧭 Problemas frecuentes

- **`MergeError` por llave repetida** → revisa la derecha con `duplicated()`; agrega el clima a una fila por fecha antes de cruzar.
- **El ingreso se duplicó después del merge** → te faltó `validate` o la derecha tenía fechas repetidas. Compara `len()` antes y después.
- **`KeyError: 'fecha'`** → una tabla tiene la fecha como índice o con otro nombre. Usa `reset_index()` o renombra antes del merge.
- **El `assert` de `is_unique` falla** → el problema puede estar en la izquierda: revisa si `diario` ya traía fechas repetidas antes del merge.

## ➡️ Siguiente tema

Vas a cargar un archivo con ñ y fechas ambiguas, y a decidir con evidencia en vez de adivinar: [[03 · Pandas, limpieza difícil]].

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (una captura, un conteo, el texto del error que no pudiste resolver). En el siguiente tema se retoma lo que quedó acá.
