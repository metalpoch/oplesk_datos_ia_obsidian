---
tipo: tema
modulo: "06 Pandas"
capa: familiaridad
herramienta: "interpretación (no es una librería)"
resultado: "una página explicando por qué el ticket promedio miente"
tiempo-lectura: 15 min
prerrequisitos: ["Pandas, limpieza difícil"]
tags: [tema, estadistica, interpretacion]
---

# Estadística para leer el número (sin fórmulas)

![[pandas.svg|64]]

> [!tip] En una frase, antes de empezar
> En este curso la estadística no es calcular: es **leer un número y saber cuándo te está mintiendo**. No vas a demostrar nada; vas a decidir si el número que tienes delante sirve para algo.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y escribe `#duda` donde algo no te cierre; esas dudas son el arranque.
>
> **Lo que NO está acá:** la página del resultado. Se escribe con los datos reales, no leyendo.

## 🤔 Para qué sirve

Te piden el ticket promedio de la tienda. Da **6,22**. Con ese número, alguien decide comprar inventario para un cliente que compra mucho y volver a contratar al mismo ritmo.

El problema: **ese 6,22 no representa a nadie**. Un solo cliente que se llevó 4.000 en una compra lo arrastró hacia arriba; la mitad de las ventas fueron de 2 o 3. Si hubieras mirado la **mediana**, el número habría sido otro y la decisión también.

Eso es lo que este tema resuelve: que no te den un promedio y lo firmes. No es un tema de matemática — es un tema de criterio.

> [!note] Qué no es
> - **No es el promedio del colegio.** Ahí calculabas; acá decides si el cálculo sirve.
> - **No es inferencia estadística.** Eso —muestras, intervalos, significancia— **no está en este curso** y se estudia después. Acá solo se lee lo que ya está en los datos.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Explicar con los datos de la tienda por qué el ticket promedio miente, usando media y mediana |
| **Después** | Encontrar una correlación en los datos y escribir una tercera variable plausible que la explique |

En este tema no vas a calcular más: vas a leer. Un número puede ser verdadero y engañoso al mismo tiempo. El trabajo es aprender a preguntarle qué está escondiendo.

## 🧠 Conceptos

1. **Media y mediana cuentan historias distintas** — la media reparte el total entre todas las ventas; la mediana es el ticket del medio. Un caso extremo mueve la media y casi no toca la mediana.
2. **La dispersión es parte del número** — dos tiendas con el mismo promedio pueden tener comportamientos opuestos; sin mirar el rango, el promedio no dice nada.
3. **Correlación no es causalidad** — dos series que suben juntas pueden estar respondiendo a una tercera variable; la correlación no te dice cuál.

## 📖 Teoría
### 1. El ticket promedio que miente

```python
import pandas as pd
tickets = pd.Series([2, 2, 2, 2, 2, 2, 2, 2, 40])
print(round(tickets.mean(), 2))    # 6.22
print(tickets.median())  # 2.0
```

Ocho clientes pagaron 2 y uno pagó 40: un pedido mayorista, no un error. La media dice 6.22; la mediana dice 2. Ningún cliente pagó 6.22. La media es útil para proyectar ingresos totales; la mediana describe al cliente típico. Si el dueño lee "ticket promedio 6.22" y compra inventario para ese ticket, se equivoca con 8 de cada 9 clientes.

**Por qué importa:** el promedio es correcto y la conclusión es falsa. Ese es el error que este tema te enseña a detectar antes de escribir un informe.

### 2. Dispersión: el mismo promedio, dos realidades

```python
pd.DataFrame({"tienda_A": [4, 4, 4, 4, 4, 4],
              "tienda_B": [1, 1, 1, 7, 7, 7]}).agg(["min", "median", "max"])
```

Las dos tiendas tienen promedio 4. La A vende siempre 4: es estable. La B vende 1 o 7: es todo o nada. Si solo miras el promedio, tomas la misma decisión para las dos, y en una te sobra inventario mientras en la otra te falta. No hace falta una fórmula: con el mínimo, la mediana, el máximo y los percentiles ya ves la diferencia.

**Por qué importa:** un promedio sin dispersión es una foto sin contexto. Antes de decidir, pregunta siempre "¿y cómo se reparten esos valores?".

### 3. Correlación no es causalidad

```python
pd.DataFrame({"pan_hamburguesa": [10, 12, 11, 30, 32, 31],
              "carbon": [2, 3, 2, 9, 10, 9]}).corr()
```

Las dos series suben juntas: cuando se vende más pan de hamburguesa, también se vende más carbón. Sería absurdo concluir que comprar pan hace comprar carbón. La tercera variable es el descanso semanal: la gente hace asado y compra las dos cosas. En los datos de la tienda vas a encontrar correlaciones parecidas con el calor, la quincena o los feriados.

**Por qué importa:** si actúas sobre la correlación equivocada, stockeas lo que no corresponde. La correlación es una pista para buscar la tercera variable, no una prueba de causa.

> [!note] Lo que NO entra en este curso
> La inferencia formal —intervalos de confianza, tests de hipótesis, p-valores, regresión— queda fuera del alcance. Acá aprendes a leer un número, no a demostrar que una diferencia es estadísticamente significativa. Para después: [Think Stats](https://greenteapress.com/wp/think-stats-2e/) y [OpenIntro Statistics](https://openintro.org/book/os/).

## ⚠️ El error más común

> [!warning] `ZeroDivisionError: division by zero`
> ```
> >>> caras = ventas[ventas["total"] > 1000]
> >>> sum(caras["total"]) / len(caras)
> Traceback (most recent call last):
>   ...
> ZeroDivisionError: division by zero
> ```
> **Qué significa:** el filtro no dejó ninguna fila y dividiste por cero. No hay ticket que promediar; el error no es de pandas, es una pregunta sin datos. **Cómo se lee:** mira el denominador antes que el numerador: `len(caras)` o `count()` en cero. Si el filtro está vacío, el problema es el filtro, no el promedio. **Cómo se arregla:** pregunta cuántas filas hay antes de dividir y reporta "no hay ventas en ese rango", no un cero que parece un dato.
>
> **El que no hace ruido:** con `total = pd.Series([100, 120, None, 80, 60])`, `.mean()` devuelve `90.0`, pero `.count()` es 4 y `.size` es 5. Pandas salteó el nulo: si esperabas 72.0 (el total repartido entre 5), el número que ves es otro y nada te avisa. Compara `count()` con `size()` antes de confiar en un promedio.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Calcula la media y la mediana del ticket en los datos reales. Escribe cinco líneas: cuál describe al cliente típico, cuál sirve para proyectar ingresos y qué venta concreta está moviendo la media. (8 min)
2. Compara dos subconjuntos con promedios parecidos pero dispersión distinta (los días hábiles contra el sábado y el domingo, o dos categorías). Usa mínimo, mediana, máximo y percentiles; escribe qué decisión cambia en cada caso. (8 min)
3. Elige dos categorías o dos series diarias, calcula su correlación y escribe una tercera variable plausible. Agrega una explicación alternativa que también podría producir esa correlación. (9 min)

### Reto — opcional

1. Encuentra un caso donde media y mediana se separen mucho; quita el 1 % de tickets más altos y mide cuánto se mueve la media. Explica qué dice eso sobre reportar solo el promedio.
2. Escribe la página del resultado: un título que afirme el hallazgo, dos números reales, una tabla o gráfico y la decisión que cambia. No definas "media"; muestra el caso.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] La página afirma un hallazgo, no repite una definición de manual
- [ ] Usa media y mediana reales, dice de dónde salen y muestra un caso donde el promedio esconde una diferencia
- [ ] Distingue correlación de causalidad en una frase, con una tercera variable
- [ ] Puedes explicar en una frase por qué esto es interpretación y no cálculo

## 🔗 Para profundizar

- [pandas: `describe`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html) — mínimo, cuartiles y máximo de un vistazo
- [pandas: `mean`](https://pandas.pydata.org/docs/reference/api/pandas.Series.mean.html) — el parámetro `skipna` y los nulos
- [pandas: `corr`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.corr.html) — qué mide y qué no
- [Spurious Correlations](https://www.tylervigen.com/spurious-correlations) — ejemplos para entrenar el ojo

## 🧭 Problemas frecuentes

- **`ZeroDivisionError`** → el filtro quedó vacío. Revisa `len()` antes de dividir y reporta que no hay datos.
- **La media no coincide con lo que esperabas** → hay nulos que pandas salteó. Compara `count()` con `size()`.
- **Media y mediana muy lejos** → hay valores extremos o una distribución sesgada. Reporta las dos y no borres el extremo sin una razón de negocio.
- **Una correlación alta no tiene sentido** → busca la tercera variable (día, calor, quincena, feriado) antes de afirmar una causa.

## ➡️ Siguiente tema

Vas a consultar los mismos datos con SQL sobre archivos, sin servidor, y a ver por qué el motor cambia el tiempo: [[01 · DuckDB y Parquet]].

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (una captura, un conteo, el texto del error que no pudiste resolver). En el siguiente tema se retoma lo que quedó acá.
