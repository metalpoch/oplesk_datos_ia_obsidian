---
tipo: tema
modulo: "06 Pandas"
capa: dominio inicial
herramienta: pandas
resultado: "reporte de 5 métricas de perfilado"
tiempo-lectura: 15 min
prerrequisitos: ["Listas y diccionarios"]
tags: [tema, pandas, perfilado]
---

# Pandas: leer y perfilar antes de tocar nada

![[pandas.svg|64]]

> [!tip] En una frase, antes de empezar
> **pandas es una librería de Python que abre un archivo de datos y lo deja como una tabla con la que trabajas escribiendo código.** Esa tabla se llama `DataFrame`, y en este tema la miras sin cambiarle nada.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y escribe `#duda` donde algo no te cierre; esas dudas son el arranque.
>
> **Lo que NO está acá:** la solución del reporte. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

Cada mes llega el mismo `datos/ventas.csv` y cada mes lo abres en Excel: 50.000 filas que tardan en cargar, la columna de precios alineada a la izquierda porque es texto, las fechas en dos formatos y una ñ rota. Lo arreglas a mano, lo guardas como `ventas_limpias_FINAL.xlsx`, y al mes siguiente el archivo llega igual de sucio.

Y cuando alguien pregunta "¿cuánto vendimos en Bebidas en marzo?", la respuesta vive en una tabla dinámica que solo tú sabes armar, en un archivo que solo tú tienes.

**Eso es lo que pandas resuelve:** el análisis deja de ser manual. Abres el archivo con una línea, le haces preguntas sin modificarlo y dejas la receta en un script que el mes que viene corre igual.

> [!note] Qué no es
> Excel es para mirar y tocar el dato a mano, y cada cambio se queda sin historia. pandas es para repetir el mismo análisis todos los meses con un script. Tampoco es una base de datos: no hay servidor, ni usuarios, ni permisos; es un programa que corre en tu máquina sobre un archivo.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Cargar `datos/ventas.csv` en un DataFrame y describirlo sin modificar una sola fila |
| **Después** | Producir un reporte de 5 métricas que otra persona pueda leer para decidir qué limpiar |

En este tema el archivo de la tienda entra a pandas por primera vez. La tentación es limpiarlo ya; el trabajo aquí es mirarlo. Un dato que no entendiste no se arregla: se empeora.

## 🧠 Conceptos
1. **DataFrame y Serie** — el DataFrame es la tabla; una columna es una Serie, y su `dtype` decide qué operaciones tienen sentido.
2. **Perfilar antes de limpiar** — `shape`, `dtypes`, nulos, duplicados y rangos raros se miden primero, para tener una línea base contra la cual comparar.
3. **Filtro booleano** — una condición sobre una columna devuelve una Serie de `True`/`False`; con ella eliges filas sin tocar el archivo.

## 📖 Teoría
### 1. DataFrame, Serie y el dtype que manda

```python
import pandas as pd

ventas = pd.read_csv("datos/ventas.csv")
print(type(ventas))              # <class 'pandas.core.frame.DataFrame'>
print(type(ventas["precio_unit"]))    # <class 'pandas.core.series.Series'>
print(ventas["precio_unit"].dtype)    # object
```

`ventas` es la tabla completa; `ventas["precio_unit"]` es una columna, una Serie con su propio tipo. Con doble corchete, `ventas[["precio_unit"]]`, obtienes un DataFrame de una columna: no es lo mismo, y pandas no siempre te avisa cuando los confundes.

El `dtype` es el contrato de la columna. `int64` y `float64` son números: se suman, se promedian, se comparan. `object` significa "aquí hay lo que Python encontró", casi siempre texto. Un precio en `object` no se puede sumar: falla o, peor, concatena texto.

**Por qué importa:** el 80 % de los errores de este tema y del siguiente empiezan en un `dtype` que no era el que creías.

### 2. Las cinco medidas del perfilado

```python
print(ventas.shape)                  # (filas, columnas)
print(ventas.dtypes)                 # tipo de cada columna
print(ventas.isna().sum())           # nulos por columna
print(ventas.duplicated().sum())     # filas repetidas completas
print(ventas["total"].describe())    # min, cuartiles, max
```

Cada medida responde una pregunta distinta: `shape` el tamaño; `dtypes` qué operaciones son válidas; `isna().sum()` qué falta; `duplicated().sum()` cuántas filas son idénticas **en todas** sus columnas (una repetición por llave necesita `subset=`, y eso se ve en el tema de limpieza difícil); `describe()` el rango y los cuartiles de una numérica.

Cuidado con las fechas en texto: `ventas["fecha"].min()` no da la fecha más vieja, da la que gana alfabéticamente. Con dos formatos mezclados, ese "rango" parece razonable y es falso.

**Por qué importa:** el reporte de perfilado es la línea base. Sin él no puedes demostrar que tu limpieza cambió algo, ni cuánto cambió.

### 3. Filtro booleano: preguntarle a la tabla

```python
caras = ventas[ventas["total"] > 100]
print(len(caras))
print(caras[["fecha", "producto", "total"]].head())
```

La condición `ventas["total"] > 100` devuelve una Serie de `True` y `False`, del mismo largo que la tabla; al ponerla entre corchetes, pandas se queda solo con las filas donde dice `True`. Para combinar condiciones usa `&`, `|` y `~`, siempre con paréntesis: `ventas[(ventas["categoria"] == "Bebidas") & (ventas["total"] > 100)]`.

**Por qué importa:** casi toda pregunta de negocio es un filtro y un conteo. Y filtrar es la forma de inspeccionar las filas raras sin modificar el dato.

## ⚠️ El error más común

> [!warning] `DtypeWarning: Columns (2) have mixed types`
> ```
> >>> pd.read_csv("datos/ventas.csv")
> <stdin>:1: DtypeWarning: Columns (5) have mixed types. Specify dtype option on import or set low_memory=False.
> ```
> **Qué significa:** pandas leyó el archivo por bloques y en la columna 2 encontró números y texto; para no perder nada, dejó toda la columna como `object`. El warning es el aviso; el `dtype` es el hecho. **Cómo se lee:** `Columns (5)` cuenta desde 0: es la sexta columna del CSV, `precio_unit`. Busca comas decimales, símbolos de moneda, espacios o `N/D`: ahí está el 80 % de las veces. **Cómo se arregla:** no lo tapes con `low_memory=False`. Lee esa columna como texto (`dtype={"precio_unit": "string"}`), mira qué valores no son número y conviértelos con `pd.to_numeric(..., errors="coerce")`, contando cuántos quedan nulos. Si el archivo entra en un solo bloque y el warning no aparece, el `dtype` `object` te lo dice igual.

Este no es un error de adorno: es el que más caro sale si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Carga `datos/ventas.csv` e imprime `shape`, `dtypes` y `head(3)`. No modifiques nada. Anota cuántas columnas son `object` y cuáles deberían ser numéricas o fechas. (8 min)
2. Calcula las cinco métricas para el dataset completo y para solo las ventas de `Bebidas`. Para el rango de `fecha`, escribe por qué el mínimo y el máximo en texto no te sirven. (8 min)
3. Filtra las ventas con `total` mayor a 100 y las de categoría `"Bebidas"`. Luego usa `and` en vez de `&` a propósito, lee el error completo y corrige con paréntesis. (9 min)

### Reto — opcional

1. Escribe una función `perfila(df)` que devuelva un diccionario con las cinco métricas y las imprima en un formato que otra persona entienda sin ti.
2. Agrega el porcentaje de nulos por columna, ordénalo y explica cuál es la columna que no usarías para un informe y por qué.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Tu script corre de punta a punta y no modifica el CSV
- [ ] Imprime las cinco métricas y sabes cuántas columnas son `object`
- [ ] Señalas la columna de precio y explicas por qué todavía no se puede sumar
- [ ] Puedes explicar en una frase por qué usaste pandas y no una hoja de cálculo

## 🔗 Para profundizar

- [pandas: `read_csv`](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html) — los parámetros `dtype`, `usecols` y `low_memory`
- [pandas: 10 minutes to pandas](https://pandas.pydata.org/docs/user_guide/10min.html) — selección, filtros y columnas

## 🧭 Problemas frecuentes

- **`FileNotFoundError`** → estás en otra carpeta. Corre `pwd` y `ls`, y usa la ruta completa del CSV.
- **`KeyError: 'precio'`** → el nombre real tiene mayúscula, espacio o tilde. Imprime `ventas.columns.tolist()` y copia el nombre exacto.
- **`ValueError: The truth value of a Series is ambiguous`** → usaste `and` u `or`. Cambia a `&` y `|`, con paréntesis en cada condición.
- **El `DtypeWarning` no aparece** → no significa que esté limpio. Mira `ventas.dtypes`: si `precio` es `object`, el problema existe igual.

## ➡️ Siguiente tema

Vas a convertir esas fechas en `datetime` y esas categorías en seis valores canónicos, y a producir un reporte que diga cuántas filas entraron y cuántas salieron: [[02 · Pandas, limpiar y reportar]].

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (una captura, un conteo, el texto del error que no pudiste resolver). En el siguiente tema se retoma lo que quedó acá.
