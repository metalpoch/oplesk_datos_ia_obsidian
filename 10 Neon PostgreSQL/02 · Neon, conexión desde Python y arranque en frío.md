---
tipo: tema
modulo: "10 Neon PostgreSQL"
capa: dominio inicial
herramienta: Neon y psycopg
resultado: "un script que se conecta a Neon con la cadena guardada en `.env`, lee el conteo de `ventas` y avisa cuando la base está despertando"
tiempo-lectura: 20 min
prerrequisitos: ["01 · Neon, Postgres en la nube"]
tags: [tema, neon, postgresql, python]
---

# Neon desde Python: la cadena de conexión, el arranque en frío y el pooler

![[neon.svg|64]] ![[postgresql.svg|64]]

> [!tip] En una frase
> La **cadena de conexión** es una línea de texto con la dirección de la base, el usuario, la contraseña y el cifrado; `psycopg` la usa para abrir una conversación con Postgres desde Python, y en el plan gratuito esa conversación puede empezar con una espera porque el cómputo estaba suspendido.

> [!info] Cómo leer este tema — 20 minutos
> No da nada por sabido: cada parte de la cadena, cada línea de código y cada mensaje de error se explica desde cero; si algo no cierra, escribe `#duda` en esa línea. Lo que se da por hecho es la tabla `ventas` cargada en Neon y la cadena a mano, las dos cosas de [[01 · Neon, Postgres en la nube]].

## 🤔 Para qué sirve

La tabla está en Neon, pero la única manera de consultarla es el editor SQL del navegador: alguien escribe la consulta, la ejecuta a mano y lee el resultado en pantalla. Eso no es un programa. Un programa hace la misma consulta solo, sin nadie mirando, y con el resultado hace algo: lo guarda, lo grafica, lo manda por correo.

Hay un segundo problema que no aparece en el navegador. En el plan gratuito el **cómputo** (la máquina donde corre Postgres) se suspende cuando nadie consulta, y la primera consulta después de ese silencio tarda o falla. Si tu programa no lo sabe, tú tampoco: verás un error y creerás que rompiste algo. Las condiciones exactas del plan se consultan en la documentación de Neon, no se memorizan: lo que cambia es el número, no la conducta.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Conectar desde Python con la cadena guardada en `.env`, ejecutar `select count(*) from ventas` y leer el resultado con `with`, sin dejar la conexión abierta |
| **Después** | Medir cuánto tarda la primera consulta tras la inactividad y avisar en la aplicación en vez de mostrar un error seco |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Ejemplo |
|---|---|---|
| **Cadena de conexión** | un texto con los datos para llegar a la base | `postgresql://usuario:password@host/base?sslmode=require` |
| **`sslmode=require`** | exige que el canal vaya cifrado; Neon rechaza lo que no | `?sslmode=require&channel_binding=require` |
| **`psycopg`** | el driver: la biblioteca de Python que habla con Postgres | `import psycopg` |
| **Conexión y cursor** | el canal abierto contra el servidor y el objeto que envía la consulta | `psycopg.connect(url)`, `cur.fetchone()` |

## 🧭 Qué no es

- **`psycopg` no es Postgres.** Es el cliente. No instala ni administra un servidor; solo sabe hablar con uno que ya existe.
- **El arranque en frío no es una falla ni una pérdida de datos.** El cómputo se apaga, el dato sigue guardado. Lo que cambia es el tiempo de la primera consulta.
- **Ni toda la nube hace falta.** Si el dato es de una sola persona, vive en su máquina y nadie escribe desde otro sitio, un archivo Parquet leído con DuckDB alcanza y sobra: sin latencia, sin secreto que cuidar, sin arranque en frío y sin depender de la red. La base remota se justifica cuando varias personas leen y escriben lo mismo, o cuando el proceso que produce el dato corre en otro servidor.

## 🧠 Conceptos

1. **La cadena es la dirección completa, y cada pedazo falla distinto** — host mal escrito no resuelve; contraseña mal, rechazo de autenticación; sin cifrado, el servidor corta. Eso permite diagnosticar leyendo el mensaje, sin adivinar.
2. **Con `psycopg` se abre, se ejecuta, se lee y se cierra — y `with` lo hace por ti** — una conexión es un recurso del servidor; si no la cierras, se acumulan hasta que deja de aceptar nuevas.
3. **El cómputo se suspende por inactividad, y eso tiene dos caras** — la primera consulta tarda, y una conexión que quedó abierta puede morir con el cómputo suspendido. Se mide y se avisa; no se "arregla" con un ping cada minuto.

## 📖 Teoría

### 1. La cadena de conexión, parte por parte

Neon te entrega una línea larga. Ahí están todos los datos, y cada uno tiene su propio error.

| Parte | Qué es | Si está mal |
|---|---|---|
| `neondb_owner` | el rol de la base (Neon lo crea con el proyecto) | `password authentication failed` |
| `AbC123dEf` | la contraseña del rol | `password authentication failed` |
| `ep-cool-name-123456…neon.tech` | el host: identificador del endpoint + región | `could not translate host name` |
| `neondb` | el nombre de la base | `relation "ventas" does not exist` |
| `sslmode=require` | exige canal cifrado | el servidor rechaza la conexión |

**Dónde se consigue:** en la consola, botón **Connect**. La cadena no se escribe a mano: se copia completa. No aparece el puerto porque Postgres usa `5432` por defecto, y si trae parámetros extra como `channel_binding=require`, no los borres. Si la contraseña tiene `@`, `#`, `%` o `:`, esos caracteres van codificados dentro de la URL, y escribirlos a mano es la causa más común de un fallo de autenticación con la contraseña correcta.

### 2. Conectarse desde Python

Se instalan dos cosas con `pip install "psycopg[binary]" python-dotenv`: el driver y el lector del `.env`. El paquete se llama `psycopg`, sin el `3`, y esta es la versión 3: no existe un paquete `psycopg3`.

```python
import os
import psycopg
from dotenv import load_dotenv

load_dotenv()
url = os.environ["DATABASE_URL"]

with psycopg.connect(url, connect_timeout=15) as conn:
    with conn.cursor() as cur:
        cur.execute("select count(*) from ventas")
        total = cur.fetchone()[0]

print(f"{total} filas")
```

Cinco piezas, en orden: `connect()` abre la conexión contra el servidor; `cursor()` crea el objeto que envía consultas; `execute()` manda el SQL; `fetchone()` devuelve la primera fila como tupla y `[0]` toma su primer campo; `with` cierra el cursor al salir del bloque interno y la conexión al salir del externo.

Ese es el motivo de usar `with` en vez de dejarlo al olvido: en `psycopg` 3 la conexión **sí se cierra** al terminar el bloque, incluso si hubo un error en medio; sin `with` quedarían conexiones abiertas acumulándose. Para una consulta sola hay un atajo con el mismo efecto: `conn.execute("select count(*) from ventas").fetchone()[0]`.

### 3. La credencial va en el entorno, no en el código

En el archivo `.env`, una línea:

```text
DATABASE_URL=postgresql://neondb_owner:AbC123dEf@ep-cool-name-123456.us-east-2.aws.neon.tech/neondb?sslmode=require&channel_binding=require
```

El código se comparte, se sube al repositorio y se muestra en una demostración; la cadena, no. `load_dotenv()` lee el archivo y carga esas líneas como variables de entorno, y después `os.environ["DATABASE_URL"]` las recupera. El `.env` se agrega al `.gitignore` y se versiona un `.env.example` con el valor vacío para que otro sepa qué hace falta.

Si la cadena se filtra —un commit, una captura, un mensaje— hay que **rotarla**, no borrar el mensaje: cualquiera con esa línea entra a tu base, lee todo y escribe todo. Rotar es regenerar la contraseña del rol en la consola, actualizar el `.env` y los secretos de donde corra el programa, y revisar si hubo conexiones que no reconoces. Borrar el archivo no alcanza: si estuvo en un commit, sigue en el historial.

### 4. Arranque en frío: medirlo y avisarlo

Cuando nadie consulta por un rato, el cómputo se suspende. La primera consulta lo despierta y ahí pagas la espera; las siguientes van rápido. **No se pierde ningún dato.** La documentación de Neon describe ese despertar en cientos de milisegundos; lo que tú midas incluye además la distancia hasta la región y tu red, así que dos personas con el mismo proyecto obtienen números distintos y ninguna está equivocada.

La misma conexión, ahora como función: mide cada intento, avisa antes de la espera y reintenta solo donde sirve, usando el `url` y los `import` del bloque anterior.

```python
import time

def contar_ventas() -> int:
    print("Consultando la base. Si estuvo inactiva, puede tardar unos segundos.")
    for intento in (1, 2):
        t0 = time.perf_counter()
        try:
            with psycopg.connect(url, connect_timeout=15) as conn:
                total = conn.execute("select count(*) from ventas").fetchone()[0]
            print(f"Intento {intento}: {time.perf_counter() - t0:.2f} s -> {total} filas")
            return total
        except psycopg.errors.ConnectionTimeout:
            print(f"Intento {intento}: la base no respondió todavía.")
        except psycopg.OperationalError as error:
            print(f"Intento {intento}: {error}")
            raise
    raise RuntimeError("La base no respondió después de dos intentos.")
```

El orden de los `except` importa: `ConnectionTimeout` es un caso particular de `OperationalError`, así que va primero; al revés, nunca se ejecutaría. El segundo `except` vuelve a lanzar el error en vez de reintentar, porque una contraseña equivocada no se arregla esperando. Los dos números que imprime son los que justifican el `connect_timeout`: algo como `3,41 s` en el primer intento y `0,06 s` en el segundo; si tu primera consulta tarda 3 segundos, un límite de 2 corta justo cuando estaba funcionando.

### 5. Cadena directa o cadena del pooler

La consola ofrece dos cadenas. Son la misma base y el mismo usuario; la diferencia está en el host: la del pooler lleva `-pooler` antes de la región. Postgres crea un proceso del sistema por cada conexión abierta y eso tiene un techo; el pooler mantiene un grupo chico de conexiones reales y las presta a muchos clientes, uno por vez, así que no acelera tu consulta: evita tumbar el servidor a fuerza de abrir conexiones.

| Situación | Cadena |
|---|---|
| Aplicación web o dashboard que atiende a varias personas | **pooler** |
| Migraciones, `pg_dump`, cargas grandes, análisis largos | **directa** |

La regla práctica: la aplicación que atiende usuarios va por el pooler; las tareas de administración van por la directa. El motivo es que el pooler trabaja en modo transacción y no conserva el estado entre una consulta y la siguiente: un `SET search_path` puede quedar aplicado en otra conexión, o perderse. No hace falta configurar nada más: se elige la cadena y se usa igual que la otra.

## ⚠️ El error más común

> [!warning] `password authentication failed for user "neondb_owner"` — y su hermano `connection is insecure`
> **Qué significa y cómo se lee:** el servidor te reconoció el host y te pidió autenticarte, así que el problema está en lo que mandaste, no en la red. Busca el final del mensaje: si dice `FATAL:  password authentication failed for user "neondb_owner"`, falló la contraseña; si dice `connection is insecure`, la cadena perdió `sslmode=require` y el problema es el canal, no la credencial. El texto documentado por Neon para el segundo caso es `ERROR: connection is insecure (try using sslmode=require&channel_binding=require)`, y llega envuelto en `port 5432 failed:`. **Cómo se arregla:** copia la cadena completa desde **Connect** y pégala sin editar; si la editaste, revisa que la contraseña conserve los caracteres codificados y que no hayas cortado la línea al copiar. Provoca los dos errores y guarda el texto exacto que te sale: cambia un poco según la versión del cliente, y ese es el que vas a buscar cuando vuelva a pasar.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Escribe `contar.py`: carga el `.env`, abre la conexión con `with`, ejecuta `select count(*) from ventas` e imprime el total. El número tiene que coincidir con el conteo que validaste en la consola. Después corre `git status` y comprueba que el `.env` no aparece.
2. Rompe la cadena dos veces y guarda los dos mensajes: primero quítale `?sslmode=require&channel_binding=require`; después cambia un carácter de la contraseña. Para cada uno, escribe en una línea qué parte de la cadena falló y cómo lo supiste leyendo el mensaje.
3. Mide con `time.perf_counter()`: dos corridas seguidas y una tercera cuando la base estuvo inactiva el tiempo suficiente para suspenderse. Con el número más alto, elige el `connect_timeout` de tu script y escribe por qué ese valor y no otro.

### Reto — opcional

1. Convierte el script en la función `contar_ventas()` con aviso y un reintento para el caso de espera. Después provoca una contraseña incorrecta y comprueba que **no** reintenta: el error sale en el primer intento. Explica en dos líneas por qué esa diferencia es correcta.
2. Corre el mismo script con la cadena directa y con la del pooler. Anota el tiempo de cada una, si alguna falla por resolución de nombre y cuál dejas en `.env` para la aplicación. Justifícalo con lo que mediste.

## ✅ Criterios de logro

- [ ] `contar.py` imprime el conteo de `ventas`, la cadena no aparece dentro del `.py` y `git status` no muestra el `.env`
- [ ] Sabes qué parte de la cadena falló cuando ves `password authentication failed` y cuando ves `connection is insecure`
- [ ] Tienes tres tiempos medidos con `perf_counter` y un `connect_timeout` elegido a partir de ellos
- [ ] Distingues cuándo usas la cadena directa y cuándo la del pooler, y puedes dar el motivo

## 🔗 Para profundizar

- [Basic module usage — psycopg 3](https://www.psycopg.org/psycopg3/docs/basic/usage.html) — el `with`, el atajo `execute().fetchone()` y qué hace al cerrar
- [Connection errors — Neon](https://neon.com/docs/connect/connection-errors) — los textos de error tal como los documenta quien opera el servicio
- [Connection latency and timeouts — Neon](https://neon.com/docs/connect/connection-latency) — el arranque en frío y las formas de manejarlo
- [Connection pooling — Neon](https://neon.com/docs/connect/connection-pooling) — pooler contra directa, y qué no funciona en modo transacción

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `KeyError: 'DATABASE_URL'` | el `.env` no se cargó, está en otra carpeta o la variable tiene otro nombre | llamar `load_dotenv()` antes de leer y confirmar con `print(bool(os.environ.get("DATABASE_URL")))` |
| `ModuleNotFoundError` con `psycopg` o `dotenv` | se instaló en otro intérprete, o falta `python-dotenv` | instalar con el mismo Python que ejecuta el script |
| `terminating connection due to administrator command` | la conexión quedó abierta y el cómputo se suspendió | no guardar la conexión para siempre: abrir, consultar y cerrar |

### Si en tu red el DNS no resuelve

Desde Venezuela y desde varias redes corporativas el nombre de Neon puede no traducirse a una dirección IP. **No es tu código**, y `could not translate host name` lo dice: el problema está en el nombre, no en la contraseña. Entra a `https://console.neon.tech/` y trabaja desde ahí, porque la consola abre en el navegador y no depende del endpoint; después prueba **la cadena directa y la del pooler por separado**, porque no necesariamente se bloquean las dos y con la que pase trabajas. Si tu programa corre en GitHub Actions o en un servicio en la nube, tu red no interviene: el proceso sale desde otro servidor.

## ➡️ Siguiente tema

Con la conexión resuelta, el programa puede correr solo: un workflow de GitHub Actions que se ejecuta por horario, escribe en Neon y avisa cuando falla → [[01 · GitHub Actions, workflow y cron]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega el total que imprimió `contar.py`, los dos textos de error que provocaste, los tres tiempos medidos con su `connect_timeout` y cuál de las dos cadenas dejaste en el `.env`.
