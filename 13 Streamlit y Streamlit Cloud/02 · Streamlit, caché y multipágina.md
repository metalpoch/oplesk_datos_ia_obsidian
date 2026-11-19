---
tipo: tema
modulo: "13 Streamlit y Streamlit Cloud"
capa: dominio inicial
herramienta: Streamlit
resultado: "app con st.cache_data y bench de dos tiempos medidos en pantalla"
tiempo-lectura: 15 min
prerrequisitos: ["Streamlit, primera app", "DuckDB: conexiones y consultas"]
tags: [streamlit, cache, multipagina]
---

# Streamlit: caché, estado y multipágina

![[streamlit.svg|64]]

> [!tip] En una frase, antes de empezar
> **Cachear es guardar el resultado de un cálculo para no repetirlo: la primera vez la app consulta y guarda; las siguientes lee del cajón y responde al instante.**

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y, cuando algo no te cierre, escribe `#duda` en esa línea: esas dudas marcan por dónde empezar. **Lo que NO está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

Mueves el filtro y la app se queda pensando. Mueves otro y otra vez. Cada clic vuelve a leer el Parquet y a consultar la base, porque Streamlit corre el script entero de nuevo: lo que en tu máquina se siente rápido, del otro lado es una página que parece congelada y un dedo que vuelve a tocar el filtro para ver si pasó algo.

Y cuando por fin divides la app en páginas, aparece el segundo problema: cambias de pestaña y el filtro de fechas se perdió, aunque el usuario no lo tocó.

**Eso es lo que resuelven estas dos piezas:** el caché guarda el resultado para no recalcularlo, y `session_state` guarda lo que el usuario eligió para que sobreviva a la re-ejecución. Con las dos, la segunda interacción se siente instantánea.

> [!tip] Dos piezas distintas
> El caché guarda **resultados** para no recalcular; `session_state` guarda **valores del usuario** para que no se pierdan. Y cachear no arregla una consulta lenta: la primera vez sigue tardando lo mismo. Solo evita pagar ese tiempo de nuevo.

> [!note] Qué no es
> - **La caché no es persistencia.** Vive en la memoria del proceso: si la app se reinicia, se vacía y vuelve a consultar. No es una base, no la comparten dos usuarios y no sobrevive a un despliegue.
> - **`cache_data` no guarda conexiones.** Serializa el resultado y devuelve una copia; una conexión no se serializa, va en `cache_resource` y se comparte tal cual.
> - **Una app multipágina no es una app con login.** `pages/` reparte scripts y comparte `session_state`; no hay usuarios, ni permisos, ni datos separados por quién entra.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Hacer que la segunda ejecución sea notoriamente más rápida, con los dos tiempos medidos y visibles en pantalla |
| **Después** | Dividir la app en tres páginas que comparten el mismo filtro de fechas |

## 🧠 Conceptos

Solo tres. Si te llevas estos tres, el tema está cubierto.

1. **`cache_data` guarda datos; `cache_resource` guarda objetos** — un DataFrame o un número se serializan y se copian; una conexión no: se comparte tal cual.
2. **`session_state` es el cajón que sobrevive** — las variables normales se borran en cada re-ejecución; lo que guardas ahí dura mientras la pestaña del navegador siga abierta.
3. **Cada página es un script aparte** — `pages/` no comparte código con `app.py`: lo que comparten es el `session_state`.

## 📖 Teoría

### 1. Dos cachés, dos trabajos
```python
import duckdb
import streamlit as st
@st.cache_resource
def conectar():
    return duckdb.connect("tienda.duckdb", read_only=True)     # una sola conexión

@st.cache_data(ttl="1h", show_spinner="Consultando…")
def ingresos(desde: str, hasta: str) -> float:
    con = conectar()
    return con.execute("SELECT sum(ingreso) FROM ventas WHERE fecha BETWEEN ? AND ?",
                       [desde, hasta]).fetchone()[0]
```

**`cache_data`** guarda el **resultado** (lo serializa y te devuelve una copia: DataFrames, números). **`cache_resource`** guarda el **objeto mismo** (conexiones, clientes, modelos).

**Por qué la conexión no va en `cache_data`:** no se puede serializar y, aunque pudieras, una copia no es la misma conexión. Va en `cache_resource`, siempre. El `ttl` tampoco es decorativo: si el pipeline corre una vez al día, un `ttl` de cinco minutos no te da datos más frescos, solo más consultas.

### 2. `session_state`: lo que sobrevive entre corridas
```python
st.sidebar.date_input("Desde", value="2025-01-01", key="desde")
st.write(st.session_state.desde)      # el widget ya guardó su valor acá
```

En cada re-ejecución las variables se recrean, pero `st.session_state` (un diccionario que vive mientras dure la pestaña) no. Con `key="desde"`, el widget guarda solo; sin `key`, el valor te vuelve como retorno y lo tienes que reasignar a mano.

### 3. Multipágina: tres scripts y un estado
```text
app.py            # página 1: Resumen
pages/2_Calidad.py
pages/3_Alertas.py
utils.py          # el filtro, en un solo lugar
```

Cada archivo de `pages/` es un script independiente: **`app.py` no se ejecuta** cuando estás en la página 2. Por eso el widget no puede vivir solo en `app.py`; va en `utils.py` y cada página lo llama al principio. Ten en cuenta: navegar con un link de markdown o escribiendo la URL crea una ejecución nueva de la app y reinicia `session_state`; navega con el menú.

```python
# utils.py
def filtro_fecha():
    st.sidebar.date_input("Desde", value="2025-01-01", key="desde")
    return st.session_state.desde
```

## ⚠️ El error más común

> [!warning] `UnhashableParamError: Cannot hash argument 'con' in 'cargar_datos'`
> ```
> UnhashableParamError: Cannot hash argument 'con' (of type `duckdb.DuckDBPyConnection`) in 'cargar_datos'.
>
> To address this, you can tell Streamlit not to hash this argument by adding a
> leading underscore to the argument's name in the function signature:
> ```
> **Qué significa:** le pediste a `cache_data` que cachee una función que recibe la conexión. Para armar la clave del caché, Streamlit tiene que *hashear* los argumentos, y una conexión no se puede hashear. **Cómo se lee:** el mensaje nombra el argumento (`con`) y su tipo. Con eso alcanza: la conexión está entrando a `cache_data` cuando debería estar en `cache_resource`. **Cómo se arregla:** la conexión va en una función con `@st.cache_resource`; la consulta va en una con `@st.cache_data` que la usa por dentro, no como argumento. El atajo que el propio mensaje sugiere (renombrar el argumento a `_con`) silencia el error, pero cada ejecución abre una conexión nueva: el problema sigue ahí, escondido.

Este no es un error de adorno: es el que se repite en el proyecto si no lo has visto antes.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal / VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Mide con `time.perf_counter()` la carga y muestra los dos tiempos con dos `st.metric`, uno al lado del otro.
2. Cachea la consulta con `@st.cache_data` y comprueba que el segundo tiempo baja de forma notoria.
3. Rompe a propósito: mete la conexión dentro de la función con `@st.cache_data`, pega el error completo y arréglalo con `cache_resource`.

### Reto — opcional, para quien quiera más

1. Elige un `ttl` y justifícalo con la frecuencia con la que corre el pipeline; escribe la razón al lado del decorador.
2. Arma las tres páginas y agrega un botón que resetee el filtro (`st.session_state.clear()`), explicando qué pasa después.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Los dos tiempos están en pantalla y el segundo es notoriamente menor
- [ ] La conexión vive en `cache_resource` y la consulta en `cache_data`
- [ ] Las tres páginas comparten el mismo filtro de fechas
- [ ] El `ttl` tiene una razón escrita y defendible
- [ ] Puedes explicar qué se cachea, qué no, y por qué

## 🔗 Para profundizar

- [Caché en Streamlit](https://docs.streamlit.io/develop/concepts/architecture/caching) — `cache_data` vs `cache_resource` · [`st.cache_data`](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.cache_data) · [`st.cache_resource`](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.cache_resource)
- [`st.session_state`](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.session_state) — el diccionario que sobrevive
- [Apps multipágina](https://docs.streamlit.io/develop/concepts/multipage-apps/overview) — `pages/` es la vía legacy · En el vault: [[01 · Streamlit, primera app]] · [[03 · Streamlit, publicar]]

## 🧭 Problemas frecuentes

- **La segunda vez sigue lento** → el argumento cambia en cada corrida (una lista nueva) o el decorador quedó en la función equivocada.
- **`UnhashableParamError` con otro objeto** → eso que pasas no se puede hashear: sepáralo en `cache_resource`.
- **El filtro se pierde al cambiar de página** → no está en `session_state`, o estás navegando por URL en vez del menú.
- **Ves datos viejos** → `ttl` largo o caché de una prueba anterior: usa `st.cache_data.clear()`.

## ➡️ Siguiente tema

La app sale de tu máquina: la publicas con una URL que abre cualquiera, y aprendes qué NO prometerle a nadie sobre ella.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá los dos tiempos que mediste, la captura de las tres páginas con el mismo filtro y el texto del error que no pudiste resolver. El siguiente tema retoma lo que quede acá.
