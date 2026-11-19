# Glosario

Un término por línea, en el idioma en que te lo vas a encontrar en la documentación. Si aparece uno que no está acá, agrégalo: este archivo es de todos.

## 🐍 Python y herramientas

- **assert** — afirmación que tu propio código verifica; si falla, es un error con mensaje.
- **traceback** — el listado de errores de Python. Se lee de abajo hacia arriba.
- **módulo** — un archivo `.py` que se puede importar.
- **virtualenv / venv** — entorno de librerías aislado por proyecto.
- **pytest** — la herramienta que corre tus tests y dice cuáles pasan.

## 📊 Datos

- **CSV** — texto con filas y columnas separadas por comas. Universal y lento.
- **Parquet** — formato columnar, comprimido y con tipos. Pesa menos y se lee más rápido que el CSV.
- **columnar** — guarda los datos por columna (no por fila). Eso es lo que hace que agregar una columna entera sea mucho más rápido.
- **lazy (perezoso)** — el motor no calcula hasta que se lo pides de verdad; así puede optimizar antes.
- **perfilado** — describir un dataset antes de tocarlo: nulos, tipos, rangos, duplicados.
- **granularidad** — qué representa una fila. Si dices "una fila por venta", un join que devuelve tres filas por venta cambió la granularidad y tu suma miente.
- **fan-out** — el efecto de un join que multiplica filas sin devolver un error.
- **idempotente** — correrlo dos veces deja el mismo resultado que correrlo una.

## 🗄️ Bases de datos

- **DDL** — las instrucciones que definen la estructura (`CREATE TABLE`, `ALTER`).
- **staging** — la tabla de carga cruda, antes de tipar y normalizar.
- **índice** — estructura que acelera búsquedas por una columna.
- **vista** — una consulta guardada con nombre, que se usa como si fuera una tabla.
- **pooler** — intermediario que reutiliza conexiones a la base en vez de abrir una por usuario.
- **arranque en frío (cold start)** — el retraso de la primera consulta cuando la base estuvo inactiva.

## 🌐 Web y APIs

- **endpoint** — la URL concreta a la que le pides datos.
- **status code** — el número que dice qué pasó: 200 está bien, 404 no existe, 400 pediste mal, 500 falló el otro.
- **JSON** — formato de texto para datos anidados; es lo que devuelven casi todas las APIs.
- **API key** — credencial que te identifica ante un servicio. Nunca va en el código.
- **robots.txt** — el archivo donde un sitio dice qué se puede rastrear y qué no.
- **scraping** — extraer datos de HTML. Requiere criterio legal y técnico, no solo una librería.

## ⚙️ Publicar y automatizar

- **deploy** — publicar tu app para que otros la abran.
- **secreto** — credencial guardada en un lugar que no es el repositorio.
- **cron** — programación de una tarea repetida. **Siempre en UTC**: Caracas es UTC-4.
- **workflow** — el archivo que define qué corre, cuándo y en qué orden en GitHub Actions.
- **runner** — la máquina (de GitHub o tuya) donde corre el workflow.

## 🤖 IA y agentes

- **agente** — un modelo que razona, usa herramientas y observa el resultado en un ciclo. No es autocompletado: puede actuar sobre tus archivos.
- **skill** — instrucción reusable con formato y criterio de aceptación, para que el agente la aplique siempre igual.
- **subagente** — un agente al que delegas una tarea con contexto acotado. Devuelve un resultado; hay que verificarlo igual.
- **alucinación** — cuando el modelo afirma algo falso con seguridad. Se detecta ejecutando, no leyendo.
- **verificación por ejecución** — la única prueba real: correr el test o la consulta y mirar el resultado.

## 📈 Estadística (lo mínimo)

- **media** — el promedio. Se lo llevan los valores extremos.
- **mediana** — el valor del medio. Resiste los extremos.
- **dispersión** — cuánto se alejan los datos del centro. Dos series con la misma media pueden ser opuestas.
- **correlación** — dos cosas se mueven juntas. **No** implica que una cause la otra.
- **inferencia** — sacar conclusiones sobre una población desde una muestra.
