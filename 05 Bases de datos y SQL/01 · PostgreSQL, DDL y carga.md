---
tipo: tema
modulo: "05 Bases de datos y SQL"
capa: dominio inicial
herramienta: PostgreSQL
resultado: "tabla ventas_raw con las 50.000 filas cargadas y el conteo validado"
tiempo-lectura: 15 min
prerrequisitos: ["Listas y diccionarios", "psql o el editor SQL de Neon"]
tags: [tema, postgresql, sql, ddl, copy, staging]
---

# PostgreSQL: la tabla, los tipos y la primera carga

![[postgresql.svg|64]]

> [!tip] En una frase, antes de empezar
> **Una base de datos es un lugar donde los datos viven ordenados en tablas, y PostgreSQL es el programa que las guarda y responde las preguntas que le haces.** En vez de un archivo que abres, tienes un servicio encendido al que le pides filas.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea.
>
> **Lo que NO está acá:** la solución del ejercicio. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

El CSV de la tienda ya tiene 50.000 filas. Hasta acá lo abriste con Python o con Excel y salió del paso. Lo que empieza a doler es lo de después:

- Abres el archivo para buscar una venta y tienes que esperar. Filtras una columna y la máquina se queda pensando.
- Dos personas abren el mismo archivo y una guarda encima de la otra. **Se perdió el cambio de alguien y no hay forma de saber de quién.**
- En la columna `fecha` hay `15/03/2024`, `2024-03-15` y una celda que dice `s/f`. Nada avisó al entrar; avisa tres meses después, cuando el informe no cuadra.

**Eso es lo que resuelve una base de datos.** Los datos dejan de ser un archivo que se abre y pasan a ser una tabla con **columnas y tipos elegidos a propósito**: una columna `fecha date` rechaza `s/f` en el momento en que intenta entrar, y el motor cuenta 50.000 filas en un segundo. PostgreSQL es el programa que hace eso, y le hablas con SQL.

> [!note] Qué no es
> - **No es Excel con SQL.** El CSV es una foto: no rechaza una fecha imposible ni deja dos personas trabajando a la vez. La base cobra por adelantado, cuando eliges el tipo de cada columna.
> - **No es SQL.** SQL es el idioma con el que se pregunta; PostgreSQL es uno de los programas que lo hablan, y lo que aprendas del idioma sirve en los demás.
> - **Cargar no es validar.** Que `\copy` termine sin error dice que las filas entraron, no que estén bien: el conteo contra el archivo y la conversión de tipos se comprueban aparte.
> - **La tabla staging no es la final.** `ventas_raw`, todo `text`, existe para que entre todo y poder contarlo; es en `ventas` donde los tipos empiezan a rechazar.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Tener una base PostgreSQL arriba y la tabla `ventas_raw` creada, con las columnas y los tipos elegidos a propósito |
| **Después** | Cargar el CSV completo con `\copy` y demostrar que `count(*)` coincide con las líneas del archivo |

## 🧠 Conceptos

1. **El tipo es una decisión, no un detalle** — define qué entra, qué se rechaza y qué se puede calcular después.
2. **`COPY` no perdona** — si una fila no entra en el tipo, aborta la carga entera y no cargó nada.
3. **Crudo primero, tipos después** — carga todo como texto, cuenta las filas y recién ahí convierte.

## 📖 Teoría

### 1. La tabla y sus tipos se deciden antes de cargar
```sql
CREATE TABLE ventas_raw (
  id_venta text, fecha text, categoria text, producto text,
  cantidad text, precio_unit text, metodo_pago text, cliente_id text
);
```
Todo `text` acepta cualquier cosa, y eso es lo que quieres en el primer paso: que entre. Pero esa tabla no te deja sumar ni ordenar por fecha. La tipada es otra decisión:
```sql
CREATE TABLE ventas (
  id_venta integer PRIMARY KEY, fecha date, precio_unit numeric(10,2)
);
```
`numeric(10,2)` para dinero, nunca `float`: `float` redondea, y en dinero eso es un centavo que aparece y desaparece. `date` para la fecha, no `text`: solo `date` rechaza un 31 de febrero y ordena cronológicamente.

**Por qué importa:** el tipo es la validación más barata que existe. Una columna `date` rechaza `15/03/2024` al entrar; una `text` la acepta y te deja el problema tres meses después.

### 2. `COPY`: 50.000 filas en un segundo
```sql
\copy ventas_raw FROM 'ventas.csv' WITH (FORMAT csv, HEADER true)
```
El `\copy` es un comando de `psql`, no de SQL: lee el archivo **en tu computadora** y lo manda por la conexión. `COPY` (sin barra) le pide al **servidor** que abra una ruta de *su* disco: contra Neon no ve tus archivos y falla. `\copy` sí funciona. Esa diferencia es la que decide si puedes trabajar contra una base en la nube.

### 3. Carga crudo, cuenta, y después tipa
```sql
SELECT count(*) FROM ventas_raw;   -- tiene que dar 50000
```
Y del otro lado, en la terminal, `wc -l ventas.csv` da **50001**, porque cuenta el encabezado. Si esos números no cierran, no sigas: se perdió algo.

Recién cuando el conteo cierra, se tipa. La conversión va en una consulta, que puedes volver a correr las veces que haga falta:
```sql
SELECT to_date(fecha, 'DD/MM/YYYY') AS fecha,
       replace(precio_unit, ',', '.')::numeric(10,2) AS precio
FROM ventas_raw LIMIT 5;
```

## ⚠️ El error más común

> [!warning] `ERROR: date/time field value out of range: "15/03/2024"`
> ```
> ERROR:  date/time field value out of range: "15/03/2024"
> HINT:  Perhaps you need a different "datestyle" setting.
> CONTEXT:  COPY ventas, line 2, column fecha: "15/03/2024"
> ```
> **Qué significa:** metiste `15/03/2024` en una columna `date` y el motor leyó `15` como el mes. La fecha no está mal: el formato `día/mes/año` no es el que Postgres espera. **Cómo se lee:** el `CONTEXT` es la línea más útil del mensaje: dice **línea 2, columna fecha** del archivo. Empieza por ahí, no adivines. Si el valor fuera basura absoluta, como `s/f` ("sin fecha"), el mensaje sería `invalid input syntax for type date: "s/f"`. **Cómo se arregla:** no toques el `datestyle` "para que pase": es configuración del servidor, arregla tu sesión y rompe la de otro. Carga la columna como `text` y convierte con `to_date(fecha, 'DD/MM/YYYY')`, que dice en voz alta cuál es el formato.

Este no es un error de adorno: es el que más caro sale si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min
Los haces en tu terminal o en el editor SQL de Neon. Al terminar, pega el conteo en "Mi resultado".

1. **(10 min)** Deja una base arriba y crea `ventas_raw` con las ocho columnas, todas `text`.
2. **(10 min)** Carga el CSV con `\copy` y compara `SELECT count(*)` contra `wc -l ventas.csv`. Los números tienen que cerrar.
3. **(10 min)** Rompe a propósito: crea una tabla `ventas` con `fecha date` y cópiale el mismo CSV. Lee el `CONTEXT` y anota línea y columna.

### Reto — opcional, para quien quiera más
1. Carga el mismo CSV dos veces y compara `count(*)` con `count(DISTINCT id_venta)`. ¿Cuántas filas sobran?
2. Cuenta cuántos `precio_unit` vienen con coma decimal y cuántos con punto. Ese número decide qué `replace` necesitas.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `\dt` lista `ventas_raw` y su `count(*)` da 50000, igual a `wc -l ventas.csv` menos el encabezado
- [ ] Provocaste el error de fecha y sabes en qué línea y columna del CSV estaba
- [ ] Explicas en una frase por qué cargaste todo como `text` antes de tipar

## 🔗 Para profundizar

- [PostgreSQL — COPY](https://www.postgresql.org/docs/current/sql-copy.html) — opciones `FORMAT csv`, `HEADER` y `DELIMITER`; y [tipos numéricos](https://www.postgresql.org/docs/current/datatype-numeric.html), por qué `numeric` y no `float`
- [PostgreSQL — tipos de fecha y hora](https://www.postgresql.org/docs/current/datatype-datetime.html) — formatos de entrada y `DateStyle`
- Anexo del vault: [[Chuleta de errores]]

## 🧭 Problemas frecuentes

- **Docker no es requisito.** Hay dos caminos más cortos: PostgreSQL instalado nativo, o una base gratis en la nube en [console.neon.tech](https://console.neon.tech).
- **`ERROR: missing data for column "categoria"`** → el CSV usa `;` como separador (Excel en español lo hace por defecto). Agrega `DELIMITER ';'` al `\copy`.
- **`ERROR: could not open file "ventas.csv" ... HINT: ... psql's \copy`** → usaste `COPY` contra una base en la nube: el servidor no ve tu disco.
- **Problemas de DNS o de red al conectarte** → no los resuelvas a mano aquí: se ven en detalle en [[01 · Neon, Postgres en la nube]].

Si nada de esto lo resuelve, pega el mensaje completo en "Mi resultado". Ese mensaje es el punto de partida.

## ➡️ Siguiente tema

Con las 50.000 filas adentro, vas a poder hacerle preguntas al negocio con SQL: filtrar, agrupar y sumar.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá el resultado de `count(*)` junto con la cantidad de líneas de tu CSV, y el error completo si no coinciden. Dos números y un mensaje: con eso sabemos si se cargó todo o si se perdió la mitad en el camino.