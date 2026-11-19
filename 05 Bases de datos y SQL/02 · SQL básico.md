---
tipo: tema
modulo: "05 Bases de datos y SQL"
capa: dominio inicial
herramienta: SQL
resultado: "3 consultas con la pregunta del negocio comentada arriba"
tiempo-lectura: 15 min
prerrequisitos: ["PostgreSQL, DDL y carga"]
tags: [tema, sql, postgresql, group-by, agregados]
---

# SQL básico: filtrar, agrupar y desconfiar del total

![[postgresql.svg|64]]

> [!tip] En una frase, antes de empezar
> **SQL es el idioma con el que le haces preguntas a una base de datos: escribes qué quieres y el motor decide cómo conseguirlo.** Una consulta es una frase con `SELECT` (qué columnas), `FROM` (de qué tabla) y `WHERE` (qué filas).

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea.
>
> **Lo que NO está acá:** la solución del ejercicio. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

Las 50.000 filas ya están en la base, y aparece la pregunta del negocio: **¿cuánto se vendió por categoría?** Tienes tres maneras de responderla y dos te van a fallar.

- Pedirle las 50.000 filas a Python y sumar con un `for`. Funciona, pero cada pregunta nueva es un script nuevo, y cada script se puede equivocar distinto.
- Exportar a Excel y hacer una tabla dinámica. Funciona con 500 filas; con 50.000 la máquina sufre, y el mes que viene lo rehaces a mano.
- Contarla a ojo. Nadie lo hace, pero es donde se termina cuando las otras dos tardan.

**Eso es lo que resuelve SQL.** La pregunta se escribe **una vez, en una frase**, y la responde el motor, que está hecho para eso: `GROUP BY categoria` te da una fila por categoría con su suma. No hay `for`, no hay tabla que rehacer; cambias una palabra y preguntas otra cosa.

> [!note] Qué no es
> - **`WHERE` no elige columnas.** Filtra filas enteras: la columna que no quieres se saca del `SELECT`, y ningún filtro la puede esconder.
> - **No es un lenguaje de propósito general.** No escribe archivos ni arma interfaces: pregunta a una base y devuelve filas. El cálculo que no cabe en una consulta se hace en Python.
> - **`GROUP BY` no ordena.** Agrupa; el orden lo decide `ORDER BY`, y sin él el motor no promete ninguno. Que hoy salga en orden no es una garantía.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Escribir 3 consultas que respondan 3 preguntas del negocio, cada una con su pregunta comentada arriba |
| **Después** | Demostrar cuánto se infla el ingreso cuando no deduplicas, y explicar por qué dos consultas a la misma pregunta dan cifras distintas |

Las 50.000 filas ya están en la base. El problema ahora no es cargar: es que **hay varias respuestas a la misma pregunta**, y tienes que defender la tuya.

## 🧠 Conceptos

1. **`WHERE` filtra filas, no columnas** — decide qué filas entran al resultado; las columnas se eligen en el `SELECT`.
2. **`GROUP BY` elige qué cuenta como una fila** — después de agrupar, cada fila es un grupo y solo puedes mostrar la clave o un agregado.
3. **El nulo no suma y el duplicado infla** — dos formas silenciosas de que tu cifra esté mal sin que nada falle.

## 📖 Teoría

### 1. `WHERE` filtra filas, no columnas
```sql
SELECT producto, cantidad * precio_unit AS ingreso
FROM ventas
WHERE metodo_pago = 'efectivo';
```
El `SELECT` elige **qué columnas** ves. El `WHERE` elige **qué filas** aparecen. Si quieres que una columna no aparezca, simplemente no la listas: no hay forma de "esconder" una columna con un filtro, porque un filtro actúa sobre filas enteras.

**Por qué importa:** la mitad de las consultas mal escritas vienen de acá. "Quiero solo las ventas de bebidas" no es sacar una columna: es descartar las filas que no lo son.

### 2. `GROUP BY` con agregados
```sql
SELECT categoria,
       count(*) AS ventas,
       sum(cantidad * precio_unit) AS ingreso
FROM ventas
GROUP BY categoria
ORDER BY ingreso DESC;
```
Antes del `GROUP BY`, cada fila es una venta. Después, cada fila es **una categoría**. Por eso `categoria` puede mostrarse tal cual, pero `cantidad` no: hay muchas y el motor no sabe cuál quieres. Lo que sí puede hacer es resumirlas con un agregado: `count` (cuántas), `sum` (cuánto suman), `avg` (promedio), `min`, `max`.

**Por qué importa:** es la consulta que reemplaza la tabla dinámica de Excel. Y entender por qué el motor exige esa regla es entender qué es un grupo.

### 3. El nulo no suma y el duplicado infla
Dos consultas a la misma pregunta pueden dar números distintos, y las dos corren sin error. Las dos causas más comunes:

```sql
SELECT count(*) AS filas,               -- cuenta filas
       count(cliente_id) AS con_dato,   -- cuenta solo las que NO son nulo
       count(DISTINCT id_venta) AS unicos
FROM ventas;
```

- **El nulo no suma:** `sum`, `avg` y `count(columna)` **ignoran** los nulos. Un `avg(precio)` divide por la cantidad de precios, no por la de filas.
- **El duplicado infla:** si un lote se cargó dos veces, `count(*)` sube y `sum(...)` suma las ventas repetidas. El ingreso crece **en la misma proporción en que crecen las filas**: un 16 % más de filas es un 16 % más de ingreso inventado.

## ⚠️ El error más común

> [!warning] `ERROR: column "v.categoria" must appear in the GROUP BY clause or be used in an aggregate function`
> ```
> ERROR:  column "v.categoria" must appear in the GROUP BY clause or be used in an aggregate function
> LINE 1: SELECT v.categoria, count(*) FROM ventas_raw v GROUP BY v.me...
>                ^
> ```
> **Qué significa:** pediste mostrar `categoria` agrupando por `metodo_pago`. Dentro del grupo "efectivo" hay cinco categorías y el motor no sabe cuál mostrar. **Cómo se lee:** el `^` marca la columna que sobra. Todo lo que está en el `SELECT` sin un agregado alrededor tiene que estar también en el `GROUP BY`. **Cómo se arregla:** dos caminos, y elegir es la parte importante. O agrupas por `categoria` (cambia la pregunta: ahora es "ventas por categoría"), o la envuelves en un agregado (`min(categoria)`). Lo que no se hace es meterla en el `GROUP BY` para que el error desaparezca.

Este no es un error de adorno: es el que más caro sale si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min
Sobre `ventas_raw` (o `ventas`, si ya la tipaste). Al terminar, pega las 3 consultas en "Mi resultado".

1. **(10 min)** Escribe 3 consultas para 3 preguntas del negocio: cuánto se vendió por categoría, cuántas ventas hubo por método de pago, y cuál fue el ticket promedio. Arriba de cada una, la pregunta como comentario `--`.
2. **(10 min)** Compara `count(*)` contra `count(DISTINCT id_venta)`. Si no coinciden, averigua qué filas sobran.
3. **(10 min)** Rompe a propósito: saca `categoria` del `GROUP BY` y corre. Lee el `^`. Después calcula el ingreso sin deduplicar y mide el porcentaje de inflación.

### Reto — opcional, para quien quiera más
1. Calcula el promedio de `precio_unit` de dos formas: `avg(precio_unit)` y `avg(coalesce(precio_unit, 0))`. Explica la diferencia con una frase.
2. Quédate solo con las categorías que superen las 8.000 ventas. ¿`WHERE` o `HAVING`? Prueba las dos y cuenta qué pasó.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Tus 3 consultas corren y cada una tiene su pregunta comentada arriba
- [ ] Puedes explicar por qué `count(*)` y `count(cliente_id)` pueden dar distinto
- [ ] Mediste el porcentaje de inflación del ingreso al no deduplicar
- [ ] Explicas en una frase por qué usaste `GROUP BY` y no un `for` de Python

## 🔗 Para profundizar

- [PostgreSQL — SELECT](https://www.postgresql.org/docs/current/sql-select.html) — el orden en que se evalúan `WHERE`, `GROUP BY` y `HAVING`
- [PostgreSQL — funciones de agregado](https://www.postgresql.org/docs/current/functions-aggregate.html) — qué hace cada una con los nulos
- [PostgreSQL — expresiones de tabla](https://www.postgresql.org/docs/current/queries-table-expressions.html) — `GROUP BY` y `HAVING` en detalle
- Anexo del vault: [[01 · PostgreSQL, DDL y carga]]

## 🧭 Problemas frecuentes

- **`must appear in the GROUP BY clause`** → mira el `^`: esa columna no es clave de grupo ni está agregada. Decide cuál quieres que sea.
- **El total no cuadra con el CSV** → o cargaste dos veces (compara con `count(DISTINCT id_venta)`), o hay filas partidas. Primero el conteo, después la suma.
- **`sum` devuelve vacío (NULL)** → todas las filas del grupo tenían nulo en esa columna. Es distinto de cero, y presentarlo como cero es un error de negocio.
- **`column "categoria" does not exist`** → o está mal escrita, o los datos crudos están todos en `text` y todavía no tipaste la tabla.

Si nada de esto lo resuelve, pega la consulta completa y su salida en "Mi resultado": ese mensaje es el punto de partida.

## ➡️ Siguiente tema

Vas a cruzar estas ventas con el clima, y ahí aparece el error más caro del curso: uno que no muestra ningún mensaje y te triplica la cifra.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá tus tres consultas y el resultado de cada una (o una captura de la salida). Si una cifra no te cuadra con el CSV, pega también las dos versiones: es la duda que más se repite y conviene resolverla antes de seguir.