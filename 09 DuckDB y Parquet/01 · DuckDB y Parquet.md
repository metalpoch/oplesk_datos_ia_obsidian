---
tipo: tema
modulo: "09 DuckDB y Parquet"
capa: familiaridad
herramienta: DuckDB
resultado: "ventas.parquet verificado con 3 cifras contra la consulta SQL"
tiempo-lectura: 15 min
prerrequisitos: ["Pandas: fechas y groupby", "SQL: GROUP BY"]
tags: [duckdb, parquet, sql]
---

# DuckDB y Parquet: SQL sobre tus archivos, sin servidor

![[duckdb.svg|64]]

> [!tip] En una frase, antes de empezar
> **DuckDB es un programa que habla SQL sobre tus archivos —el CSV, el Parquet— sin servidor, sin usuario y sin nada que instalar aparte.** Parquet es la forma de guardar ese archivo para que se lea rápido: el mismo dato, mejor empaquetado.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y, cuando algo no te cierre, escribe `#duda` en esa línea: esas dudas marcan por dónde empezar. **Lo que NO está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

Necesitas tres cifras del CSV y ya sabes escribir el `GROUP BY`. Pero usar SQL, hasta ahora, significa montar un servidor: instalar PostgreSQL, crear un usuario, una base, dejarlo corriendo y acordarte de levantarlo cada vez. Para tres consultas sobre un archivo, eso es un proyecto.

La otra salida es pandas, y para una pregunta que en SQL son tres líneas escribes quince, con la sintaxis a medio recordar. Encima, cada corrida lee el archivo completo, aunque solo te interese una columna.

**Eso es lo que resuelve DuckDB:** SQL donde está el archivo, dentro de tu proceso de Python. Y Parquet resuelve la otra mitad: guardar el mismo dato por columnas, comprimido y con los tipos declarados, para que la próxima lectura toque menos bytes.

> [!note] Qué no es
> DuckDB no es un servidor y no reemplaza a PostgreSQL: es un motor para analizar lo que tienes en disco, de a una persona. Si dos personas escriben a la vez, si necesitas permisos por usuario o si la app tiene que estar arriba mientras se escribe, esto no es la herramienta.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Consultar `ventas.csv` con SQL desde DuckDB, sin instalar ni levantar ningún servidor, y sacar tres cifras del negocio |
| **Después** | Convertir ese CSV en `ventas.parquet` y comprobar que las tres cifras son idénticas a las de la consulta original |

**La pregunta pendiente se responde aquí.** En el tema de Polars se cronometró el mismo cálculo y quedó anotado por qué tardaba distinto que en Pandas. La respuesta no era la librería: era el **motor columnar**. Y no se contesta con una opinión: se mide.

## 🧠 Conceptos

Solo tres. Si te llevas estos tres, el tema está cubierto.

1. **SQL sobre archivos, sin servidor** — DuckDB es una librería que habla SQL, no un servicio que hay que levantar. Sirve para lo que tienes en disco; no para varias personas escribiendo a la vez.
2. **Parquet no es un CSV comprimido** — guarda por columnas, comprime y recuerda el tipo de cada dato. Pesa menos, y el motor lee menos.
3. **El porqué: columnar, lazy y multihilo** — lee solo las columnas que pides, no trabaja hasta que pides el resultado y usa varios núcleos.

## 📖 Teoría

### 1. SQL donde está el archivo
```python
import duckdb

con = duckdb.connect()          # en memoria: no hay servidor, no hay puerto
con.sql("SELECT count(*) AS filas FROM read_csv_auto('ventas.csv')").show()
```

`connect()` abre un motor dentro de tu proceso de Python: sin usuario, sin contraseña, sin servicio que se caiga. `read_csv_auto` adivina el esquema leyendo unas filas, y adivinar tiene un costo: si se equivoca, la columna queda con el tipo equivocado y todo lo que hagas después miente. Cuando el archivo importa, **declaras los tipos** en la misma llamada, con `columns = {...}`, en vez de confiar en el adivinador (abajo lo ves completo).

**Cuándo NO usarlo.** DuckDB es un motor *embebido*: un proceso, un archivo, una persona. No lo uses si dos personas escriben a la vez, si necesitas permisos por usuario, si una transacción dura minutos o si la app tiene que estar arriba mientras se escribe. Para eso está PostgreSQL (el módulo de bases de datos): DuckDB lee y analiza, no es el sistema de registro.

> **Fija la versión antes de empezar.** Hoy `pip install duckdb` trae la 2.0 (salió el 21/10/2026) y este material está escrito para la línea 1.5. En `requirements.txt` va `duckdb==1.5.6` —o la 1.x exacta que tengas— y no la cambias a mitad del curso.

### 2. Parquet: el mismo dato, mejor guardado
```python
con.sql("""
    COPY (SELECT * FROM read_csv('ventas.csv', header = true,
        columns = {'fecha': 'VARCHAR', 'categoria': 'VARCHAR', 'precio': 'VARCHAR', 'cantidad': 'INTEGER'}))
    TO 'ventas.parquet' (FORMAT parquet, COMPRESSION zstd)
""")
```

Y se consulta igual, sin convertir nada: `con.sql("SELECT categoria, sum(precio) FROM 'ventas.parquet' GROUP BY 1")`. - **Columnar:** el CSV guarda fila por fila; Parquet, columna por columna. Si pides `precio`, el motor no toca `fecha` ni `categoria`.
- **Comprimido:** zstd achica sin perder nada. Menos bytes en disco es menos bytes leídos.
- **Con tipos:** el archivo ya sabe que `cantidad` es entero. No hay que volver a adivinar en cada lectura.

### 3. El porqué: motor columnar, ejecución lazy y multihilo
**Columnar** contesta la pregunta pendiente: si guardas por columnas, sumar `precio` recorre un vector de números, no una fila con texto en el medio. Eso es lo que hace rápidos a Polars, a DuckDB y a todo motor analítico moderno; Pandas, en muchas operaciones, sigue yendo fila por fila.

**Lazy** significa que `con.sql(...)` **todavía no ejecutó nada**: devuelve una relación y el trabajo arranca cuando pides el resultado (`.show()`, `.df()`, `.fetchall()`). Por eso puede descartar columnas y empujar el filtro antes de leer. Por eso tampoco sirve comparar "cuánto tardó" sin mirar qué leyó.

**Multihilo:** usa varios núcleos sin que se lo pidas. `PRAGMA threads` te dice cuántos, y `EXPLAIN` te muestra el plan real de la consulta.

## ⚠️ El error más común

> [!warning] `Conversion Error: Could not convert string '3,50' to DOUBLE`
> ```
> Conversion Error:
> Could not convert string '3,50' to DOUBLE
> ```
> **Qué significa:** pediste un número y en esa fila hay texto: el precio venía con coma decimal —como lo escribe una persona en Venezuela— y el motor no la lee como separador decimal. **Cómo se lee:** primero, qué columna estás sumando o casteando; después, su tipo real con `DESCRIBE SELECT * FROM read_csv_auto('ventas.csv');`. Si dice `VARCHAR` donde esperabas `DOUBLE`, el problema no es la consulta: es la lectura. **Cómo se arregla:** lee esa columna como texto y normaliza antes de castear:
> ```sql
> SELECT CAST(replace(precio, ',', '.') AS DOUBLE) AS precio
> FROM read_csv('ventas.csv', header = true, columns = {'precio': 'VARCHAR'});
> ```
> Y cuidado con el atajo: `try_cast(precio AS DOUBLE)` **no falla**, devuelve `NULL`. El total da un número creíble y equivocado. Un error que no se ve es peor que uno que sí.

Este no es un error de adorno: es el que se repite en el proyecto si no lo has visto antes.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal / VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Cuenta las filas con DuckDB y compara con `wc -l ventas.csv`. Si no coinciden, explica en una línea por qué.
2. Escribe `exploracion.sql` con tres consultas y, arriba de cada una, la pregunta del negocio que responde: ingresos por categoría, ticket promedio e ingresos por mes.
3. Rompe a propósito: suma `precio` declarándolo `DOUBLE` sin normalizar. Pega el error completo, arréglalo y vuelve a correr.

### Reto — opcional, para quien quiera más

1. Convierte el CSV a Parquet, compara tamaños con `ls -lh` y corre `EXPLAIN ANALYZE` de la misma consulta sobre cada formato.
2. Mide con `time` las tres consultas sobre CSV y sobre Parquet, y explica la diferencia **sin hablar de segundos**: habla de columnas y bytes leídos.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `con.sql(...)` devuelve las filas sin levantar ningún servidor
- [ ] `exploracion.sql` tiene las 3 consultas con su pregunta arriba
- [ ] `ventas.parquet` existe y las 3 cifras coinciden **exactamente** con las del CSV
- [ ] Puedes explicar en una frase cuándo NO usarías DuckDB
- [ ] `duckdb==1.5.6` (o tu 1.x) quedó fijada en `requirements.txt`

## 🔗 Para profundizar

- [DuckDB — leer CSV](https://duckdb.org/docs/1.5/data/csv/overview) — cómo se declaran los tipos al leer
- [DuckDB — Parquet](https://duckdb.org/docs/1.5/data/parquet/overview) — metadatos y lectura selectiva
- [DuckDB — COPY TO](https://duckdb.org/docs/1.5/sql/statements/copy) — las opciones exactas de escritura
- Anexos del vault: [[Polars en 25 minutos]]

## 🧭 Problemas frecuentes

- **`ModuleNotFoundError: No module named 'duckdb'`** → estás en otro Python. Mira `python -c "import sys; print(sys.executable)"` y activa el `venv`.
- **Dice que no encuentra `ventas.csv`** → la ruta es relativa a donde corres el script, no a donde está el archivo.
- **La consulta "no hace nada"** → es lazy: falta `.show()`, `.df()` o `.fetchall()`.
- **El total no coincide con el de Pandas** → revisa duplicados y el separador decimal. Casi nunca es el motor: es el dato.

Si nada de esto lo resuelve, anota el error completo en "Mi resultado": se retoma por ahí.

## ➡️ Siguiente tema

[[01 · Neon, Postgres en la nube]]: el mismo SQL, pero sobre una base que vive en la nube y a la que pueden conectarse un pipeline y una app.

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (las tres cifras, el tamaño del Parquet, el texto del error que no pudiste resolver). El siguiente tema retoma lo que quede acá.
