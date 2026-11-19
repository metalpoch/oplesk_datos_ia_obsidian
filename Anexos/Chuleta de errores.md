---
tipo: anexo
modulo: "Anexos"
capa: familiaridad
herramienta: "transversal a todo el curso"
resultado: "ninguno: es la chuleta de consulta más usada del vault"
tiempo-lectura: 8 min
prerrequisitos: []
tags: [anexo, errores, chuleta, depuracion]
---

# Chuleta de errores

> [!info] Esto no se lee: se busca
> `Ctrl+F` y pega el mensaje —no la causa, el mensaje. Las comillas y los números cambian entre corridas: busca por la parte estable (`must appear in the GROUP BY`, `UnhashableParamError`, `chat not found`). La columna **Etapa** dice en qué momento del trabajo aparece el error: entorno, primer contacto con datos, limpieza, consulta y agregación, verificación, publicación e integración.

## ⏱️ Los primeros 90 segundos

1. Lee **la última línea** del traceback: dice qué pasó. La de arriba dice dónde.
2. Busca el **primer** error del log, no el último: los de abajo suelen ser consecuencia.
3. Antes de tocar el código, reproduce el error dos veces. Si no se repite, el problema es el entorno o la red.
4. Si el mensaje está en esta tabla, sigue el arreglo al pie de la letra antes de improvisar.

## 🐍 Python

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `TypeError: unsupported operand type(s) for +: 'int' and 'str'` | primer contacto con datos | Sumas un texto que parece número: `"3,50"` | Encuentra la fila antes de convertir: `isinstance` + reporte, no `float()` a ciegas |
| `AssertionError` (solo, sin mensaje) | verificación | `assert total == 50_000` sin decir qué falló | Pon el valor en el mensaje: `assert total == 50_000, f"filas: {total}"` |
| `ModuleNotFoundError: No module named 'httpx'` | entorno | Instalaste en otro entorno o falta en `requirements.txt` | Activa el venv y `python -m pip install httpx`; después agrégalo a `requirements.txt` |
| `FileNotFoundError: [Errno 2] No such file or directory: 'ventas.csv'` | entorno | Corres desde otra carpeta y la ruta es relativa al cwd | `Path(__file__).resolve().parent / "data" / "ventas.csv"` |
| `AttributeError: 'NoneType' object has no attribute 'text'` | primer contacto con datos | `select_one` no encontró el precio (producto agotado) y devolvió `None` | Manéjalo explícito: `if el is None: sin_precio += 1; continue` |
| `ValueError: could not convert string to float: '4,20'` | limpieza | Coma decimal: es texto hasta que la reemplaces | `float(x.replace(".", "").replace(",", "."))` y valida el resultado |
| `ZeroDivisionError: division by zero` | consulta y agregación | Ticket promedio de un día con 0 tickets | `total / tickets if tickets else 0` |

## 🌿 Git

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `! [rejected]        main -> main (fetch first)` | integración | El remoto tiene commits que no tienes (editaste en la web o clonaste viejo) | `git pull --rebase` y vuelve a pushear; nunca `--force` sobre main |
| `CONFLICT (content): Merge conflict in ventas.csv` | integración | Dos ramas tocaron las mismas líneas | Abre el archivo, resuelve los `<<<<<<<`, después `git add` y `git commit` |
| `error: Your local changes would be overwritten by merge` | integración | Tienes cambios sin commitear en el árbol | `git stash`, haz el pull, `git stash pop` |

## 🗄️ SQL y PostgreSQL

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `ERROR: date/time field value out of range: "31/02/2024"` | primer contacto con datos | Cargaste una fecha imposible (el 31 de febrero no existe) en una columna `DATE` | Carga todo a staging `TEXT`, normaliza, y recién después `to_date(a, 'DD/MM/YYYY')` |
| `ERROR: column "v.categoria" must appear in the GROUP BY clause or be used in an aggregate function` | consulta y agregación | Eliges una columna que no está agrupada ni agregada | Agrégala al `GROUP BY` o envuélvela en un agregado |
| `ERROR: relation "ventas_raw" does not exist` | entorno | Te conectaste a otra base o no corriste el DDL | `\dt` para ver qué tablas hay y revisa a qué base apunta la conexión |
| `ERROR: duplicate key value violates unique constraint` | publicación | Carga no idempotente: corriste el insert dos veces | `DELETE` por partición + `INSERT`, o `ON CONFLICT (id) DO NOTHING` |

## 🐼 pandas

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `DtypeWarning: Columns (3) have mixed types. Specify dtype option on import` | primer contacto con datos | La columna trae números y textos: comas, `N/A`, vacíos | `dtype=str` al leer, limpia y castea explícito |
| `ValueError: time data "31/02/2024" doesn't match format "%d/%m/%Y"` | limpieza | `to_datetime` sin `errors` y hay una fecha imposible | `errors="coerce"` y cuenta los `NaT` antes y después |
| `UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 12` | primer contacto con datos | El CSV está en latin-1 (la ñ y los acentos que guardó Excel) | `encoding="latin-1"` (o `cp1252`) y documenta el encoding en el README |
| `MergeError: You are trying to merge on object and int64 columns for key 'id'` | consulta y agregación | La clave tiene distinto tipo en las dos tablas | Castea las dos al mismo tipo y usa `validate="m:1"` |
| `KeyError: 'monto_total'` | integración | Renombraste una columna y el resto pide el nombre viejo | Un solo módulo con los nombres, y un `assert` de columnas antes de usarlas |

## 🦆 DuckDB

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `Conversion Error: Could not convert string '4,20' to DOUBLE` | limpieza | DuckDB infirió texto por las comas y signos, y el cast explota | Limpia antes de leer, o usa `TRY_CAST` y cuenta los que quedan nulos |
| `Binder Error: Referenced column "monto_total" not found in FROM clause!` | integración | El Parquet o la vista tienen otro nombre de columna | `SELECT * LIMIT 0` o `DESCRIBE` para ver los nombres reales; una sola fuente de nombres |
| `Catalog Error: Table with name ventas does not exist!` | primer contacto con datos | La ruta relativa no es la que crees, o el Parquet está en otra carpeta | Apunta a la ruta del archivo en la consulta y verifica la carpeta con `ls` |

## 📊 Streamlit

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `NameError: name 'st' is not defined` | publicación | Falta `import streamlit as st`, o está después de la línea que lo usa | Importa arriba de todo y revisa el orden del script |
| `UnhashableParamError: Cannot hash argument 'conn' (of type 'psycopg.Connection')` | publicación | Metiste la conexión adentro de una función con `@st.cache_data` | `@st.cache_resource` para la conexión, `@st.cache_data` para los datos |
| `ModuleNotFoundError: No module named 'psycopg'` (en el deploy, no en local) | publicación | Falta en `requirements.txt`, o el nombre del paquete no es el del import | Agrega `psycopg[binary]` a `requirements.txt`; el log del deploy dice qué falta |
| `KeyError: 'DATABASE_URL'` | publicación | Usaste `.env` en local y en la nube no existe | `st.secrets["DATABASE_URL"]` y cárgala en el panel de secrets |

## ☁️ Nube: Actions, Neon y Telegram

| Error | Etapa | Causa más probable | Arreglo |
|---|---|---|---|
| `Invalid workflow file: .github/workflows/pipeline.yml#L14` | publicación | Indentación YAML: un `steps:` mal alineado o una clave repetida | Copia el workflow base y cambia lo mínimo; la línea del mensaje es la pista |
| `could not translate host name "ep-xxx-pooler.us-east-2.aws.neon.tech" to address` | publicación | Host mal copiado, o el DNS de tu red no lo resuelve | Verifica el host en el panel de Neon; si el DNS está bloqueado, prueba desde otra red |
| `HTTPStatusError: Client error '400 Bad Request' for url 'https://api.open-meteo.com/v1/forecast?...'` | primer contacto con datos | Un parámetro mal escrito: la API no adivina | Lee el cuerpo de la respuesta (`reason`), no solo el código, e imprime la URL |
| `Bad Request: chat not found` (en el JSON del error, `error_code` 400) | publicación | El `chat_id` no es el del chat, o el bot no está en ese grupo | Saca el id con `getUpdates` después de escribirle al bot; en grupos el id es negativo |
| `HTTPStatusError: Client error '401 Unauthorized' for url 'https://api.telegram.org/bot<TOKEN>/sendMessage'` | publicación | Token revocado o mal copiado | Token en `.env` o secrets, sin espacios; si lo mostraste en pantalla, revócalo en BotFather |

## 🆘 Si el error no está acá

1. Pega el mensaje completo en el buscador **entre comillas**, sin los números ni las rutas.
2. Si aparece en un repo, mira la versión: un issue de 2022 puede hablar de otra API.
3. Redúcelo: 5 líneas que reproduzcan el error valen más que 200 que lo escondan.
4. Si vas a pedir ayuda, pega el comando, el error completo y **qué probaste**. Con esas tres cosas, cualquiera te desbloquea.

## 🔗 Ver también

- Los temas donde aparecen: [[01 · Git y GitHub]] · [[01 · HTTP con httpx]] · [[02 · Pandas, limpiar y reportar]] · [[01 · DuckDB y Parquet]] · [[01 · Streamlit, primera app]] · [[01 · Neon, Postgres en la nube]] · [[01 · GitHub Actions, workflow y cron]]
- [[Polars en 25 minutos]] — el error de orden de filas de la 2.0 no aparece como excepción: es silencioso
