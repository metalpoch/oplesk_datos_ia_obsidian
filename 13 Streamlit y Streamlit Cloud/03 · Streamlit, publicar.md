---
tipo: tema
modulo: "13 Streamlit y Streamlit Cloud"
capa: dominio inicial
herramienta: "Streamlit Community Cloud"
resultado: "URL pública que abre en incógnito, con los secrets fuera del repo"
tiempo-lectura: 15 min
prerrequisitos: ["Streamlit, caché y multipágina", "Git y GitHub"]
tags: [streamlit, deploy, secrets]
---

# Publicar la app: URL, secrets y límites reales

![[streamlit.svg|64]]

> [!tip] En una frase, antes de empezar
> **Publicar —hacer el `deploy`— es poner tu app en un servidor para que cualquiera la abra con un link, sin instalar Python ni correr nada en su computadora.**

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y, cuando algo no te cierre, escribe `#duda` en esa línea: esas dudas marcan por dónde empezar. **Lo que NO está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

Le mandas la carpeta a tu jefe con un README que dice "instala Python, después Streamlit, después corre este comando". No lo va a hacer. Y tiene razón: no es su trabajo.

Tu app funciona mientras tu laptop está prendida y tu terminal abierta. Si cierras la tapa, deja de existir. El día de la reunión depende de que no se caiga la conexión, y si te preguntan "¿me pasas el link?", no hay link: hay una carpeta.

**Eso es lo que resuelve publicar:** un servidor clona tu repo, crea un entorno limpio, instala lo que dice `requirements.txt` y levanta la app. El resultado es una URL que abre desde cualquier teléfono.

> [!note] Qué no es
> El `push` ya lo hiciste y no alcanza: el código en GitHub no se ejecuta solo. Publicar es que el servicio lo tome y lo corra. Y no es "online para siempre": en el plan gratuito la app es pública y se duerme sin tráfico. Eso no se promete.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Publicar la app en una URL pública que arranca y lee la fuente de datos real |
| **Después** | Abrir esa URL en una ventana de incógnito sin credenciales en el repo, y medir el arranque en frío |

La app funciona en tu máquina. Publicada, deja de depender de ella, y eso trae tres problemas nuevos: las dependencias, las credenciales y los límites del servicio.

## 🧠 Conceptos

Solo tres. Si te llevas estos tres, el tema está cubierto.

1. **La app se construye desde el repo** — la nube clona tu código, crea un entorno limpio e instala lo que diga `requirements.txt`. Tu `venv` no viaja.
2. **La credencial no va al repo** — el repo es público; lo que se commitea queda en el historial. Va en `st.secrets`.
3. **El plan gratuito tiene límites reales** — app pública, se duerme a las 12 horas sin tráfico, corre en EE. UU. y los límites pueden cambiar sin aviso.

## 📖 Teoría

### 1. Se construye desde el repo, no desde tu máquina
```text
# requirements.txt — lo que probaste en local, con la versión exacta
streamlit==1.66.0
duckdb==1.5.6
pandas==2.2.3
plotly==5.24.1
```

Streamlit Community Cloud no ve tu `venv`: clona el repo, crea un entorno limpio e instala lo que dice `requirements.txt`. Si falta una línea, la app no arranca; si no fijas versiones, cualquier día se instala otra cosa y tu app cambia sola. Los números de arriba son un ejemplo: los tuyos salen de `pip freeze`, y lo que no se negocia es que haya versiones.

Y en `.gitignore`, antes del primer push: `.streamlit/secrets.toml`, `.env`, `data/`, `*.duckdb`.

### 2. `st.secrets`: la credencial no se publica
```python
import streamlit as st

url = st.secrets["DATABASE_URL"]      # en local sale de .streamlit/secrets.toml
```

En tu máquina la clave vive en `.streamlit/secrets.toml`, que **no se versiona**. En la nube se pega en **App settings → Secrets**, con el mismo formato TOML. La app nunca lee la credencial del código.

**Por qué importa:** el repo es público y cualquiera lee tu código. Una contraseña commiteada no se borra con otro commit: queda en el historial para siempre, y rotarla es la única salida limpia.

### 3. Los límites, dichos sin marketing

- **La app es pública.** Hereda la visibilidad del repo: repo público, app pública. Cualquiera con el link entra, y solo se admite **una app privada** por cuenta.
- **Se duerme a las 12 horas sin tráfico.** Cualquiera puede despertarla; mientras está dormida, la primera visita tarda más.
- **Corre en EE. UU.** La documentación lo dice explícito: no es configurable. Si tus datos no pueden salir del país, esto no es la herramienta.
- **Los límites pueden cambiar sin aviso.** CPU, memoria y disco están publicados, pero el proveedor puede ajustarlos cuando quiera.

Por eso **"online para siempre" es falso** y no se promete. Lo que sí se promete: una URL que abre, un dato real y un tiempo de arranque en frío medido.

## ⚠️ El error más común

> [!warning] `ModuleNotFoundError: No module named 'pandas'`
> ```
> ModuleNotFoundError: No module named 'pandas'
> ```
> **Qué significa:** el entorno de la nube se creó y no encontró la librería: no está en `requirements.txt`, o el archivo no está en la raíz del repo. **Cómo se lee:** el log del deploy se lee **de abajo hacia arriba**: el primer error real es el que importa, el resto suele ser consecuencia. Y mira la ruta del archivo: si quedó dentro de una subcarpeta, la nube no lo toma. **Cómo se arregla:** agrega la línea, commitea y espera el redeploy. Si lo que falta no es una librería de Python sino un paquete del sistema, va en `packages.txt`, uno por línea.

Este no es un error de adorno: es el que se repite en el proyecto si no lo has visto antes.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min

Los haces **en tu terminal / VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Genera `requirements.txt` con `pip freeze`, deja solo lo que la app usa y prueba que corra en un `venv` limpio.
2. Publica la app desde el repo y guarda la URL. Comprueba que lee la fuente de datos real, no un archivo de ejemplo.
3. Rompe a propósito: borra una línea de `requirements.txt`, sube, lee el log del deploy y vuelve a arreglarlo.

### Reto — opcional, para quien quiera más

1. Mide el arranque en frío: deja la app dormir (o usa Reboot) y cronometra la primera visita. Anota el número.
2. Pasa la credencial a `st.secrets` y comprueba con `git log -p` que nunca estuvo en el repo.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] La URL abre en una ventana de incógnito, sin login
- [ ] `requirements.txt` está en la raíz del repo y con versiones
- [ ] `git log -p | grep -iE "password|token|postgres://|api_key"` no devuelve nada
- [ ] Mide el arranque en frío y sé cuánto tarda
- [ ] Puedes explicar por qué la app se duerme y qué NO le prometes a nadie

## 🔗 Para profundizar

- [Deploy en Community Cloud](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app) — el paso a paso oficial
- [Dependencias de la app](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/app-dependencies) · [Secrets en la nube](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/secrets-management)
- [Estado y limitaciones](https://docs.streamlit.io/deploy/streamlit-community-cloud/status) — la página que hay que releer antes de prometer algo
- En el vault: [[02 · Streamlit, caché y multipágina]]

## 🧭 Problemas frecuentes

- **El deploy falla** → lee el log de abajo hacia arriba; el primer error es el que importa. **Queda como ejercicio y no bloquea el avance**: anota el log completo y vuelve a intentarlo cuando corrijas la dependencia.
- **La app dice "Oh no" o "gone over its resource limits"** → demasiadas filas en memoria: filtra en SQL, no en Pandas.
- **Se durmió y no despierta** → entra y toca "Yes, get this app back up!", o Reboot desde Manage app.
- **Abre pero sin datos** → falta el secret, o la ruta relativa cambió: la nube corre desde la raíz del repo.

## ➡️ Siguiente tema

La app ya es pública y lee un archivo local; en el siguiente tema los datos se mudan a una base de verdad: Postgres en la nube, sin servidor que administrar.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la URL, el tiempo de arranque en frío que mediste y el texto del error de deploy que no pudiste resolver. El siguiente tema retoma lo que quede acá.
