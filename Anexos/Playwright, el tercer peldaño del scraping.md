---
tipo: anexo
modulo: "Anexos"
capa: familiaridad
herramienta: Playwright
resultado: "el mismo CSV de productos, sacado de una página que renderiza con JavaScript"
tiempo-lectura: 20 min
prerrequisitos: ["Scraping responsable con BeautifulSoup", "HTTP con httpx"]
tags: [anexo, scraping, playwright, javascript]
---

# Playwright: el tercer peldaño del scraping

> [!tip] En una frase, antes de empezar
> **Playwright abre un navegador de verdad desde Python y hace lo que haría una persona: espera a que la página se arme, y recién ahí la lee.** No reemplaza a `BeautifulSoup`: le entrega el HTML que `BeautifulSoup` no podía ver.

> [!info] Cuándo leer esto — 20 minutos
> Es opcional. El tema de scraping se resuelve sin Playwright, con una copia local del HTML. Leelo cuando tengas delante un sitio que en el navegador muestra los datos y con `httpx` devuelve una página casi vacía.

## 🤔 Para qué sirve

Abres el sitio de la competencia en el navegador: hay 20 productos, con precio y stock.

Corres `httpx.get(url)` y guardas el HTML. Lo abres y encuentras esto:

```html
<div id="root"></div>
<script src="/static/js/main.8f3a.js"></script>
```

Veinte productos en pantalla, cero en el archivo. No está roto el scraping: **lo que bajaste es el andamio, no la casa.** El contenido lo construye JavaScript después de que la página carga, y `httpx` no ejecuta JavaScript — nunca lo hizo.

Playwright resuelve exactamente eso: en vez de pedir el HTML, **abre un navegador**, deja que la página se arme, y después te entrega el resultado. Es el tercer peldaño de la escalera que viste en el tema de scraping, y el último antes de rendirse.


## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Reconocer cuándo un sitio necesita navegador y no alcanza el HTML |
| Después | Instalar Playwright y extraer un dato que solo aparece al ejecutar JavaScript |
## 🧠 Qué es cada cosa

| Pieza | Función |
|---|---|
| `playwright` (el paquete) | Controla el navegador desde Python |
| Chromium (el navegador) | El que renderiza de verdad la página. Se descarga aparte |
| `página` (page) | Una pestaña del navegador |
| `locator` | Forma de buscar un elemento y **esperarlo** hasta que aparezca |

## 🧭 Qué no es

- **No es una librería de HTTP.** No reemplaza a `httpx`. Es un navegador controlado por código.
- **No es más rápido ni más robusto.** Es al revés: cada página tarda segundos en vez de milisegundos, y un navegador tiene más formas de fallar que una petición.
- **No es necesario si hay una API o un JSON.** Si el sitio ofrece los datos limpios, ese es el camino. Playwright es para cuando no hay otra.

## 📖 Teoría

### 1. Instalación: dos pasos, no uno

```bash
pip install playwright
playwright install chromium
```

El primer comando instala la librería. El segundo descarga el navegador (~281 MB). Son dos pasos porque el navegador se actualiza por su cuenta: si actualizas Playwright, puede pedirte volver a correr `playwright install`.

> [!tip] Si solo vas a usar el navegador sin ventana
> `playwright install --only-shell chromium` descarga solo la versión sin interfaz, bastante más liviana. Es lo que se usa en un servidor o en un pipeline.

### 2. El esqueleto de cualquier script

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    navegador = p.chromium.launch()      # abre el navegador
    pagina = navegador.new_page()        # abre una pestaña
    pagina.goto("https://ejemplo.com")
    html = pagina.content()              # el HTML ya renderizado
    navegador.close()
```

Las tres cosas que no se pueden olvidar: el `with` (es el que cierra todo), el `new_page()` (el navegador solo no hace nada) y el `close()`.

### 3. La combinación que casi nadie explica: Playwright + BeautifulSoup

Playwright **no reemplaza** lo que ya sabes. Trae el HTML final, y `BeautifulSoup` lo parsea igual que en el tema de scraping:

```python
from bs4 import BeautifulSoup
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    navegador = p.chromium.launch()
    pagina = navegador.new_page()
    pagina.goto("https://ejemplo.com")
    pagina.wait_for_selector("li.producto")     # espera a que existan
    html = pagina.content()
    navegador.close()

sopa = BeautifulSoup(html, "html.parser")
for item in sopa.select("li.producto"):
    nombre = item.select_one("span.nombre").get_text(strip=True)
    precio = item.select_one("span.precio")
    print(nombre, precio.get_text(strip=True) if precio else "sin precio")
```

**Ese es el patrón que vas a usar.** Playwright para conseguir el HTML armado, `BeautifulSoup` para recorrerlo. Los selectores CSS que aprendiste siguen siendo los mismos.

### 4. Esperar no es dormir

`wait_for_selector` espera **hasta que el elemento existe**, y sigue apenas aparece. No es un `time.sleep(5)`:

| | `time.sleep(5)` | `wait_for_selector` |
|---|---|---|
| Si carga en 0,3 s | Espera 5 s igual | Sigue en 0,3 s |
| Si carga en 7 s | Falla | Espera y sigue |
| Cuándo sirve | Nunca | Siempre que haya un elemento que marque "ya cargó" |

## ⚠️ El error más común

> [!warning] `Executable doesn't exist at .../chrome-headless-shell`
> ```
> playwright._impl._errors.Error: BrowserType.launch: Executable doesn't exist at
> /home/usuario/.cache/ms-playwright/chromium_headless_shell-1255/chrome-headless-shell-linux64/chrome-headless-shell
> ╔════════════════════════════════════════════════════════════╗
> ║ Looks like Playwright was just installed or updated.       ║
> ║ Please run the following command to download new browsers: ║
> ║                                                            ║
> ║     playwright install                                     ║
> ╚════════════════════════════════════════════════════════════╝
> ```
> **Qué significa:** instalaste la librería pero no el navegador. Son dos cosas distintas. **Cómo se lee:** el propio mensaje dice el comando. La ruta que muestra es dónde lo buscó y no lo encontró. **Cómo se arregla:** `playwright install chromium`. Si ya lo habías instalado y actualizaste Playwright, corré el mismo comando otra vez: la versión nueva pide un navegador nuevo.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min

1. Instala Playwright y el navegador. Comprueba que abre y cierra sin error.
2. Abre una página pública **que renderice con JavaScript** (por ejemplo, un informe de datos que se arma al cargar). Guarda el HTML con `pagina.content()` y cuéntalo en KB.
3. Haz lo mismo con `httpx.get()` sobre la misma URL. Compara los dos tamaños. **Esa diferencia es la medición que justifica Playwright.**

### Reto — opcional

1. Extrae los 20 productos de un sitio que renderiza con JS y genera el **mismo** CSV del tema de scraping (`producto, precio, stock`). Si los nombres de columna coinciden, el resto del pipeline sigue funcionando sin cambios.
2. Mide cuánto tarda cada método sobre la misma página y anótalo. Con ese número puedes explicar en una frase cuándo vale la pena cada uno.

## ✅ Criterios de logro

- [ ] Distingues los dos pasos de instalación y sabes qué falla si falta el segundo
- [ ] Puedes explicar en una frase por qué `httpx` devuelve una página vacía y Playwright no
- [ ] Tu script cierra el navegador aunque falle en el medio (el `with` lo hace)
- [ ] Reutilizas los selectores CSS del tema de scraping sobre el HTML de Playwright
- [ ] Sabes decir cuándo **no** usar Playwright

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `Executable doesn't exist` | Falta el navegador | `playwright install chromium` |
| `Timeout ... waiting for selector` | El selector no existe o el sitio cambió | Abre la página, inspecciona y verifica el selector; prueba `.first` |
| El navegador no abre en Linux | Faltan dependencias del sistema | `playwright install-deps chromium` |
| La descarga del navegador se corta | Proxy o conexión lenta | `PLAYWRIGHT_DOWNLOAD_CONNECTION_TIMEOUT=120000 playwright install` |
| El sitio devuelve 403 solo con Playwright | Bloqueo de automatización | Cambia el `user_agent` o revisa los términos del sitio: puede que no se deba |
| Todo funciona pero tarda mucho | Es lo esperado | Playwright es el último recurso, no el primero |

## 🔗 Para profundizar

- [Playwright para Python — documentación](https://playwright.dev/python/docs/intro) — instalación y primeros pasos
- [Guía de browsers](https://playwright.dev/python/docs/browsers) — qué descarga cada comando y cómo manejar proxys
- [Locators](https://playwright.dev/python/docs/locators) — la forma actual de buscar elementos
- Repaso: [[02 · Scraping responsable con BeautifulSoup]] · [[01 · HTTP con httpx]]

## ⚖️ Antes de usarlo, lo mismo de siempre

Un navegador **no cambia las reglas**. Siguen valiendo el `robots.txt`, los términos del sitio y la protección de datos personales. Que el sitio no detecte que eres un bot no lo hace legítimo. Automatizar el acceso no es un permiso: es solo una forma de acceso.

## ➡️ Qué sigue

Con esto cierras la escalera completa del scraping: **JSON primero, HTML después, navegador al final**. Lo que sigue en el curso es cómo guardar y consultar los datos que ya trajiste.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega el tamaño en KB del HTML con `httpx` y con Playwright, y las dos capturas o los dos conteos de productos. Ese contraste es lo que conviene contrastar con tus propios números.
