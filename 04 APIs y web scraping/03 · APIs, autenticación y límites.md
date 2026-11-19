---
tipo: tema
modulo: "04 APIs y web scraping"
capa: dominio inicial
herramienta: httpx y APIs con clave
resultado: "repos_octocat.csv con los 8 repositorios de octocat, traídos página por página y con la clave fuera del código"
tiempo-lectura: 20 min
prerrequisitos: ["01 · HTTP con httpx"]
tags: [tema, api, httpx, seguridad]
---

# APIs con clave: autenticación, límites y paginación

![[python.svg|64]]

> [!tip] En una frase
> Una API con clave primero te identifica, después te mide y recién entonces te entrega datos: **la clave va en una variable de entorno, el `429` se responde esperando antes de reintentar, y las páginas se recorren con un `while` que termina cuando la API devuelve una lista vacía.**

> [!info] Cómo leer este tema — 20 minutos
> No hace falta entender todo al primer intento: lee completo y, cuando algo no cierre, escribe `#duda` en esa línea. **Lo que NO está acá:** la solución de los ejercicios. Se construye en la práctica. La clave de práctica es un token de GitHub de solo lectura: no hay que pagar ni registrar una aplicación.

## 🤔 Para qué sirve

La API del clima no pidió clave: pediste 90 días de temperatura y te devolvió los 90. **Casi ninguna API de un proveedor real se comporta así:** antes de darte una fila, la que necesitas te pide identificarte con una clave, te cuenta cuántas peticiones te quedan en la hora y te entrega los resultados por páginas de 30.

Un script que ignora las tres funciona una vez en tu computadora y en el servidor falla: sin clave no entra, insistiendo lo bloquean, y sin recorrer páginas trae 30 filas de 3.000. Ese último caso es el peor, porque el `200` con 30 filas no es un error: es una trampa. Nada se rompe, el script termina, y el reporte sale con el 1 % de los datos como si fuera todo.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Autenticarte contra una API con una clave guardada en una variable de entorno, y leer el código de estado antes de usar el cuerpo |
| **Después** | Recorrer una API paginada con un `while`, acumular las páginas en una sola lista y responder a un `429` con `Retry-After` |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Cómo se ve |
|---|---|---|
| **Clave de API** (API key) | El secreto que identifica tu aplicación ante el servidor | `ghp_...` |
| **Variable de entorno** | Un valor que vive en el sistema, fuera del código | `GITHUB_TOKEN` |
| **Header `Authorization`** | La cabecera donde viaja la clave | `Authorization: Bearer ghp_...` |
| **`401`** | No hay credenciales válidas | `401 Unauthorized` |
| **`403`** | Las credenciales se entendieron y no alcanzan | `403 Forbidden` |
| **`429`** | Demasiadas peticiones en la ventana | `429 Too Many Requests` |

## 🧭 Qué no es

- **No es una contraseña ni una línea de código.** La clave identifica a la aplicación, no a la persona, y se limita, se revoca y se rota sin tocar tus datos; escrita en `api.py` termina en el repositorio, en el historial de Git y en la captura que alguien pega en un chat.
- **Un `200` no significa que tengas todo.** Significa que esa página salió bien. En una API paginada, una página es una parte.
- **Reintentar no es insistir.** Repetir a la misma velocidad después de un `429` gasta la cuota que queda y puede alargar el bloqueo.
- **Cuándo no usar una API:** si ese mismo sitio publica un archivo descargable —un CSV mensual, un ZIP, una exportación—, el archivo es más simple y más rápido. Con una cuota de 60 peticiones por hora y 200 páginas necesitas más de tres horas para lo que una descarga resuelve sin código.

## 🧠 Conceptos

1. **La clave es un secreto de despliegue, no una línea de código** — vive en el entorno y, si se filtra, se rota: borrarla del archivo no la desactiva.
2. **El número dice qué hacer y el cuerpo dice por qué** — `401` pide otra clave, `403` pide otro permiso, `429` pide esperar; el texto del cuerpo cambia de API en API.
3. **El límite y la paginación son la misma conversación** — la API te da de a poco y te dice cuánto queda; el script que no escucha las dos cosas entrega la mitad de los datos sin avisar.

## 📖 Teoría

### 1. La clave: qué es y dónde vive

La clave (API key) es la cadena que le dice al servidor quién está pidiendo; en GitHub se llama token, se genera en *Settings → Developer settings → Personal access tokens* y alcanza con uno de solo lectura de datos públicos.

> [!tip] Generar el token, paso por paso — 5 minutos
> Es la parte que más traba a quien nunca lo hizo. No hace falta registrar ninguna aplicación ni pagar nada.
>
> 1. Entra en `github.com` con tu cuenta y ve a **Settings** (tu foto, arriba a la derecha).
> 2. En la columna izquierda, al final: **Developer settings**.
> 3. **Personal access tokens → Tokens (classic)**.
> 4. **Generate new token (classic)**. Pide tu contraseña otra vez.
> 5. En **Note** escribe `curso-datos`. En **Expiration** elige 30 días. **No marques ninguna casilla de permisos.**
> 6. **Generate token** y copia lo que aparece: empieza con `ghp_` y **no se vuelve a mostrar nunca**. Si lo pierdes, generas otro.
>
> Un token sin casillas marcadas solo puede leer información pública: no puede escribir, ni borrar, ni tocar tus repositorios. Ese es exactamente el que necesitas.
>
> Guárdalo en el `.env` como `GITHUB_TOKEN=ghp_...` y comprueba con `grep -n "ghp_" auth.py` que **no** quedó escrito dentro del código.

```python
import os
from dotenv import load_dotenv

load_dotenv()                      # lee el .env de la carpeta actual
clave = os.getenv("GITHUB_TOKEN")  # en el .env: GITHUB_TOKEN=ghp_...
if clave is None:
    raise SystemExit("Falta GITHUB_TOKEN: escríbela en el .env y vuelve a correr.")
print("clave cargada:", len(clave), "caracteres")
```

La clave no va en el código: va en el `.env`, una línea por clave, fuera de Git y con un `.env.example` que lleva solo el nombre. Si se filtra no se borra del archivo: se **rota**, porque la clave sigue viva para quien la copió y también quedó en el historial de Git; revocarla donde la generaste, crear otra y reemplazar el valor es lo único que la desactiva. Para comprobar que no está en el código, `grep -n "ghp_" api.py` no debe devolver nada.

### 2. Cómo se manda: header o URL

La clave viaja en el header `Authorization`, con la palabra `Bearer` antes del token, o como parámetro de la URL. Las dos formas, del lado del cliente:

```python
import httpx

r = httpx.get("https://api.github.com/user",
              headers={"Authorization": f"Bearer {clave}"}, timeout=15)
r.raise_for_status()
print("login:", r.json()["login"])      # 200 y tu usuario
r = httpx.get("https://httpbin.org/get", params={"api_key": "demo"}, timeout=15)
print(r.json()["args"])                 # {'api_key': 'demo'}
print(r.request.url)                    # https://httpbin.org/get?api_key=demo
```

La diferencia no es de estilo: lo que va en la URL queda escrito en los logs del servidor, en los proxies del camino, en el historial del navegador y en cualquier mensaje de error que imprima la dirección. **Se usa el header, no la URL.** El parámetro se acepta solo cuando la API no ofrece otra forma, y para datos sensibles conviene no usar esa API.

### 3. `401` y `403`: quién eres y qué puedes

`401 Unauthorized` quiere decir que la petición no trae credenciales válidas y el servidor no sabe quién eres; `403 Forbidden` quiere decir que el servidor entendió la petición y la rechaza, porque sabe quién eres y el permiso no alcanza.

```python
def revisar(r):
    if r.status_code == 401:
        raise SystemExit("401: la API no reconoce la clave. Revísala o genera otra.")
    if r.status_code == 403:
        raise SystemExit("403: la clave es válida, pero no alcanza para esto.")
    r.raise_for_status()
```

En GitHub el número no distingue el motivo y el cuerpo sí: sin header, o con el header mal escrito, responde `401` con `{"message": "Requires authentication"}`; con una clave que llegó y no sirve, `401` con `{"message": "Bad credentials"}`. Un `403` puede ser permiso insuficiente o cuota agotada, y se distinguen mirando `x-ratelimit-remaining`: si es `0`, no es un problema de permisos. Ninguno de los dos se arregla reintentando; la RFC 9110 lo dice para el `403`, porque las mismas credenciales reciben el mismo rechazo.

### 4. `429` y `Retry-After`: la cuota

El límite de peticiones (rate limit) es una cuota por ventana de tiempo, no un castigo: GitHub permite 60 peticiones por hora sin autenticar y 5.000 por hora con un token, y manda `x-ratelimit-limit`, `x-ratelimit-remaining` y `x-ratelimit-reset`, este último en segundos desde 1970. Cuando la cuota se agota, la respuesta es `429 Too Many Requests`, y el header opcional `Retry-After` dice cuántos segundos conviene esperar antes de volver.

```python
import time, httpx

intento = 1
while intento <= 3:
    r = httpx.get("https://httpbin.org/status/429", timeout=15)
    espera = r.headers.get("retry-after")
    print(intento, r.status_code, "Retry-After:", espera)
    if r.status_code != 429:
        break
    time.sleep(int(espera or 2))
    intento += 1
```

Ese servidor devuelve `429` y no manda el header, así que la salida son tres líneas `1 429 Retry-After: None`, `2 429 Retry-After: None` y `3 429 Retry-After: None`: por eso el código espera dos segundos, el valor por defecto. Reintentar sin esperar empeora el problema, porque cada reintento es una petición más en la misma ventana y la ventana no se reinicia porque insistas; una ráfaga de treinta reintentos en tres segundos gasta lo que queda de cuota y, en APIs como GitHub, escala a un bloqueo secundario más largo. La práctica correcta es mirar `x-ratelimit-remaining` antes de pedir, espaciar las peticiones y, con un `429`, esperar lo que dice el header.

### 5. Paginación: pedir de a páginas

Una API no devuelve 3.000 filas juntas: una respuesta así tarda, ocupa memoria en el servidor y se corta por timeout; entrega páginas y da los parámetros para recorrerlas, en GitHub `per_page` (máximo 100 en la mayoría de los endpoints) y `page`; el ejemplo pide de a 2 para que los 8 repositorios de la cuenta ocupen cuatro páginas.

```python
import httpx

URL = "https://api.github.com/users/octocat/repos"
pagina = 1
repos = []

while True:
    r = httpx.get(URL, params={"per_page": 2, "page": pagina}, timeout=15)
    r.raise_for_status()
    lote = r.json()
    if not lote:            # lista vacía: no hay más páginas
        break
    repos.extend(lote)
    print(f"página {pagina}: {len(lote)} repos, acumulado {len(repos)}")
    pagina += 1

print("total:", len(repos))
```

El `while` no sabe cuántas páginas hay y no le hace falta: para cuando la API devuelve una lista vacía, y no cuando trajo menos de lo pedido, porque la API puede reducir `per_page` al pasarse del máximo y la respuesta sigue siendo válida. `extend` suma los elementos del lote; con `append` quedaría una lista de páginas dentro de la lista. La salida son cuatro líneas de `página N` acumulando de a 2 y un `total: 8`. Conviene además un tope, `if pagina > 50: break`, para que un servidor que nunca devuelve vacío no deje un bucle sin fin; y si la documentación ofrece un header `Link` con la URL de la página siguiente, esa es la forma más robusta: no hay que adivinar cómo se arma la próxima dirección.

## ⚠️ El error más común

> [!warning] `httpx.HTTPStatusError: Client error '401 Unauthorized'`
> ```
> ... (frames internos de httpx, recortados) ...
> httpx.HTTPStatusError: Client error '401 Unauthorized' for url 'https://api.github.com/user'
> For more information check: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401
> ```
> **Qué significa:** la credencial no llegó como el servidor la espera. Con el header incompleto —sin la palabra `Bearer`, por ejemplo— GitHub responde `401` y el cuerpo dice `Requires authentication`, igual que si no hubieras mandado nada. **Cómo se lee:** el `401` descarta el permiso y apunta a la credencial. **Cómo se arregla:** imprime `len(clave)` para confirmar que cargó sin comillas, revisa el header carácter por carácter y, si la clave estaba vencida o revocada, genera otra.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min
1. **(7 min)** Genera un token de solo lectura en GitHub, guárdalo en `.env` como `GITHUB_TOKEN` y escribe `auth.py` que lo lea con `os.getenv` y pida `https://api.github.com/user`. Tiene que imprimir el status y tu login. Después corre `grep -n "ghp_" auth.py`: no debe devolver ninguna línea.
2. **(10 min)** Escribe `paginas.py`: recorre `https://api.github.com/users/octocat/repos` con `per_page=2`, acumula todos los repositorios en una lista y escribe `repos_octocat.csv` con las columnas `nombre,estrellas,lenguaje`, usando `csv.writer` y `newline=""` como en el tema anterior. Tiene que quedar con **8 filas**, y el script imprime cuántas páginas pidió y cuántos repos juntó.
3. **(8 min)** Rompe a propósito dos veces: quita la palabra `Bearer` del header y corre; después deja el header completo con un token inventado. Anota qué mensaje trae el cuerpo en cada caso: los dos son `401`, y lo que cambia entre uno y otro es solo el texto.

### Reto — opcional
1. Escribe `limite.py` que pida `https://httpbin.org/status/429`, lea `Retry-After` con un valor por defecto y reintente como máximo dos veces; antes de pedir, consulta `https://api.github.com/rate_limit` y detente si `x-ratelimit-remaining` es `0`.
2. Cambia la paginación del ejercicio 2 por la del header `Link`: lee `r.headers.get("link")` y sigue la URL marcada con `rel="next"` hasta que no haya. Explica en un comentario qué se rompe con el contador de páginas si la API cambia el orden de los resultados.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `repos_octocat.csv` tiene 9 líneas y `grep -n "ghp_" auth.py` no devuelve nada
- [ ] Distingues `401`, `403` y `429` y sabes qué hace tu código con cada uno
- [ ] Explicas por qué un `200` con una sola página no significa que tengas todos los datos

## 🔗 Para profundizar

- [GitHub — autenticarse en la API REST](https://docs.github.com/en/rest/authentication/authenticating-to-the-rest-api) — el formato del header y el límite de intentos fallidos
- [GitHub — límites de la API REST](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api) — las cuotas por hora, los headers `x-ratelimit-*` y qué hacer con `403` y `429`
- [GitHub — paginación](https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api) — `per_page`, `page` y el header `Link`, con el ejemplo del `rel="next"`
- [RFC 6585, sección 4](https://www.rfc-editor.org/rfc/rfc6585#section-4) — la definición de `429 Too Many Requests`; y [RFC 9110, sección 15.5](https://www.rfc-editor.org/rfc/rfc9110#section-15.5), `401` y `403` con las palabras del estándar
- Anexo del vault: [[01 · HTTP con httpx]] — el pedido, los parámetros y el status, sin clave

## 🧭 Problemas frecuentes

- **`ModuleNotFoundError: No module named 'dotenv'`** → el paquete se instala con `pip install python-dotenv`, aunque se importe como `dotenv`.
- **`401` con una clave que acabas de generar** → falta `Bearer `, sobraron comillas del copiado o el `.env` no se cargó. Imprime `len(clave)` antes de armar el header.
- **`403` con `x-ratelimit-remaining: 0`** → no es un permiso: es la cuota. Espera hasta el valor de `x-ratelimit-reset` o usa un token válido.
- **`429` sin `Retry-After`** → el header es opcional. Usa un valor por defecto y limita los reintentos; un bucle sin tope no arregla la cuota.
- **`AttributeError` al acumular páginas o bucle que no termina** → usaste `append` con el lote (va `extend`), o la API no devuelve vacío: pon el tope de páginas.

## ➡️ Siguiente tema

Los datos ya llegan por la red y quedan en un CSV: el tema siguiente es dónde se juntan, se cruzan y se pueden preguntar cosas sin abrir el archivo entero, **Bases de datos y SQL**.

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá el resultado de `grep -n "ghp_" auth.py` (vacío es lo correcto), cuántas páginas pidió tu script, las 3 primeras filas de `repos_octocat.csv` y los dos mensajes de cuerpo del ejercicio 3.
