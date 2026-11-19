---
tipo: tema
modulo: "13 Streamlit y Streamlit Cloud"
capa: dominio inicial
herramienta: Streamlit
resultado: "app.py de ~40 líneas con 3 métricas y un filtro, corriendo en local"
tiempo-lectura: 15 min
prerrequisitos: ["Python: funciones y módulos", "DuckDB: consultas con parámetros"]
tags: [streamlit, app]
---

# Streamlit: tu primera app de datos

![[streamlit.svg|64]]

> [!tip] En una frase, antes de empezar
> **Streamlit es una librería de Python que convierte tu script en una página web con filtros, métricas y gráficos, sin que escribas HTML, CSS ni JavaScript.**

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y, cuando algo no te cierre, escribe `#duda` en esa línea: esas dudas marcan por dónde empezar. **Lo que NO está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

Tienes el script que calcula las tres cifras. Cada vez que alguien quiere verlas te escribe: "¿me pasas el total de Bebidas de marzo?". Le mandas una captura. Al día siguiente, otra vez, con otro filtro.

Tu jefe no va a abrir VS Code ni a correr `python informe.py` en una terminal. Si el análisis no está detrás de un link que abra en el teléfono y tenga un desplegable, para él no existe: existe el Excel que le mandaste el mes pasado.

**Eso es lo que resuelve Streamlit:** el mismo script, con widgets arriba y las métricas abajo, en una página que se abre en el navegador. El filtro lo mueve quien mira, no tú.

> [!note] Qué no es
> No escribes HTML ni JavaScript, y tampoco es una herramienta de arrastrar y soltar. Y no es "publicar": esto corre en tu máquina, en `localhost`, mientras la terminal está abierta. Sacarlo de ahí es el tema [[03 · Streamlit, publicar]].

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Levantar en local una app de ~40 líneas con tres métricas y un filtro, con `streamlit run app.py` |
| **Después** | Hacer que un gráfico se mueva con el filtro y comprobar dos totales distintos contra la consulta SQL |

## 🧠 Conceptos

Solo tres. Si te llevas estos tres, el tema está cubierto.

1. **Streamlit re-ejecuta todo el script** — de arriba a abajo, en cada interacción. Ese único hecho explica casi todos sus errores.
2. **Los widgets devuelven un valor** — `categoria = st.selectbox(...)` guarda en la variable lo que el usuario eligió. No hay callbacks.
3. **El orden de las líneas importa** — primero el filtro, después la consulta. Al revés, o falla o muestra datos viejos.

## 📖 Teoría

### 1. El script entero corre otra vez
```python
import streamlit as st

st.title("Ventas de la tienda")
st.write("Esta línea se ejecuta cada vez que tocas un widget")
```

No hay un servidor que recuerde tus variables: Streamlit vuelve a correr `app.py` desde la primera línea, con el valor nuevo del widget, y redibuja la página. **Por qué importa:** todo lo que pongas ahí se repite en cada clic, incluida la lectura de 50.000 filas. Por eso el siguiente tema se dedica al caché. Y por eso una variable normal pierde su valor entre corridas: lo que tiene que sobrevivir se guarda en `st.session_state`.

### 2. Los widgets devuelven un valor
```python
categoria = st.selectbox("Categoría", ["Todas", "bebidas", "panaderia", "abarrotes"])
minimo = st.slider("Ticket mínimo", 0, 200, 20)
```

`st.selectbox` **devuelve** la opción elegida y la guardas en una variable. Nada más: sin `on_change`, sin listener. El slider devuelve un número. Esos dos valores son tu filtro.

### 3. El orden: el filtro va antes de la consulta
```python
import duckdb
import streamlit as st
import plotly.express as px
con = duckdb.connect()
categoria = st.selectbox("Categoría", ["Todas", "bebidas", "panaderia", "abarrotes"])
df = con.execute("""
    SELECT * FROM 'ventas.parquet'
    WHERE ($cat = 'Todas' OR categoria = $cat)
""", {"cat": categoria}).df()
col1, col2, col3 = st.columns(3)
col1.metric("Ingresos", f"${df['ingreso'].sum():,.0f}")
col2.metric("Tickets", f"{len(df):,}")
col3.metric("Ticket promedio", f"${df['ingreso'].mean():,.2f}")
fig = px.line(df.sort_values("fecha"), x="fecha", y="ingreso", title="Ingresos del período filtrado")
st.plotly_chart(fig)
```

Dos cosas hacen que esto funcione:

- `categoria` se lee **antes** de la consulta, porque la consulta la usa.
- La consulta va **parametrizada** (`$cat` con un diccionario) y las métricas se calculan sobre `df`, que ya viene filtrado.

Se corre desde la carpeta del proyecto: `streamlit run app.py` (abre en `localhost:8501` y se recarga al guardar).

## ⚠️ El error más común

> [!warning] `NameError: name 'categoria' is not defined`
> ```
> NameError: name 'categoria' is not defined
> ```
> **Qué significa:** el script llegó a una línea que usa `categoria` antes de que exista. En Streamlit casi siempre es lo mismo: la consulta está arriba del widget. **Cómo se lee:** mira la línea que Streamlit marca y sube hasta dónde se define `categoria`. Si aparece después, ese es el problema. Atención a la variante sin error: si la consulta no usa la variable, la app corre y **no filtra nada**; el número no cambia y nadie se entera. **Cómo se arregla:** mueve el widget arriba de la consulta y comprueba el valor antes de usarlo:
> ```python
> categoria = st.selectbox("Categoría", ["Todas", "bebidas"])
> st.write("filtro actual:", categoria)      # si esto no cambia, nada va a cambiar
> ```

Este no es un error de adorno: es el que se repite en el proyecto si no lo has visto antes.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal / VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Arma `app.py` con título y tres métricas leyendo `ventas.parquet`, y levántalo con `streamlit run app.py`.
2. Agrega un `selectbox` de categoría y un `slider`, y comprueba que dos valores distintos dan dos totales distintos.
3. Rompe a propósito: mueve la consulta arriba del widget, pega el error completo y después arréglalo.

### Reto — opcional, para quien quiera más

1. Verifica los dos totales contra la consulta SQL de [[01 · DuckDB y Parquet]] y deja esa consulta anotada al lado de cada métrica.
2. Agrega un `st.button("Actualizar")` y explica en una frase qué hace en un script que ya se re-ejecuta solo.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `streamlit run app.py` abre la app en `localhost:8501`
- [ ] Hay tres métricas y al menos un filtro visible
- [ ] Mover el filtro cambia las métricas, comprobado con dos valores
- [ ] Los dos totales coinciden con la consulta SQL de [[01 · DuckDB y Parquet]]
- [ ] Puedes explicar en una frase por qué la app se vuelve a ejecutar entera

## 🔗 Para profundizar

- [Streamlit — arquitectura y ejecución](https://docs.streamlit.io/develop/concepts/architecture) — el capítulo que explica la re-ejecución
- [Referencia de la API](https://docs.streamlit.io/develop/api-reference) — widgets, métricas y gráficos, con ejemplos
- [Instalación](https://docs.streamlit.io/get-started/installation) — si el comando `streamlit` no aparece · En el vault: [[01 · Visualización con Plotly y Seaborn]] · [[02 · Streamlit, caché y multipágina]]

## 🧭 Problemas frecuentes

- **`streamlit: command not found`** → no está en el `venv` activo. Prueba `python -m streamlit run app.py`.
- **El filtro no cambia nada** → la consulta no usa la variable o está antes del widget. Mira el `st.write` del filtro.
- **`Port 8501 is already in use`** → hay otra app corriendo: `streamlit run app.py --server.port 8502`.

## ➡️ Siguiente tema

La app va a responder rápido: vas a cachear la consulta y medir los dos tiempos, en pantalla.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la captura de la app con el filtro aplicado, los dos totales que comprobaste y el texto del error que no pudiste resolver. El siguiente tema retoma lo que quede acá.

---
![[meme-excel.jpg]]