---
tipo: tema
modulo: "12 Telegram Bot API"
capa: familiaridad
herramienta: Telegram Bot API
resultado: "tres mensajes (ok / aviso / error) en el chat del equipo, disparados desde el pipeline"
tiempo-lectura: 15 min
prerrequisitos: ["GitHub Actions, secretos y alertas"]
tags: [telegram, api, automatizar]
---

# Telegram: el bot que avisa cuando el pipeline falla

![[telegram.svg|64]]

> [!tip] En una frase, antes de empezar
> **Un bot de Telegram es una cuenta que maneja un programa tuyo, no una persona: le mandas un mensaje por su API y aparece en tu teléfono como cualquier otro chat.**

> [!info] Lee esto antes de empezar — 15 minutos
> Léelo completo y marca `#duda` donde no te cierre. **No está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

El pipeline se cayó un domingo a las 3 de la mañana. El run quedó rojo, la alerta quedó en un log, y te enteraste el miércoles cuando alguien preguntó por qué el número no se movía. Tres días mirando un dashboard con datos viejos sin saberlo.

Mientras tanto, el chat del equipo lo revisas diez veces al día. El aviso ya existe: el problema es que vive en el lugar equivocado.

**Eso es lo que resuelve un bot:** el aviso del pipeline llega a la app que ya tienes abierta, en el canal correcto. Un `POST` a una API HTTP y el mensaje aparece en el teléfono.

> [!note] Qué no es
> Un log es para cuando alguien lo va a leer; el teléfono es para enterarse. Y no lo uses para mandar un mensaje por fila: el límite de tasa te corta justo cuando más lo necesitas. Un mensaje con el resumen, no cincuenta con el detalle.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Crear un bot y hacer que te mande un mensaje a un chat, desde tu terminal |
| **Después** | Que el pipeline avise por Telegram cuando falla, con un mensaje que diga qué se cayó |

En el tema anterior la alerta era un `echo` en un log que nadie mira. Aquí llega al teléfono del equipo.

## 🧠 Conceptos

1. **BotFather y el token** — el bot se crea con un chat de Telegram; el token es su contraseña y va a los secretos del repo, nunca al código.
2. **El `chat_id`** — a quién le habla. Se obtiene de `getUpdates` después de escribirle al bot, y es negativo en grupos y canales.
3. **`sendMessage` por HTTPS** — es una API HTTP como la de [[01 · HTTP con httpx]]: un POST, un JSON, un código de estado. Y tiene límite de tasa.

## 📖 Teoría

### 1. BotFather y el token

En Telegram buscas `@BotFather`, mandas `/newbot`, eliges nombre y usuario, y te devuelve algo así:

```text
Use this token to access the HTTP API:
7123456789:AAF3kQ9x2vLmT8pR1sN4uY6wZ0aB7cD2eFg
```

Ese token **es** el bot: con él se manda y se lee todo. Va en *Settings → Secrets and variables → Actions* como `TELEGRAM_TOKEN`. Si se filtra —lo mostraste en una captura, lo pegaste en el chat del curso, lo subiste en un commit— **el bot está comprometido**: `/revoke` en BotFather y token nuevo.

**Por qué importa:** un token en un log o en una grabación es una puerta abierta a un bot que puede leer y escribir mensajes. Por eso se usa un **bot de práctica** y se revoca al terminar.

### 2. El `chat_id`: a quién le habla

Primero le escribes algo al bot (o lo agregas a un grupo). Después pides las novedades:

```bash
curl -s "https://api.telegram.org/bot<TU_TOKEN>/getUpdates"
```

En la respuesta buscas `"chat": {"id": ...}`. Nota: en grupos y canales el id es **negativo** (los canales arrancan con `-100`). Ese número, que suele tener 10 dígitos, es el que después se equivoca todo el mundo.

### 3. `sendMessage`: es una API HTTP

```python
import os, httpx

def avisar(texto: str) -> None:
    url = f"https://api.telegram.org/bot{os.environ['TELEGRAM_TOKEN']}/sendMessage"
    r = httpx.post(url, json={"chat_id": os.environ["TELEGRAM_CHAT_ID"], "text": texto}, timeout=10)
    print(r.status_code, r.json()["ok"])   # 200 True
```

La respuesta lo dice sin vueltas: `{"ok":true,...}` o `{"ok":false,...}`. Y hay un detalle que arruina el ejercicio: **los límites de tasa**. Un mensaje por segundo por chat, unos 20 por minuto en un grupo. Si el pipeline manda **un mensaje por fila**, a la fila 30 empiezan los `429` y el aviso se corta justo cuando más lo necesitas. Se manda **un** mensaje con el resumen.

> [!caution] El token también viaja en la URL
> Esta API lleva el token adentro de la dirección. Si usas `r.raise_for_status()` o imprimes `r.request.url`, el mensaje de error incluye la URL completa **con el token**. Captura el error y muestra solo el status y el `description`.

## ⚠️ El error más común

> [!warning] `{"ok":false,"error_code":400,"description":"Bad Request: chat not found"}`
> ```json
> {"ok":false,"error_code":400,"description":"Bad Request: chat not found"}
> ```
> **Qué significa:** el `chat_id` que mandaste no es uno al que el bot pueda escribir. O el número está mal, o el bot nunca estuvo en ese chat. **Cómo se lee:** `ok:false` te dice que falló; `description` te dice por qué. El `error_code` separa los casos: `400` es pedido mal armado (id o texto), `403` es que el bot está pero no tiene permiso, `429` es que mandaste demasiado rápido. **Cómo se arregla:** saca el id de `getUpdates` en vez de copiarlo de memoria. Si es un canal, el bot tiene que ser **administrador**; si es un grupo, tiene que estar adentro; si es un chat privado, tienes que haberle escrito `/start` al menos una vez.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Crea un bot de práctica con BotFather, abre el chat con tu bot nuevo, mándate `/start`, saca tu `chat_id` con `getUpdates` y mándate un mensaje desde la terminal. Guarda la respuesta completa.
2. Arma los tres mensajes del resultado: `✅ ok`, `⚠️ aviso` y `❌ error`, con el prefijo y una línea de contexto (qué corrió, a qué hora de Caracas).
3. Rompe el `chat_id` a propósito (sácale un dígito, o prueba un canal donde el bot no es admin) y pega la respuesta exacta que devuelve la API.

### Reto — opcional

1. Un solo mensaje de resumen con tres líneas (filas cargadas, fallos, hora). Nada de un mensaje por fila.
2. Si la respuesta es `429`, espera lo que diga `parameters.retry_after` y reintenta una vez. Explica por qué no reintentas en bucle.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] El bot te mandó un mensaje y sabes de dónde salió ese `chat_id`
- [ ] Los tres mensajes (ok / aviso / error) llegan distintos y legibles al chat del equipo
- [ ] Reprodujiste el `chat not found` y sabes cuál de las tres causas era la tuya
- [ ] Puedes explicar en una frase por qué el token va a los secretos y no al YAML

## 🔗 Para profundizar

- [sendMessage](https://core.telegram.org/bots/api#sendmessage) — todos los parámetros, incluido `parse_mode` y `disable_notification`
- [Bots FAQ: límites](https://core.telegram.org/bots/faq) — la sección de *broadcasting*, con los números de tasa
- [Tutorial oficial de bots](https://core.telegram.org/bots/tutorial) — el recorrido completo, desde BotFather
- [httpx: quickstart](https://www.python-httpx.org/quickstart/) — el mismo cliente de [[01 · HTTP con httpx]]

## 🧭 Problemas frecuentes

- **El token quedó en pantalla** → revócalo ya con `/revoke` en BotFather y genera otro. Un token visto en una grabación es un bot comprometido, aunque "no lo haya visto nadie".
- **`429 Too Many Requests`** → mandaste muy seguido. Un mensaje por segundo por chat, ~20 por minuto por grupo. Manda un resumen, no un mensaje por fila.
- **`400 ... can't parse entities`** → pusiste `parse_mode: "Markdown"` y el texto trae `_` o `*` sueltos (nombres de archivo, rutas). O escapas, o mandas texto plano.
- **`400 ... message is too long`** → el texto pasa los 4096 caracteres. Recorta o parte en dos mensajes con una pausa.

## ➡️ Siguiente tema

Vas a pedirle un cambio a un agente, correr tus tests y descubrir que la propuesta rompe la mitad. Sin tests, ese error llega a producción → [[Agentes, el agente como compañero]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la captura de los tres mensajes y el JSON exacto del `chat not found`. El siguiente tema retoma lo que quede acá.

---
![[meme-telegram.jpg]]