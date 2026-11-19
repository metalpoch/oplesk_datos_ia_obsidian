---
tipo: tema
modulo: "04 APIs y web scraping"
capa: dominio inicial
herramienta: httpx
resultado: "clima_caracas.csv con 90 días de temperatura máxima y lluvia"
tiempo-lectura: 15 min
prerrequisitos: ["Listas y diccionarios", "Terminal y VS Code"]
tags: [tema, httpx, http, api, open-meteo]
---

# Traer datos por HTTP: pedirle datos a una API con httpx

![[python.svg|64]]

> [!tip] En una frase, antes de empezar
> **Una API es una dirección web que, en vez de devolverte una página para leer, te devuelve los datos en un formato que un programa entiende, casi siempre JSON.** `httpx` es la herramienta de Python que va a esa dirección, hace el pedido y trae la respuesta.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea.
>
> **Lo que NO está acá:** la solución del ejercicio. Se construye en la práctica, paso a paso.

## 🤔 Para qué sirve

Necesitas el clima de Caracas de los últimos 90 días para cruzarlo con las ventas. Lo buscas, y encuentras una página con una tabla: un día por fila. ¿Y ahora qué?

- Copias las 90 filas a mano a un CSV. Son 90 hoy; el mes que viene son otras 90, y las de ayer ya cambiaron porque el último día se actualiza.
- Guardas la página y esperas que el formato no cambie. Cambió: la tabla está en otra pestaña y el `Ctrl+C` te trae el menú de navegación.
- Necesitas que corra **solo**, mañana y el mes que viene, sin que nadie pegue nada.

**Eso es lo que resuelve una API.** El sitio que muestra esa tabla tiene detrás una dirección que devuelve **el dato en crudo**, sin diseño y sin botones: le pides por nombre los campos que quieres, te contesta JSON, y tú lo conviertes en filas de un CSV. `httpx` hace el pedido desde Python y te deja el resultado en una variable.

> [!note] Qué no es
> - **No es un navegador.** Pide el HTML y lo entrega tal como llegó: no ejecuta JavaScript y no hay DOM que consultar. Ves la respuesta, no la página.
> - **No es scraping.** `httpx` trae el texto; convertirlo en filas —recorrer el HTML y decidir qué es un producto— es el tema siguiente, con `BeautifulSoup`.
> - **No reintenta solo.** Cuando el servidor responde `5xx` o se agota el `timeout`, el pedido se pierde: el reintento lo escribes tú y decides cuántas veces. Un `4xx` no se arregla reintentando, porque el pedido sigue mal.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Pedirle a una API pública 90 días de temperatura y lluvia de Caracas y dejarlos en `clima_caracas.csv`, una fila por día |
| **Después** | Hacer que ese script lea el código de estado antes de usar el cuerpo, no muera cuando la API rechaza el pedido y reintente una vez |

Hasta acá los datos te los daban. A partir de aquí los vas a buscar: la tienda necesita saber si un día de lluvia vende menos, y el clima no está en tu computadora, está en un servidor.

## 🧠 Conceptos

1. **La petición es un contrato** — le pides campos por nombre; si el nombre está mal escrito, la API no adivina: falla.
2. **El status va primero, el cuerpo después** — un 400 también trae cuerpo, pero no trae tus datos.
3. **El JSON anidado hay que aplanarlo** — lo que llega es un árbol; lo que necesitas es una fila.

## 📖 Teoría

### 1. GET con parámetros: la URL es un formulario
Un `GET` pide algo que ya existe, y lo que pides va en la *query string*: lo que sigue al `?`, pares `clave=valor` unidos por `&`. En vez de armarla a mano, le pasas un diccionario a `params`:

```python
import httpx

URL = "https://api.open-meteo.com/v1/forecast"
params = {
    "latitude": 10.4806, "longitude": -66.9036,       # dónde
    "daily": "temperature_2m_max,precipitation_sum",  # qué campos
    "past_days": 90, "forecast_days": 0,              # cuántos días
    "timezone": "America/Caracas",                    # en qué reloj
}
r = httpx.get(URL, params=params)
print(r.request.url)
```

Lee cada clave como una frase: *"dame, para estas coordenadas, el máximo diario y la suma de lluvia de los últimos 90 días, sin pronóstico, en hora de Caracas"*. `forecast_days=0` importa: con el valor por defecto te vienen 91 días y el CSV no cuadra. Open-Meteo es gratis y no pide API key; por eso lo elegimos. Si la API no responde o no tienes internet, en la carpeta `datos/` viene una copia de esos mismos 90 días para que puedas seguir con el cruce.

**Por qué importa:** si pegas la URL con `+`, un espacio o un `&` dentro de un valor la corta. `params` lo escapa y deja escrito qué pides.

### 2. El status primero, el cuerpo después
`2xx` salió bien. `4xx` **tu pedido está mal** (parámetro inválido, ruta que no existe): reintentar no arregla nada. `5xx` el problema es del servidor: ahí sí vale reintentar.

```python
r = httpx.get(URL, params=params)
r.raise_for_status()   # si NO es 2xx, corta acá con un error claro
datos = r.json()       # recién ahora el cuerpo tiene datos que te sirven
```

Sin ese `raise_for_status()`, un 400 te devuelve un cuerpo **sin** la clave `daily`, y el script explota tres líneas después con un `KeyError` que no dice nada. Distinguir "me equivoqué yo" de "se cayó el servicio" separa corregir de reintentar.

### 3. De JSON anidado a filas
Open-Meteo devuelve un objeto con **listas paralelas**: la posición 0 de cada lista es el mismo día.

```python
diario = datos["daily"]   # {"time": [...], "temperature_2m_max": [...], "precipitation_sum": [...]}
filas = list(zip(diario["time"], diario["temperature_2m_max"], diario["precipitation_sum"]))
print(filas[0])    # ('2026-08-18', 28.1, 2.9)
```

Cada API tiene su forma: unas dan listas paralelas y otras una lista de diccionarios. Tu trabajo es el mismo: leer la forma, recorrerla y devolver filas.

## 🧠 `zip()`, que aparece aquí por primera vez

> [!info] Pegar tres listas en filas
> La API no devuelve filas: devuelve **una lista por variable**. `diario["time"]` es la lista de fechas, `diario["temperature_2m_max"]` la de temperaturas, y así. Están separadas, y para escribir el CSV hacen falta pegadas.
>
> `zip()` toma una posición de cada lista y arma una tupla:
>
> ```python
> fechas = ["2024-01-01", "2024-01-02"]
> temps  = [24.5, 26.1]
> lluvia = [0.0, 3.2]
>
> for fila in zip(fechas, temps, lluvia):
>     print(fila)
> # ('2024-01-01', 24.5, 0.0)
> # ('2024-01-02', 26.1, 3.2)
> ```
>
> Se corta en la lista más corta: si una trae 90 valores y otra 89, la fila 90 no existe y **no avisa**. Por eso conviene comprobar los tres largos antes de escribir el archivo.

## ⚠️ El error más común

> [!warning] `httpx.HTTPStatusError: Client error '400 Bad Request'`
> ```
> Traceback (most recent call last):
>   File "clima.py", line 13, in <module>
>     r.raise_for_status()
>   ... (frames internos de httpx, recortados) ...
> httpx.HTTPStatusError: Client error '400 Bad Request' for url '...daily=temperatura_2m_max&past_days=90...'
> ```
> **Qué significa:** la API recibió tu pedido y lo rechazó. No se cayó: no entendió lo que le pediste. Casi siempre es un nombre de campo mal escrito. **Cómo se lee:** el `400` ya dice que la culpa es del pedido, no del servidor. Mira lo que va después de `daily=` y, antes de suponer, imprime `r.status_code` y `r.json()["reason"]`. **Cómo se arregla:** ese `reason` devuelve `Invalid value: Cannot initialize ForecastVariableDaily from invalid String value temperatura_2m_max`, o sea: el campo no existe. Corrígelo contra la documentación (`temperature_2m_max`, no `temperatura_2m_max`); los nombres están en inglés y en `snake_case`, no se traducen.

Este no es un error de adorno: es el que más se repite si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min
Los haces **en tu terminal o en VS Code**, no acá. Al terminar, pega la salida en "Mi resultado".

1. **(10 min)** Pide 90 días de `temperature_2m_max` y `precipitation_sum` para Caracas y escribe `clima_caracas.csv` con encabezado `fecha,temp_max_c,lluvia_mm`. Tiene que quedar con **90 filas**: ni 89 ni 91. Escribe con `csv.writer` y `newline=""`.
2. **(5 min)** Antes de guardar nada, imprime el status, cuántos días vino y el primer registro. Nada se escribe sin haberlo mirado.
3. **(10 min)** Rompe a propósito: escribe `temperatura_2m_max` y corre. Lee el error completo. Después saca el `raise_for_status()` y pon un `print` del status y del `reason`. ¿Cuál versión te dice más?

### Reto — opcional, para quien quiera más
1. Parametriza latitud y longitud por línea de comandos y guarda un CSV por ciudad (`python clima.py 10.48 -66.90 maracay`).
2. Pon `timeout=15` y reintenta **una sola vez** si el status es `5xx` o si salta `httpx.TimeoutException`. Imprime en qué intento salió.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `wc -l clima_caracas.csv` devuelve 91 (90 datos + encabezado)
- [ ] Tu script imprime el status antes de tocar el cuerpo de la respuesta
- [ ] Provocaste el 400 a propósito y sabes leer el `reason` que devuelve la API
- [ ] Explicas en una frase por qué usaste `params` y no la URL armada a mano

## 🔗 Para profundizar

- [httpx — QuickStart](https://www.python-httpx.org/quickstart/) — "Passing parameters in URLs" y "Response status codes"; y [excepciones](https://www.python-httpx.org/exceptions/), qué falla da cada una
- [Open-Meteo — documentación](https://open-meteo.com/en/docs) — la tabla de parámetros y las variables `daily`
- Anexo del vault: [[04 · Listas y diccionarios]]

## 🧭 Problemas frecuentes

- **`httpx.ConnectError: [Errno -2] Name or service not known`** → o no hay internet, o hay un typo (`api.open-metio.com`). Prueba la URL en el navegador.
- **`json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)`** → te devolvieron HTML, no JSON: un 404 o una página de error. Imprime `r.status_code` y `r.headers["content-type"]` antes de `r.json()`.
- **`KeyError: 'daily'`** → el cuerpo no tiene tus datos porque el pedido falló. Vuelve al concepto 2.
- **`ModuleNotFoundError: No module named 'httpx'`** → lo instalaste en otro Python: activa tu entorno virtual y repite `pip install httpx`.

Si nada de esto lo resuelve, anota el status y el error completo en "Mi resultado": de ahí se retoma el tema.

## ➡️ Siguiente tema

Cuando el dato no esté en ninguna API, vas a tener que sacarlo de una página web; y ahí el problema deja de ser técnico y pasa a ser si está bien hacerlo.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá el código de estado que devolvió la API, las 3 primeras filas de `clima_caracas.csv` y cuántos días te quedaron. Si falló, pega el error completo: con el status y el mensaje alcanza para ubicarlo sin que tengas que explicarlo.