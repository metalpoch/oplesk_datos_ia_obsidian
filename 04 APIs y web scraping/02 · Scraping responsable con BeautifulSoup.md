---
tipo: tema
modulo: "04 APIs y web scraping"
capa: familiaridad
herramienta: BeautifulSoup
resultado: "precios_competencia.csv con 20 productos y una nota de 5 líneas sobre legalidad"
tiempo-lectura: 15 min
prerrequisitos: ["HTTP con httpx", "Listas y diccionarios"]
tags: [tema, scraping, beautifulsoup, html, legalidad]
---

# Scraping responsable: leer el HTML de una página con BeautifulSoup

![[python.svg|64]]

> [!tip] En una frase, antes de empezar
> **Hacer scraping es pedirle a una página web su HTML y sacarle los datos con un programa, en vez de copiarlos a mano.** `BeautifulSoup` es la herramienta que recorre ese HTML y encuentra los pedazos que te interesan. No ejecuta JavaScript: lee el HTML que ya llegó. Si el dato lo pinta el navegador después, no está ahí, y hay que bajar a otro peldaño.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea.
>
> **Lo que NO está acá:** la solución del ejercicio. Se construye en la práctica, paso a paso.

## 🤔 Para qué sirve

La competencia tiene una página con sus 20 productos y sus precios. Los quieres. Y ahí empieza el trabajo aburrido:

- Abres la página y copias 20 nombres y 20 precios a una hoja. Te toma 20 minutos. La próxima vez, otros 20.
- Un producto dice "agotado" en vez de precio. La hoja queda con un hueco y **no sabes si el producto desapareció o se agotó**.
- La página cambia el orden de los productos y tu copia queda corrida: el precio de la harina termina en la fila del aceite.

**Eso es lo que resuelve el scraping.** La página que ves es un archivo de texto, el HTML, con la información adentro. Un programa lo lee, baja por sus ramas hasta el precio y arma una fila por producto. `BeautifulSoup` hace esa búsqueda y te deja pedir *"el texto de cada `span.precio`"* en vez de copiar y pegar.

> [!note] Qué no es
> - **No es "bajar la página".** El HTML es el estuche: tener el archivo no es tener los datos, y el trabajo es recorrer el árbol y dejar una fila por producto.
> - **No funciona con lo que pinta JavaScript.** `BeautifulSoup` lee el HTML que ya llegó; si el dato aparece después, en el archivo no está. Ahí se busca el JSON que consume la página y, si no hay, hace falta un navegador.
> - **No exime de `robots.txt` ni de los términos del sitio.** El primero dice qué se puede pedir; los segundos son un contrato. Revisarlo es el primer paso y no necesita código.
> - **No convierte datos personales en un dataset.** Nombres, cédulas o teléfonos son personas: en Venezuela los protege la Ley Orgánica de Protección de Datos Personales (2021).

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Extraer nombre, precio y stock de los 20 productos de una **copia local** del HTML de la competencia y dejarlos en `precios_competencia.csv` |
| **Después** | Reportar cuántos de esos 20 no tenían precio y por qué, sin que el script se caiga, y escribir 5 líneas sobre por qué hacerlo así es legítimo |

Aquí se trabaja **contra una copia local del HTML**, no contra el sitio en vivo: no es una limitación técnica, es la decisión correcta. Scrapear un sitio real queda como reto por tu cuenta.

## 🧠 Conceptos

1. **El HTML es un árbol y el selector es el camino** — no buscas texto, bajas por ramas: `li.producto` → `span.precio`.
2. **`select_one` devuelve uno o `None`; `select` devuelve una lista** — el dato que falta no es un error, es parte del dato.
3. **Primero se decide de dónde salen los datos, después cómo** — hay una escalera de tres peldaños (JSON, HTML, navegador) y se empieza por arriba; además, `robots.txt`, términos del sitio y datos personales.

## 📖 Teoría
### 1. El HTML es un árbol y el selector CSS es el camino
```html
<li class="producto">
  <span class="nombre">Harina P.A.N. 1kg</span>
  <span class="precio">1,20</span>
</li>
<li class="producto agotado">
  <span class="nombre">Aceite Vatel 1L</span>
  <span class="stock">agotado</span>
</li>
```

Cada etiqueta es una rama, y la `class` es la etiqueta que el sitio le puso a esa rama. Un selector CSS describe el camino: `li.producto` es "cada `li` con la clase `producto`".

```python
from bs4 import BeautifulSoup
html = open("competencia.html", encoding="utf-8").read()
sopa = BeautifulSoup(html, "html.parser")
print(len(sopa.select("li.producto")))      # 20
print(sopa.select_one("span.precio").text)  # 1,20
```

**Por qué importa:** buscar con `split` o regex "el texto que está después de `>`" se rompe cuando el sitio agrega un espacio. El selector por clase sobrevive a los cambios de diseño, porque apunta a lo que significa, no a dónde está.

### 2. `select_one` vs `select`, y qué hacer con los faltantes
`select` devuelve **siempre una lista**: si no encontró nada, `[]`. `select_one` devuelve el primero o `None`. Y ese `None` es el que rompe todo si lo tratas como si fuera un dato.

```python
for item in sopa.select("li.producto"):
    nombre = item.select_one("span.nombre").text
    nodo = item.select_one("span.precio")
    precio = None if nodo is None else nodo.text
    print(nombre, precio)      # ... Aceite Vatel 1L None
```

"Agotado" no es una falla: es información que el negocio necesita. Un script que revienta en el primer agotado no sirve.

### 3. Las tres capas: JSON, HTML o navegador

`BeautifulSoup` **no ejecuta JavaScript**. No es una limitación nueva ni un defecto: nunca fue un navegador. Recorre el HTML que ya llegó. Por eso, antes de escribir una línea de scraping hay una escalera de tres peldaños, y se baja desde arriba:

| # | Qué buscar | Herramienta | Cuándo |
|---|---|---|---|
| 1 | Una **API o un JSON** | `httpx` + `json` | Siempre que exista. Lo más estable y lo más limpio |
| 2 | **HTML completo** desde el servidor | `httpx` + `BeautifulSoup` | Sitios estáticos o que renderizan del lado del servidor |
| 3 | **HTML armado con JavaScript** | Playwright (navegador real) | Último recurso: lento, pesado y frágil |

**El peldaño 1 es el que casi nadie usa y el que mejor funciona.** Muchos sitios hechos con React o Next.js **sí traen los datos dentro del HTML**, en un `<script type="application/json">`. No hace falta un navegador: se lee ese JSON.

```python
import json
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, "html.parser")
bloque = soup.find("script", id="__NEXT_DATA__")     # el que usa Next.js
if bloque:
    datos = json.loads(bloque.string)                 # los datos, ya estructurados
```

Ese bloque puede contener el catálogo entero. Antes de escribir doscientos selectores, conviene buscar `__NEXT_DATA__`, `window.__INITIAL_STATE__` o `/api/` en el código fuente de la página: son diez segundos mejor invertidos.

> [!warning] Qué significa para este tema
> En el curso se trabaja con una **copia local del HTML**, así que `BeautifulSoup` alcanza: es el peldaño 2. Lo que hay que llevarse es **la escalera**, no la librería. Si el sitio del trabajo resulta ser una aplicación React que no trae los datos en el HTML, la respuesta correcta no es «más BeautifulSoup»: es buscar el JSON y, si no hay, usar un navegador.
>
> Playwright no se instala como parte de este tema, porque la copia local no lo necesita. Está en el anexo, para quien quiera el peldaño 3: es un `pip install` más un navegador (`playwright install chromium`), unos 300 MB en total y se hace una sola vez.

## ⚠️ El error más común

> [!warning] `AttributeError: 'NoneType' object has no attribute 'text'`
> ```
> Traceback (most recent call last):
>   File "scrapear.py", line 8, in <module>
>     precio = item.select_one("span.precio").text
>              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
> AttributeError: 'NoneType' object has no attribute 'text'
> ```
> **Qué significa:** `select_one` no encontró ningún `span.precio` y devolvió `None`. Después pediste `.text` sobre `None`, y `None` no tiene texto. **Cómo se lee:** el `None` es la respuesta honesta del HTML: ese producto **no tiene precio**. Mira el `li` completo del producto que falló (te lo dice la línea) y vas a ver que dice `agotado`. **Cómo se arregla:** separa la búsqueda del uso. Guarda el nodo en una variable, pregunta `if nodo is None`, anota el nombre del producto y sigue con el siguiente. En el CSV, una fila con la celda de precio vacía: perder el producto entero porque no tiene precio es peor que entregar 20 filas con un hueco declarado.

Este no es un error de adorno: es el que más se repite si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min
Los haces **en tu terminal o en VS Code**, sobre tu copia local. Al terminar, pega la salida en "Mi resultado".

1. **(10 min)** Sobre `competencia.html` (20 productos, en los materiales del tema; si no lo tienes, repite el bloque de la teoría con nombres y precios distintos hasta llegar a 20), genera `precios_competencia.csv` con columnas `nombre,precio,stock` y **20 filas**.
2. **(5 min)** Reporta tres cosas: cuántos productos encontró tu selector, cuántos quedaron sin precio y por qué. Suma la **nota de 5 líneas** sobre legalidad: por qué esta copia local es legítima y qué cambiaría contra el sitio en vivo.
3. **(10 min)** Rompe a propósito: saca el `if nodo is None` y corre. Lee el traceback completo. ¿En qué producto se cayó?

### Reto — opcional, para quien quiera más
1. Agrega una columna `disponible` (verdadero/falso) mirando si el `li` tiene la clase `agotado`, sin usar el texto del stock.
2. Escribe el mismo extractor dos veces: una con `select` y otra con `select_one`, y explica en un comentario cuándo conviene cada una.
3. **El peldaño 3:** elige un sitio que en el navegador muestre los datos y con `httpx` devuelva una página casi vacía. Saca los mismos 20 productos con Playwright y compara el tamaño del HTML en los dos casos. Está todo en [[Playwright, el tercer peldaño del scraping]].

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `wc -l precios_competencia.csv` devuelve 21 (20 productos + encabezado)
- [ ] El script corre de punta a punta **sin traceback**, aunque haya productos sin precio
- [ ] Tu reporte dice cuántos quedaron sin precio y el motivo de cada uno
- [ ] Explicas en una frase por qué trabajaste sobre una copia y no contra el sitio
- [ ] Puedes explicar la escalera: cuándo se usa el JSON, cuándo el HTML y cuándo hace falta un navegador

## 🔗 Para profundizar

- [BeautifulSoup — documentación en español](https://www.crummy.com/software/BeautifulSoup/bs4/doc.es/#css-selectors) — selectores CSS, que son todo lo que usas aquí; y [MDN: selectores CSS](https://developer.mozilla.org/es/docs/Web/CSS/CSS_selectors) para afinarlos
- [Robots Exclusion Protocol — RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html) — el estándar de `robots.txt`, en 20 minutos de lectura
- Anexo del vault: [[Playwright, el tercer peldaño del scraping]] — el peldaño 3 con instalación, código y su error típico
- Anexo del vault: [[01 · HTTP con httpx]]

## 🧭 Problemas frecuentes

- **`select("li.producto")` devuelve `[]` o salta `UnicodeDecodeError`** → el archivo no se leyó bien: falta `encoding="utf-8"` (los acentos y la ñ rompen el default del sistema) o no es el archivo que crees. Imprime los primeros 200 caracteres de `html`.
- **`ModuleNotFoundError: No module named 'bs4'`** → el paquete se instala con `pip install beautifulsoup4`, aunque se importe como `bs4`.
- **`urllib.robotparser` dice `False` para todo** → el `robots.txt` no se leyó (403 o 404) y asume lo peor. Y `parse` espera las líneas separadas, no un string.

Si nada de esto lo resuelve, pega el `li` completo del producto que falla en "Mi resultado". Con eso arrancamos.

## ➡️ Siguiente tema

Ya tienes datos que nadie te dio: en el siguiente tema aparece el lugar donde se juntan y se pueden preguntar cosas, y la primera decisión de tipos que te va a costar cara si la tomas tarde.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá las 5 primeras filas de tu CSV, cuántos productos extrajiste y **cuántos quedaron sin precio**. Ese último número es el ejercicio: si da 0, revisaste el HTML con más cuidado que el script. Si no te coincide, pega el error.