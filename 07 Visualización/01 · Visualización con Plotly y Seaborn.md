---
tipo: tema
modulo: "07 Visualización"
capa: dominio inicial
herramienta: "Plotly y Seaborn"
resultado: "figuras/ingresos.html y figuras/categorias.png + una frase por gráfico"
tiempo-lectura: 15 min
prerrequisitos: ["Pandas: groupby y merge", "DuckDB y Parquet"]
tags: [plotly, seaborn, visualización]
---

# Visualización: el gráfico se elige por la pregunta

![[plotly.svg|64]]

> [!tip] En una frase, antes de empezar
> **Plotly y Seaborn son dos librerías de Python que dibujan gráficos a partir de un `DataFrame`: Plotly produce un HTML que se puede explorar, Seaborn un PNG que se ve igual en todos lados.**

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y, cuando algo no te cierre, escribe `#duda` en esa línea: esas dudas marcan por dónde empezar. **Lo que NO está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

Entregas un informe de doce páginas con tres tablas de cuarenta números. Nadie lo lee. Entregas un gráfico y te preguntan "¿y esto qué me dice?", porque el título decía `Ingresos mensuales` y el eje de fechas venía en texto: cincuenta etiquetas amontonadas donde había doce periodos.

El dato estaba bien en los dos casos. Lo que faltaba era la última milla: convertir una tabla en algo que otra persona mire en cinco segundos y saque una conclusión.

**Eso es lo que resuelven estas dos librerías:** el mismo `DataFrame` que ya tienes se vuelve una imagen o una página interactiva, con título, ejes y fuente. Y la pregunta decide cuál: barras para comparar, línea para el tiempo, dispersión para la relación.

> [!note] Qué no es
> Un gráfico exportado es una foto que no cambia; un dashboard tiene filtros y se mueve —eso llega en la nota siguiente, con Streamlit. Y no todo necesita gráfico: si la pregunta es "¿cuánto vendimos?", un número con su fuente alcanza y se lee más rápido.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Producir un gráfico interactivo de la serie de ingresos por periodo y exportarlo a `figuras/ingresos.html`, con título que afirma el hallazgo |
| **Después** | Producir un gráfico de distribución por categoría con color por grupo, exportarlo a PNG y escribir en una frase qué se ve |

## 🧠 Conceptos

Solo tres. Si te llevas estos tres, el tema está cubierto.

1. **La pregunta elige el gráfico** — comparación, distribución, evolución o relación. Sin pregunta en una frase, el gráfico es decoración.
2. **Plotly es interactivo; Seaborn es estático** — uno se comparte como HTML para explorar; el otro se guarda como PNG para un informe.
3. **El título afirma el hallazgo** — "Ingresos mensuales" describe columnas; "Los ingresos caen 18 % en enero" dice algo. Y todo gráfico lleva la fuente y las fechas del dato.

## 📖 Teoría

### 1. La pregunta elige el gráfico
| Si la pregunta es… | Usa… | Porque… |
|---|---|---|
| ¿Qué categoría vende más? | barras | se comparan longitudes, no ángulos ni áreas |
| ¿Cómo se reparte el ticket? | histograma o boxplot | muestra la forma, no solo el promedio |
| ¿Cómo evoluciona en el tiempo? | línea | el eje horizontal es continuo |
| ¿Precio y cantidad se mueven juntos? | dispersión | cada punto es un par de valores |

### 2. Plotly interactivo vs Seaborn estático
```python
import pandas as pd
import plotly.express as px
df = pd.read_parquet("ventas.parquet")
df["fecha"] = pd.to_datetime(df["fecha"], errors="coerce")   # sin esto el eje miente (ver abajo)
periodos = df.set_index("fecha").resample("W")["ingreso"].sum().reset_index()  # una fila por periodo
fig = px.line(periodos, x="fecha", y="ingreso", markers=True,
              labels={"fecha": "Periodo", "ingreso": "Ingreso (USD)"},
              title="Los ingresos caen 18 % en enero")  # ejemplo: pon tu cifra
fig.add_annotation(text="Fuente: ventas.parquet · 16/11/2026", xref="paper", yref="paper", x=0, y=-0.2, showarrow=False)
fig.write_html("figuras/ingresos.html")
```

Seaborn dibuja para papel y corre **sobre Matplotlib**: de ahí sale `plt`. No lo enseñamos como tema; lo usamos para lienzo, título y guardado.

```python
import seaborn as sns
import matplotlib.pyplot as plt
fig, ax = plt.subplots(figsize=(8, 4.5))
sns.boxplot(data=df, x="categoria", y="ingreso", hue="categoria", legend=False, ax=ax)
ax.set(title="Bebidas tiene la mediana más alta y la cola más larga", xlabel="", ylabel="Ingreso por ticket (USD)")
fig.text(0.01, 0.01, "Fuente: ventas.parquet · 16/11/2026", fontsize=8)
fig.savefig("figuras/categorias.png", dpi=150, bbox_inches="tight")
```

**Cuándo cada uno.** Plotly si van a hacer zoom o pasar el mouse: un HTML lo abre cualquier navegador. Seaborn si el gráfico va en un documento o una presentación: el PNG se ve igual en todos lados.

### 3. El título afirma, el gráfico declara su fuente
- **Mal:** `Ingresos mensuales`. **Bien:** `Los ingresos caen 18 % en enero`.
- El título es una **frase con verbo y, si se puede, cifra**; los nombres de las variables ya están en los ejes.
- **La fuente y las fechas del dato van en la imagen**, no en el chat: si alguien reenvía el PNG, la fuente viaja con él. Y si el último periodo está incompleto por el `resample`, anótalo.

## ⚠️ El error más común

> [!warning] `AttributeError: Can only use .dt accessor with datetimelike values`
> ```
> AttributeError: Can only use .dt accessor with datetimelike values
> ```
> **Qué significa:** usaste `.dt` sobre una columna que **es texto**, y ese mismo texto es la causa del eje amontonado: Plotly dibuja cada valor de texto como una categoría, no como un punto en el tiempo. **Cómo se lee:** mira el `dtype` antes de tocar el gráfico: `print(df["fecha"].dtype)`. Si dice `object` y no `datetime64[ns]`, el problema está en el dato. **Cómo se arregla:** convierte una vez, al principio, y cuenta lo que no se pudo convertir:
> ```python
> df["fecha"] = pd.to_datetime(df["fecha"], errors="coerce")
> print(df["fecha"].isna().sum())    # si es > 0, hay filas que no eran fechas válidas
> ```
> `errors="coerce"` no arregla el dato: lo vuelve `NaT` y te deja contarlo. Si lo tapas con `dropna()` sin mirar, pierdes filas y el total cambia sin que nadie lo note.

Este no es un error de adorno: es el que se repite en el proyecto si no lo has visto antes.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min

Los haces **en tu terminal / VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Convierte `fecha` a datetime y reporta cuántos `NaT` aparecieron; si hay, di de qué filas vienen.
2. Genera `figuras/ingresos.html` con Plotly y comprueba en el navegador que el eje muestra fechas.
3. Genera `figuras/categorias.png` con Seaborn, color por grupo, título que afirme algo y fuente con fechas.

### Reto — opcional, para quien quiera más

1. Haz small multiples (`facet_col` por categoría) y decide si se lee mejor o peor que un solo gráfico. Justifica.
2. Anota en el gráfico el periodo parcial y explica en una frase por qué no es una caída de ventas.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `figuras/ingresos.html` y `figuras/categorias.png` existen y abren fuera de VS Code
- [ ] El eje de fechas muestra fechas, no texto amontonado
- [ ] Cada título es una frase con un hallazgo, no el nombre de las columnas
- [ ] Los dos gráficos llevan fuente y fechas visibles **dentro** de la imagen
- [ ] Puedes decir en una frase qué pregunta responde cada gráfico y para quién lo hiciste

## 🔗 Para profundizar

- [Plotly Express](https://plotly.com/python/plotly-express/) — el punto de partida de todos los gráficos de este tema
- [Plotly — series de tiempo](https://plotly.com/python/time-series/) — qué leer si el eje de fechas no se comporta
- [Seaborn — distribuciones](https://seaborn.pydata.org/tutorial/distributions.html) · [Matplotlib — `savefig`](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.savefig.html)
- En el vault: [[01 · DuckDB y Parquet]] · [[01 · Streamlit, primera app]]

## 🧭 Problemas frecuentes

- **Las fechas no parsean** → pueden venir `dd/mm/aaaa`: prueba `dayfirst=True` y cuenta los `NaT`.
- **El PNG sale en blanco** → guardaste antes de dibujar o después de `plt.show()`: usa `fig.savefig(...)`.
- **El eje sigue amontonado** → la columna volvió a texto en algún paso. Mira el `dtype` otra vez.
- **El HTML pesa muchísimo** → `include_plotlyjs="cdn"` y se abre igual, con menos peso.

## ➡️ Siguiente tema

Pasas de un gráfico exportado a una **app**: el mismo gráfico, pero con un filtro que lo mueve y tres métricas arriba.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá las dos frases (una por gráfico), la ruta de los archivos y el texto del error que no pudiste resolver. El siguiente tema retoma lo que quede acá.
