---
tipo: tema
modulo: "10 Neon PostgreSQL"
capa: familiaridad
herramienta: Neon
resultado: "proyecto en Neon con la tabla `ventas` cargada y el conteo validado contra el CSV"
tiempo-lectura: 18 min
prerrequisitos: []
tags: [neon, postgres, automatizar]
---

# Neon: tu Postgres en la nube, sin servidor que administrar

![[neon.svg|64]] ![[postgresql.svg|64]]

> [!tip] En una frase, antes de empezar
> **Neon es PostgreSQL en la nube: una base de datos donde guardas tablas y las consultas con SQL, sin instalar ni mantener el servidor —de eso se encarga el servicio.**

> [!info] Lee esto antes de empezar — 18 minutos
> Léelo completo una vez y marca `#duda` donde no te cierre. **Lo que NO está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

La app ya es pública, pero lee un archivo. Mientras tanto, el pipeline corre en GitHub, en una máquina que no tiene tu `ventas.parquet`, y cuando alguien carga una venta nueva no hay dónde escribirla: un archivo no es un lugar donde varios leen y escriben a la vez.

La otra salida es instalar PostgreSQL en tu laptop. Funciona hasta que cierras la tapa: ahí el dashboard deja de tener de dónde leer, y el compañero que quería probarlo no puede.

**Eso es lo que resuelve Neon:** el dato vive en un servidor al que todos llegan, aunque tu laptop esté apagada, y se consulta con el mismo SQL de siempre. Tú administras tablas, no servidores.

> [!note] Qué no es
> DuckDB analiza archivos, y lo hace muy bien, pero es de una sola persona y no es el sistema de registro. Neon es donde el dato vive y varios leen y escriben. Si tu dashboard tiene que leer con tu laptop apagada, esto es lo que necesitas; si no, un Parquet alcanza.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Crear el proyecto desde la consola y dejar la tabla `ventas` con tus filas cargadas, vista desde el editor SQL |
| **Después** | Validar ese conteo contra el CSV y medir en segundos cuánto tarda la primera consulta tras la inactividad |

## 🧠 Conceptos

1. **Postgres gestionado: el motor es el mismo, el trabajo no** — Neon te ahorra instalar, actualizar y respaldar. Te ata a su red, su contraseña y su política de suspensión. El SQL es el mismo que ya escribiste.
2. **La cadena de conexión: seis datos en una línea** — host, puerto, base, usuario, contraseña y `sslmode`. Cada uno falla distinto, y eso te sirve para diagnosticar sin adivinar.
3. **Arranque en frío: la primera consulta paga la cuenta** — en el plan gratuito el cómputo se suspende por inactividad. La primera consulta lo despierta y tarda; las que siguen van rápido. **No se borra ningún dato.**

## 📖 Teoría

### 1. Postgres gestionado vs. local

Hasta ahora el motor corría en tu máquina. Neon es el mismo PostgreSQL con alguien más de guardia: te ahorra instalar, actualizar, respaldar y estar de guardia; te ata a su red (y su DNS), su contraseña, su política de suspensión y su factura.

**Por qué importa:** el criterio no es "la nube es mejor", es que el dashboard lea el dato con tu laptop apagada. Si eso no lo necesitas, DuckDB sobre Parquet te sobra ([[01 · DuckDB y Parquet]]).

### 2. La cadena de conexión y SSL

Neon te da una línea larga: la *connection string*. Ahí están los seis datos, y cada uno tiene su propio error.

```text
postgresql://usuario:password@ep-cool-name-123456.us-east-2.aws.neon.tech/neondb?sslmode=require
             └ usuario   └ password   └── host + región ──┘  └ base┘   └─ SSL ─┘
```

- **`sslmode=require` no se saca:** Neon rechaza conexiones sin cifrar.
- **Pooler:** la consola ofrece otra cadena con `-pooler` en el host. Misma base, un intermediario de conexiones.

```python
import os, psycopg
with psycopg.connect(os.environ["DATABASE_URL"]) as conn:
    filas = conn.execute("select count(*) from ventas").fetchone()[0]
print(filas)
```

La cadena va en `.env`, **nunca** en el `.py` ni en el repo.

### 3. Arranque en frío

Cuando nadie consulta por un rato, el cómputo se suspende. La primera consulta lo despierta y ahí pagas la espera; las siguientes son instantáneas.

```python
import time
for intento in range(2):
    t0 = time.perf_counter()
    # ... tu consulta acá ...
    print(f"consulta {intento + 1}: {time.perf_counter() - t0:.2f} s")
```

Vas a ver algo como `3.41 s` y después `0.06 s`. Si no lo sabes, vas a creer que tu consulta está mal. Y si "solucionas" el problema con un cron cada 5 minutos para mantenerlo despierto, te comes la cuota mensual de cómputo. Los números del plan cambian: revísalos en su documentación antes de decidir.

## ⚠️ El error más común

> [!warning] `psycopg.OperationalError: could not translate host name "..." to address: Name or service not known`
> ```text
> Traceback (most recent call last):
>   File "cargar.py", line 6, in <module>
>     conn = psycopg.connect(os.environ["DATABASE_URL"])
> psycopg.OperationalError: could not translate host name "ep-cool-name-123456.us-east-2.aws.neon.tech"
> to address: Name or service not known
> ```
> **Qué significa:** tu equipo no pudo traducir ese nombre a una IP. O el host está mal escrito, o el DNS no lo resuelve (bloqueo, red del café, VPN). **Cómo se lee:** dice *translate host name*: el problema es **el nombre**, no la contraseña. Compara el host carácter por carácter contra la consola. Si es idéntico, es red → "🌐 Si el DNS te bloquea". **Cómo se arregla:** copia la cadena desde *Connect*, no la escribas a mano. Si el nombre está bien, cambia el DNS a 1.1.1.1 o 8.8.8.8. Las otras dos caras: `could not connect to server: Connection refused` es nombre resuelto sin nadie escuchando (puerto `5432`, proyecto suspendido). Y `FATAL: SSL connection is required. Please specify SSL options and retry.` es la cadena sin `?sslmode=require`.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Crea el proyecto en `https://console.neon.tech/`, copia la cadena a tu `.env` como `DATABASE_URL` y haz una consulta de una fila que devuelva `1`. Si eso no corre, nada de lo demás importa.
2. Crea la tabla `ventas` (el DDL de [[01 · PostgreSQL, DDL y carga]]) y carga el CSV. **La carga la resuelves tú** y dejas anotada la decisión que tomaste.
3. Rompe el host a propósito (cámbiale una letra) y corre de nuevo. Guarda ese error. Después quita `?sslmode=require` y guarda el segundo.

### Reto — opcional

1. Mide el arranque en frío con número: deja el proyecto quieto, cronometra la primera consulta y la segunda. Reporta los dos valores y la hora.
2. Carga el CSV dos veces seguidas sin duplicar el conteo. Elige el mecanismo (`truncate` antes, llave primaria, `on conflict`) y explica por qué.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `select count(*) from ventas` en la consola devuelve **el mismo número** que tus filas del CSV
- [ ] La cadena vive en `.env` y `git status` no la muestra
- [ ] Puedes explicar en una frase por qué usaste Neon y no tu Postgres local

## 🔗 Para profundizar

- [Connect to Neon securely](https://neon.com/docs/connect/connect-securely) — `sslmode` y pooler: el capítulo que importa
- [Connection latency and timeouts](https://neon.com/docs/connect/connection-latency) — el arranque en frío explicado por quien lo opera
- [Parámetros de conexión de libpq](https://www.postgresql.org/docs/current/libpq-connect.html) — la tabla de `sslmode` es de Postgres, no de Neon

## 🧭 Problemas frecuentes

- **`password authentication failed for user "..."`** → la contraseña trae `@`, `#` o `%` y en la cadena hay que escaparlos, o la copiaste mal. Regenérala y pega la cadena completa.
- **`relation "ventas" does not exist`** → estás en otra base u otro proyecto. Verifica el nombre en la cadena y corre `select current_database()`.

### Si el DNS te bloquea

Desde Venezuela (y desde varias redes corporativas) el nombre de Neon puede no resolver. **No es tu código.** En orden:

1. **Entra por `https://console.neon.tech/` y trabaja ahí.** La consola abre en el navegador y trae editor SQL: administras el proyecto y corres consultas a mano. Eso funciona aunque el DNS de tu equipo bloquee el endpoint.
2. **Cambia el DNS** del equipo o del router a `1.1.1.1` o `8.8.8.8`. Dos minutos, y te sirve todo el curso.
3. **Prueba las dos cadenas por separado**: la directa y la del pooler. No necesariamente se bloquean las dos; si una pasa, con esa trabajas.
4. **Si nada funciona, la demo se hace en el editor SQL de la consola.** Misma consulta, mismo resultado, cero drama.

Lo importante: **los procesos de nube a nube no dependen de tu red.** GitHub Actions escribiendo en Neon, o Streamlit Cloud leyendo de Neon, corre en los servidores de ellos. El DNS roto te afecta a ti en la terminal, no al pipeline.

## ➡️ Siguiente tema

Vas a hacer que un workflow corra solo en cada push y por horario, y a calcular a qué hora de Caracas cae un cron escrito en UTC → [[01 · GitHub Actions, workflow y cron]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (el conteo, los dos tiempos, el texto de los errores que provocaste). El siguiente tema retoma lo que quede acá.
