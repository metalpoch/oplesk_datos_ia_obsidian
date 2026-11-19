---
tipo: tema
modulo: "09 DuckDB y Parquet"
capa: dominio inicial
herramienta: DuckDB
resultado: "agrupar 5.000.000 de filas y cruzar dos Parquet, con el conteo igual antes y después del JOIN"
tiempo-lectura: 20 min
prerrequisitos: ["01 · DuckDB y Parquet"]
tags: [tema, duckdb, parquet]
---

# Consultar de verdad: agrupar, cruzar y comprobar sobre Parquet

![[duckdb.svg|64]]

> [!tip] En una frase
> Sobre un archivo Parquet grande se hacen tres cosas: **agrupar** para convertir cinco millones de filas en diez números, **cruzar** dos archivos para agregarles contexto, y **comprobar** con conteos y sumas que el resultado cuadra antes de creerle.

> [!info] Cómo leer este tema — 20 minutos
> El tema anterior enseñó a leer un archivo y convertirlo a Parquet; este lo da por sabido y no lo repite. Los dos archivos de práctica se generan con el primer bloque de código: son datos inventados, no hay nada que descargar. **Lo que NO está aquí:** las soluciones de los ejercicios.

## 🤔 Para qué sirve

Una pregunta de negocio normal: cuánto se vendió por ciudad, y si se vende distinto cuando hace calor. La respuesta está en dos archivos que nadie quiere cargar en memoria: el de ventas tiene 5.000.000 de filas y, para terminar mostrando diez números, pandas ocuparía varios cientos de megabytes. DuckDB lee el archivo por partes, toca solo las columnas que la consulta nombra, lo cruza con el segundo archivo y devuelve el resultado: el archivo nunca se carga completo.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Agrupar 5.000.000 de filas por ciudad y por franja de temperatura, sin cargar el archivo en memoria |
| Después | Cruzar los dos archivos y demostrar con conteos y una suma de control que el resultado no se infló |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Ejemplo |
|---|---|---|
| **`GROUP BY`** | cambia muchas filas por una fila por grupo | `GROUP BY ciudad` → 5.000.000 de filas se vuelven 10 |
| **`JOIN`** | pega las columnas de dos archivos usando una llave común | ventas + clima por `fecha` y `ciudad` |
| **`EXPLAIN`** | pide el plan sin ejecutar la consulta | qué archivo y qué columnas lee |

## 🧭 Qué no es

- **No es un reemplazo de PostgreSQL, ni sirve para escrituras concurrentes, ni para varios usuarios, ni para transacciones largas.** DuckDB lee archivos para analizarlos; PostgreSQL es donde la aplicación guarda y escribe datos. Si dos procesos escriben el mismo archivo, si hacen falta permisos por usuario o si una transacción dura minutos, la herramienta correcta es PostgreSQL.
- **No es un cargador.** Un `JOIN` no significa "traer todo y filtrar después": el motor decide el orden de lectura según el tamaño de cada archivo.

## 🧠 Conceptos

1. **Agrupar es cambiar filas por números** — y el motor columnar lee solo las columnas que la consulta nombra, por eso agrupar 5.000.000 de filas tarda milisegundos.
2. **Cruzar agrega columnas, pero puede multiplicar filas** — si la llave no es única en uno de los dos archivos, el `JOIN` devuelve más filas de las que había y el total se infla sin avisar.
3. **Un resultado se cree cuando cuadra** — conteo antes, conteo después y suma de control; si los tres coinciden, el número es defendible.

## 📖 Teoría

### 1. Los dos archivos de práctica y una primera cuenta

El bloque reutiliza `con`, la conexión del tema anterior. Crea un archivo grande, deriva de él uno chico y muestra los tres números que sirven de control para todo lo demás.

```python
import duckdb

con = duckdb.connect()

con.sql("""
    COPY (
        SELECT DATE '2024-01-01' + CAST((i // 5000) % 1000 AS INTEGER) AS fecha,
               (['Bogotá', 'Lima', 'Santiago', 'Buenos Aires', 'Ciudad de México', 'Quito', 'Montevideo', 'San José', 'Panamá', 'La Paz'])[1 + CAST((i // 500) % 10 AS INTEGER)] AS ciudad,
               (['Lácteos', 'Bebidas', 'Limpieza', 'Panadería'])[1 + CAST((i // 50) % 4 AS INTEGER)] AS categoria,
               CAST(1 + i % 7 AS INTEGER) AS unidades,
               round(5 + (hash(i) % 100000) / 1000.0, 2) AS importe
        FROM range(5000000) t(i)
    ) TO 'ventas.parquet' (FORMAT parquet)
""")

con.sql("""
    COPY (SELECT DISTINCT fecha, ciudad,
                 round(18 + (hash(CAST(fecha AS VARCHAR) || ciudad) % 120) / 10.0, 1) AS temp_max
          FROM 'ventas.parquet') TO 'clima.parquet' (FORMAT parquet)
""")

con.sql("SELECT count(*) FROM 'ventas.parquet'").show()                 # 5.000.000
con.sql("SELECT count(*) FROM 'clima.parquet'").show()                  # 10.000
con.sql("SELECT round(sum(importe), 2) FROM 'ventas.parquet'").show()    # 274961323.66
```

`ventas.parquet` queda con 5.000.000 de filas y cinco columnas. `clima.parquet` se arma con los pares `fecha` y `ciudad` que existen en ventas: mil días por diez ciudades, 10.000 filas, así el cruce cubre todas las ventas. La suma `274961323.66` es el control de todo el tema: cualquier agrupación o cruce posterior tiene que seguir sumando lo mismo.

### 2. Agrupar, y por qué es rápido

```python
con.sql("""
    SELECT ciudad, count(*) AS lineas, round(sum(importe), 2) AS ingresos
    FROM 'ventas.parquet'
    GROUP BY ciudad ORDER BY ingresos DESC
""").show()
```

Devuelve diez filas, de `27.528.896,54` (Montevideo) a `27.451.776,66` (Quito), con 500.000 líneas cada una, en 14 milisegundos. La razón es que Parquet guarda por columna y comprimido: el motor lee `ciudad` e `importe`, descarta las otras tres columnas y reparte el trabajo entre varios núcleos. No hay índice que crear ni tabla que cargar antes. El costo aparece en otro lado: cuando las filas salen del motor hacia Python. La misma tabla, tres consultas, medidas en una computadora común con el archivo en caché:

| Consulta | Filas que salen del motor | Tiempo |
|---|---|---|
| `SELECT sum(importe) FROM 'ventas.parquet'` | una sola | ~11 ms |
| `SELECT ciudad, importe FROM 'ventas.parquet'` con `.fetchall()` | 5.000.000 con 2 columnas | ~3.600 ms |
| `SELECT * FROM 'ventas.parquet'` con `.fetchall()` | 5.000.000 con 5 columnas | ~7.000 ms |

Los milisegundos exactos cambian con la máquina; el orden de magnitud, no. La regla útil no es "nunca uses `*`", es **saber cuántas columnas salen del motor**: dentro de una agregación DuckDB descarta las columnas que no usa, y `SELECT count(*) FROM (SELECT * FROM 'ventas.parquet')` tarda 3 milisegundos porque poda las cinco antes de leer. Con `.df()` o un `COPY ... TO` pagas las columnas que pediste, todas.

### 3. Cruzar dos archivos y comprobar que el conteo cuadra

El cruce agrega la temperatura a las ventas. La llave son dos columnas, porque una fecha sola aparece diez veces en `clima`, una por ciudad:

```python
con.sql("""SELECT count(*) FROM 'ventas.parquet' v
           JOIN 'clima.parquet' c ON v.fecha = c.fecha AND v.ciudad = c.ciudad""").show()   # 5.000.000
```

Antes del cruce había 5.000.000 de filas en ventas y 10.000 en clima; después hay 5.000.000. **El conteo no cambió, y esa es la prueba.** Si el `JOIN` hubiera duplicado filas, el número sería mayor. Con el conteo en orden, el cruce ya responde:

```python
con.sql("""
    SELECT CASE WHEN c.temp_max >= 25 THEN 'calor' ELSE 'fresco' END AS franja,
           count(*) AS lineas, round(sum(v.importe), 2) AS ingresos
    FROM 'ventas.parquet' v
    JOIN 'clima.parquet' c ON v.fecha = c.fecha AND v.ciudad = c.ciudad
    GROUP BY franja ORDER BY ingresos DESC
""").show()
```

Dos filas: `fresco` con 2.900.500 líneas y `159.524.308,7`, y `calor` con 2.099.500 líneas y `115.437.014,96`. La suma de las dos cifras es `274961323.66`, el control del apartado 1: el cruce no inventó ni perdió dinero. La tabla no dice nada sobre el clima real porque los datos son inventados; sirve para practicar con números que puedes verificar. **El cruce que se infla:** si la llave queda incompleta y se cruza solo por `fecha`, cada fila de ventas se encuentra con diez filas de clima: el conteo pasa de 5.000.000 a **50.000.000** y la suma a **2.749.613.236,6**, diez veces el total real. No hay mensaje de error; el número es creíble y está mal. Por eso el conteo va antes y después, siempre.

### 4. `EXPLAIN`: pedirle el plan en una línea

```python
con.sql("EXPLAIN SELECT ciudad, sum(importe) FROM 'ventas.parquet' GROUP BY ciudad").show()
```

`EXPLAIN` no ejecuta la consulta: devuelve el plan, un árbol de operaciones que se lee de abajo hacia arriba. Abajo de todo aparece una caja `PARQUET_SCAN` y dentro de esa caja una línea `Projections:` con las columnas exactas que el archivo leyó; aquí son `ciudad` e `importe`. Con `SELECT *` la lista tiene las cinco, y con `SELECT count(*)` no hay `PARQUET_SCAN` sino `COLUMN_DATA_SCAN` con `~1 row`, porque solo se leyeron los metadatos. Sirve para responder dos preguntas antes de pelear con un tiempo: **¿qué archivo y qué columnas está leyendo?** y **¿cuántas filas espera?** El plan trae una estimación de filas por caja, y esa estimación funciona como alarma: si espera 50 millones donde hay 5, la llave del `JOIN` está mal. No enseña optimización; enseña a mirar antes de adivinar.

## ⚠️ El error más común

> [!warning] `Binder Error: Ambiguous reference to column name "fecha" (use: "v.fecha" or "c.fecha")`
> **Qué significa:** los dos archivos tienen una columna `fecha` y la consulta pidió `fecha` sin decir de cuál; el motor no elige por ti. **Cómo se lee:** el mensaje nombra la columna y ofrece las dos salidas posibles, así que el problema está en el `SELECT` o en el `GROUP BY`, no en el `ON`. **Cómo se arregla:** alias en cada archivo y prefijo en la columna:
> ```python
> con.sql("""SELECT v.fecha, count(*) FROM 'ventas.parquet' v JOIN 'clima.parquet' c
>            ON v.fecha = c.fecha AND v.ciudad = c.ciudad GROUP BY v.fecha""").show()
> ```
> El otro error, sin mensaje y más caro, es cruzar con una llave incompleta: la consulta termina bien, devuelve una tabla y el total está multiplicado por diez. Ese no lo detecta el motor; lo detecta el conteo antes y después.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces en tu terminal o en VS Code, no aquí. Cuando termines, pega la salida en "Mi resultado".

1. Ejecuta el primer bloque y confirma los tres números: `5.000.000`, `10.000` y `274961323.66`. Si alguno no coincide, no sigas: el problema está en la generación.
2. Cambia el `GROUP BY` del apartado 2 para agrupar por `categoria` y corre `SELECT round(sum(ingresos), 2)` sobre ese resultado. Compara con `274961323.66` y pega los dos números.
3. Rompe el cruce a propósito: en la consulta del apartado 3 deja solo `ON v.fecha = c.fecha`. Anota el conteo y la suma, explica en una línea por qué quedaron multiplicados por diez, y corrige.

### Reto — opcional

1. Corre `EXPLAIN` sobre tres consultas —`SELECT count(*)`, `SELECT sum(importe)` y `SELECT *`— y anota qué operador aparece en cada plan: `PARQUET_SCAN` con `Projections:` o `COLUMN_DATA_SCAN`.
2. Mide con `time.perf_counter()` alrededor de `.fetchall()` las consultas del apartado 2 y explica la diferencia **sin escribir segundos**: habla de columnas leídas y de filas devueltas.

El reto no es obligatorio. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] El `GROUP BY` por ciudad devuelve diez filas y esas diez cifras suman `274.961.323,66`
- [ ] El conteo del `JOIN` por `(fecha, ciudad)` da `5.000.000`, y sabes explicar por qué cruzar solo por `fecha` lo multiplica a `50.000.000`
- [ ] Corriste `EXPLAIN` y ubicaste en el plan qué columnas lee el archivo
- [ ] Puedes decir en una frase qué problema tiene una escritura concurrente y a qué motor le corresponde

## 🔗 Para profundizar

- [DuckDB — GROUP BY](https://duckdb.org/docs/1.5/sql/query_syntax/groupby) — reglas de agrupación y `GROUP BY ALL`
- [DuckDB — FROM y JOIN](https://duckdb.org/docs/1.5/sql/query_syntax/from) — tipos de cruce y sintaxis `USING`
- [DuckDB — EXPLAIN](https://duckdb.org/docs/1.5/guides/meta/explain) — cómo leer el plan y qué significa cada operador

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `IO Error: No files found that match the pattern "ventas.parquet"` | el proceso corre en otra carpeta | ruta absoluta, o ejecutar desde la carpeta del archivo |
| `Binder Error: Referenced column "importee" not found in FROM clause!` | nombre mal escrito | leer la línea `Candidate bindings` del mensaje |
| `Binder Error: Ambiguous reference to column name "fecha"` | las dos tablas tienen esa columna | prefijo con alias: `v.fecha` |
| `Binder Error: column "ciudad" must appear in the GROUP BY clause…` | se pidió una columna que no se agrupó | agregarla al `GROUP BY` o envolverla en `sum()` o `count()` |
| El `JOIN` devuelve `50.000.000` de filas y la suma da diez veces más | la llave no es única en uno de los dos archivos | revisar el `ON` y comparar conteos antes y después |

## ➡️ Siguiente tema

**Neon, Postgres en la nube:** el mismo SQL, pero sobre una base donde varias aplicaciones escriben a la vez, donde hay permisos por usuario y donde el `JOIN` se apoya en índices en lugar de archivos.

---

## 📝 Mi resultado

> [!note] Tu espacio — pega el `GROUP BY` por categoría con las dos sumas de control, los conteos del apartado 3 antes y después del cruce, y el conteo inflado del ejercicio 3.
