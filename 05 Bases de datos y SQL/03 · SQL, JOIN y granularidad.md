---
tipo: tema
modulo: "05 Bases de datos y SQL"
capa: dominio inicial
herramienta: SQL
resultado: "vista v_ventas_clima con una fila por fecha y el conteo verificado"
tiempo-lectura: 15 min
prerrequisitos: ["SQL básico", "PostgreSQL, DDL y carga"]
tags: [tema, sql, postgresql, join, granularidad, vistas]
---

# JOIN y granularidad: cuando 50.000 se vuelven 150.000

![[postgresql.svg|64]]

> [!tip] En una frase, antes de empezar
> **Un JOIN pega dos tablas por una columna que comparten y te devuelve una sola tabla con las columnas de las dos.** `ventas` tiene fechas e ingresos, `clima` tiene fechas y lluvia, y el JOIN las pone en la misma fila cuando la fecha coincide.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea.
>
> **Lo que NO está acá:** la solución del ejercicio. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

Tienes las ventas en una tabla y el clima en otra. Las dos cargaron bien. Y ahora la pregunta que importa: **¿los días de lluvia se vende menos?** Para responderla necesitas las dos cosas en la misma fila, y no las tienes:

- En un CSV tendrías que pegar la columna de lluvia al lado de cada venta, a mano, 50.000 veces. O usar el `BUSCARV` de Excel y confiar en que las fechas están escritas igual en las dos.
- Puedes hacer dos consultas y comparar los totales en la cabeza. Eso no es un análisis: es una impresión.
- Y si el clima cubre menos días que las ventas, una de las dos tablas pierde filas en el camino **sin avisarte**.

**Eso es lo que resuelve un JOIN.** Le dices al motor con qué columna se corresponden las dos tablas (`ON v.fecha = c.fecha`) y te devuelve una fila por venta **con** la lluvia de ese día. La relación se escribe una vez y el motor la aplica a las 50.000, sin copiar y pegar.

> [!note] Qué no es
> - **No es un `BUSCARV`.** `BUSCARV` devuelve el primer resultado y sigue como si nada; el JOIN usa todas las coincidencias, y con la clave repetida devuelve una fila por cada una. Así un informe pasa de 50.000 filas a 150.000.
> - **No une por parecido.** El `ON` compara valores iguales: si una tabla guarda la fecha como `15/03/2024` y la otra como `2024-03-15`, texto contra texto no coincide, y con `LEFT` el clima sale `NULL` sin ningún mensaje.
> - **Una vista no acelera ni congela nada.** Guarda la consulta, no los datos: se vuelve a ejecutar en cada lectura, con los cambios del origen incluidos.
> - **No todo se resuelve con JOIN.** Si la pregunta es sobre una sola tabla, esa consulta va sola; el JOIN es para cuando necesitas las dos a la vez.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Cruzar ventas y clima verificando el conteo **antes y después** del JOIN, y detectar si las filas se multiplicaron |
| **Después** | Dejar `v_ventas_clima` creada y consultable, con una fila por fecha y un `count(*)` que se puede defender |

Ventas y clima están en la misma base, en dos tablas. Al juntarlas aparece el error más caro del curso: uno que **no muestra mensaje**, porque la consulta no falla; la cifra está mal.

## 🧠 Conceptos

1. **El JOIN encuentra dos tablas por un valor compartido** — y lo que no encuentra depende de `INNER` o `LEFT`.
2. **Granularidad: cuántas filas tiene la tabla por clave** — si el lado que crees "uno" tiene la clave repetida, el join multiplica sin avisar.
3. **Una vista es un contrato con nombre** — la lógica se define una vez y se consulta sin repetirla.

## 📖 Teoría

### 1. JOIN por clave, y qué pasa con las filas sin match
```sql
SELECT count(*) FROM ventas v LEFT JOIN clima c ON v.fecha = c.fecha;   -- 150000
SELECT count(*) FROM ventas v JOIN clima c      ON v.fecha = c.fecha;   -- menos o igual
```
`JOIN` se queda **solo con las filas que tienen pareja** en las dos tablas. `LEFT JOIN` conserva **todas** las de la izquierda y rellena con `NULL` la derecha sin pareja.

El clima cubre menos días que las ventas: con `JOIN` a secas, las ventas de los días sin clima **desaparecen sin decir nada**. Un informe armado así pierde días enteros.

**Por qué importa:** elegir `INNER` o `LEFT` no es sintaxis: es decidir qué hacer con lo que falta, y poder justificarlo.

### 2. Granularidad: el fan-out que multiplica filas
La regla: **el resultado tiene una fila por cada combinación que matchea**. Si del lado del clima hay 3 filas para la misma fecha, cada venta de esa fecha aparece 3 veces.

```sql
SELECT fecha, count(*) AS filas FROM clima
GROUP BY fecha HAVING count(*) > 1
ORDER BY filas DESC LIMIT 5;
```
Ese es el diagnóstico: dice si la clave es única del lado que estás pegando. Acá devuelve 3 para casi todas las fechas, porque `clima` se cargó tres veces; el join hace 50.000 × 3. La salida no es `DISTINCT` sobre el resultado —eso esconde el problema—: es **colapsar el clima a una fila por fecha antes de unir** y guardarlo como `clima_diario`, que usa la vista del concepto 3.

### 3. La vista como contrato reutilizable
```sql
CREATE VIEW v_ventas_por_dia AS
SELECT fecha, count(*) AS ventas,
       sum(cantidad * precio_unit) AS ingreso
FROM ventas
GROUP BY fecha;
```
Una vista no guarda datos: guarda **la consulta** y la vuelve a ejecutar cada vez. Fija el contrato en un solo lugar: si cambia cómo se calcula el ingreso, lo cambias ahí y listo.

**Por qué importa:** sin vista, cada uno reescribe el join y le agrega un `LEFT` distinto, y "una fila por fecha" deja de ser una regla. La que vas a construir tú lleva además el clima: misma idea.

## ⚠️ El error más común

> [!warning] Sin mensaje: el conteo pasa de 50.000 a 150.000
> ```
> SELECT count(*) FROM ventas v LEFT JOIN clima c ON v.fecha = c.fecha;
>  count
> --------
> 150000
> ```
> **Qué significa:** cada venta se contó tres veces porque el clima tiene tres filas para la misma fecha. No hay error: hay un número equivocado, el peor tipo, porque pasa el informe y falla en la reunión. **Cómo se lee:** no leas el total final: lee el total **de los dos lados antes de unir** y el de cada fecha del lado derecho. Si hay más de una fila por clave, el join infla. **Cómo se arregla:** reduce el clima a una fila por fecha (`GROUP BY fecha` o `DISTINCT ON (fecha)`) y vuelve a contar. El ingreso también se triplica: no creció el negocio, se contó tres veces la misma venta.

Este no es un error de adorno: es el que más caro sale si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min
Sobre `ventas` y `clima`. Al terminar, pega los conteos en "Mi resultado".

1. **(10 min)** Cuenta las filas de `ventas`, las de `clima` y las del `LEFT JOIN`. Anota los tres números y explica la diferencia.
2. **(10 min)** Diagnostica la granularidad del clima: `GROUP BY fecha HAVING count(*) > 1`. Después arma una tabla (o subconsulta) con **una fila por fecha**.
3. **(10 min)** Rompe a propósito: cambia `LEFT` por `JOIN` y cuenta cuántas ventas desaparecen. Después vuelve al `LEFT` y crea la vista.

### Reto — opcional, para quien quiera más
1. Consulta `v_ventas_clima` y verifica que `count(*)` coincide con la cantidad de fechas distintas de `ventas`.
2. Compara el ingreso total de la vista contra el de la tabla sin duplicar. Si no coinciden, todavía hay fan-out.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Tienes los tres conteos anotados (ventas, clima, join) y `count(*)` de `v_ventas_clima` iguala las fechas distintas de `ventas`
- [ ] Explicas en una frase por qué usaste `LEFT JOIN` y no `JOIN`, y cuántas ventas se pierden con cada uno
- [ ] Explicas qué es granularidad con un ejemplo de tu propia tabla

## 🔗 Para profundizar

- [PostgreSQL — expresiones de tabla](https://www.postgresql.org/docs/current/queries-table-expressions.html) — "Joined Tables" y los tipos de join
- [PostgreSQL — CREATE VIEW](https://www.postgresql.org/docs/current/sql-createview.html) — qué guarda una vista y qué recalcula; y [SELECT](https://www.postgresql.org/docs/current/sql-select.html) para el `DISTINCT ON`
- Anexo del vault: [[02 · SQL básico]] · [[01 · HTTP con httpx]]

## 🧭 Problemas frecuentes

- **El resultado tiene el doble o el triple de filas** → la clave está repetida del lado derecho. Corre el `GROUP BY fecha HAVING count(*) > 1` antes de cualquier join.
- **El clima sale todo en `NULL`** → el `LEFT JOIN` no encontró pareja: revisa el rango de fechas y que los tipos sean los dos `date`, no `text` contra `date`.
- **`column reference "fecha" is ambiguous`** → las dos tablas tienen `fecha` y no dijiste de cuál. Pon alias (`v.fecha`, `c.fecha`) en el `SELECT`, en el `ON` y en el `GROUP BY`.
- **`CREATE VIEW` falla porque ya existe** → usa `CREATE OR REPLACE VIEW`, o `DROP VIEW` primero si le cambiaste las columnas.

Si nada de esto lo resuelve, pega los tres conteos en "Mi resultado". Ese es el punto de partida.

## ➡️ Siguiente tema

Con ventas y clima en una vista que no infla filas, vas a poder sacar conclusiones y graficarlas sin que un error de conteo te arruine la historia.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá el `count(*)` de ventas **antes y después** del JOIN, y el `count(*)` de la vista. Si los números no son iguales, pega también la consulta: ese es exactamente el error que este tema enseña a detectar.