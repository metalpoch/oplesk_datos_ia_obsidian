---
tipo: tema
modulo: "01 Python básico y lógica"
capa: dominio inicial
herramienta: Python
resultado: "ventas.py con el total por categoría en un diccionario"
tiempo-lectura: 20 min
prerrequisitos: ["Bucles"]
tags: [tema, python, fundamentos]
---

# Listas y diccionarios

![[python.svg|64]]

> [!tip] En una frase
> Una **lista** entra por posición y un **diccionario** entra por nombre; juntas, una lista de diccionarios es una tabla.

> [!info] Cómo leer este tema — 20 minutos
> Una sola responsabilidad: **guardar datos y agruparlos**; los tipos y el `for` tienen su propio tema. Lee completo una vez y escribe `#duda` en la línea que no cierre. La solución no está aquí.

## 🤔 Para qué sirve

Tienes una hoja de cálculo con dos columnas, **categoría** y **monto**. Son seis filas, y sumarlas a mano es un minuto. El problema aparece cuando la tabla crece, supongamos 50.000 filas:

- Quieres el total **por categoría**, y cada categoría nueva obliga a rehacer la tabla dinámica.
- La hoja se traba al abrir el archivo, o directamente no lo abre.
- Necesitas que el cálculo quede escrito: la próxima vez lo ejecutas y ya está.

**Python guarda esa tabla en una forma que puedes leer:** cada fila es un diccionario, porque cada dato tiene nombre de columna, y el conjunto de filas es una lista de diccionarios. Agrupar por categoría son seis líneas, la misma forma que esperan `groupby` en Pandas y `GROUP BY` en SQL: un `if` dentro de un `for` hace aquí lo que allá resuelve una línea.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Guardar una tabla como lista de diccionarios y recorrerla |
| Después | Agrupar con un diccionario acumulador y obtener el total por categoría |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Ejemplo |
|---|---|---|
| **Lista** | colección ordenada de valores; se entra por posición | `[3.5, 2.0, 1.5]` |
| **Índice** | la posición de un elemento; la primera es `0` | `ventas[0]` → `3.5` |
| **Diccionario** | colección de pares clave → valor; se entra por nombre | `{"categoria": "bebidas", "monto": 3.5}` |
| **Lista de diccionarios** | las filas de una tabla; cada diccionario es una fila | `[{"categoria": "bebidas"}, ...]` |

## 🧭 Qué no es

- **Una lista no es una columna de Excel.** No trae encabezado, no aplica formato y no recalcula.
- **Un diccionario no se entra por posición.** `venta[0]` falla: `0` no es una clave.
- **`in` no significa lo mismo en las dos.** En la lista pregunta por un valor; en el diccionario, por una clave.
- **Una lista de diccionarios no es una base de datos:** es la estructura que Pandas y SQL consumen.

## 🧠 Conceptos

1. **La lista entra por posición, el diccionario por nombre.** Elegir mal la estructura explica la mitad de los errores.
2. **Una lista de diccionarios es una tabla:** N diccionarios con las mismas claves son filas y columnas.
3. **El diccionario acumulador agrupa:** crear la clave en `0` y sumar es el `groupby` de Pandas y el `GROUP BY` de SQL.

## 📖 Teoría

### 1. La lista: posición, operaciones y recorrido

```python
ventas = [3.5, 2.0, 1.5]

print(ventas[0])          # 3.5      -> primera posición
print(ventas[-1])         # 1.5      -> última, sin cuentas
print(len(ventas))        # 3        -> cuántos elementos hay

ventas.append(4.0)        # agrega al final
print(ventas.count(2.0))  # 1        -> cuántas veces aparece
print(4.0 in ventas)      # True     -> buscar
print(ventas.index(1.5))  # 2        -> posición de la primera aparición

total = 0
for monto in ventas:
    total = total + monto

print(total)              # 11.0
```

La primera posición es `0`, no `1`, así que la última es `len(ventas) - 1` o `ventas[-1]`. `append()` agrega al final, `count()` cuenta apariciones, `in` responde sí o no e `index()` da la posición. El `for` recorre la lista un elemento por vuelta y tiene su propio tema; aquí solo se aplica.

### 2. Un diccionario describe una fila con nombres

```python
venta = {"categoria": "bebidas", "monto": 3.5}

print(venta["categoria"])         # bebidas
print("monto" in venta)           # True   -> la clave existe
print(venta.get("descuento", 0))  # 0      -> no falla si la clave falta
```

Un diccionario es `clave: valor`: la clave es el nombre del campo y el valor es el dato de la fila. Se entra con corchetes y el nombre exacto; `get()` devuelve el valor por defecto en lugar de fallar.

### 3. De la tabla de ventas al total por categoría

```python
ventas = [
    {"categoria": "bebidas", "monto": 3.5},
    {"categoria": "panaderia", "monto": 2.0},
    {"categoria": "bebidas", "monto": 1.5},
]

print(len(ventas))                 # 3
print(ventas[0]["categoria"])      # bebidas

totales = {}
for venta in ventas:
    categoria = venta["categoria"]
    if categoria not in totales:
        totales[categoria] = 0
    totales[categoria] = totales[categoria] + venta["monto"]

print(totales)    # {'bebidas': 5.0, 'panaderia': 2.0}
```

Son 3 filas y 2 columnas: las claves son las columnas y cada diccionario es una fila. `ventas[0]["categoria"]` se lee de afuera hacia adentro, la lista por la posición y la fila por la clave. Es la misma tabla que Pandas convierte en DataFrame y que en SQL se guarda con esos dos nombres de columna.

**La clave se crea en `0` antes de sumar.** Sin ese `if`, la primera venta de una categoría intenta sumar sobre una clave que no existe y el programa falla con `KeyError`. Con `get()` la misma suma se escribe en una línea; `groupby` de Pandas y `GROUP BY` de SQL reemplazan este bloque completo.

## ⚠️ El error más común

> [!warning] `KeyError: 'bebidas'`
> ```
> Traceback (most recent call last):
>   File "ventas.py", line 3, in <module>
>     print(venta["bebidas"])
>           ~~~~~^^^^^^^^^^^
> KeyError: 'bebidas'
> ```
> **Qué significa:** entraste con un **valor** (`"bebidas"`) en lugar del **nombre de la clave** (`"categoria"`). Python busca la clave `'bebidas'`, no la encuentra y se detiene. **Cómo se lee:** la última línea dice qué pasó; la de arriba, en qué archivo y en qué línea. **Cómo se arregla:** imprime la fila con `print(venta)`, mira las claves como están escritas y usa una de ellas, o `venta.get(clave)`. La mitad de los `KeyError` reales son `"total"` contra `"monto"`.

## 🏋️ Ejercicios

### Base — obligatorio

En un archivo `ventas.py`, desde la terminal o VS Code. Pega la salida en "Mi resultado".

1. Crea la lista de seis filas e imprime cuántas hay y la categoría de la primera.

   ```python
   ventas = [
       {"categoria": "bebidas", "monto": 3.5},
       {"categoria": "panaderia", "monto": 2.0},
       {"categoria": "bebidas", "monto": 1.5},
       {"categoria": "abarrotes", "monto": 4.2},
       {"categoria": "limpieza", "monto": 3.0},
       {"categoria": "panaderia", "monto": 1.0},
   ]
   ```

2. Agrega dos filas con `append()`: `bebidas` con `2.5` y `limpieza` con `2.5`. Cuenta las de `bebidas` con un `for`; debe dar 3.
3. Imprime `categoria` y `monto` de cada fila; después usa `ventas[0]["bebidas"]`, lee el traceback completo y anota qué confundiste.

### Reto — opcional

1. Arma el diccionario acumulador con los totales por categoría e imprime cuántas categorías distintas hay.
2. Encuentra la fila de **mayor** `monto`, guardándola entera en una variable, e imprime su categoría.

El reto no es obligatorio para aprobar. Suma si puedes explicar la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Tu archivo crea la lista de seis filas y la recorre sin traceback
- [ ] Accedes a un diccionario por clave y explicas la diferencia con la posición
- [ ] Tu diccionario acumulador devuelve el total por categoría
- [ ] Explicas en una frase por qué `venta["bebidas"]` falla y `venta["categoria"]` no
- [ ] Describes con tus palabras qué forma tiene una tabla escrita en Python

## 🔗 Para profundizar

- [Estructuras de datos, tutorial oficial](https://docs.python.org/es/3/tutorial/datastructures.html) — capítulo 5
- [Tipos incorporados](https://docs.python.org/es/3/library/stdtypes.html) — los métodos de `list` y `dict`
- Anexo del vault: [[Chuleta de errores]]

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `IndexError: list index out of range` | Pides una posición que no existe | Revisa `len(ventas)`; la última posición es `len(ventas) - 1` |
| `KeyError: 'total'` | La fila no tiene esa clave | Imprime la fila y copia el nombre exacto del campo |
| `TypeError: list indices must be integers or slices, not str` | Tratas la lista como diccionario | Entra primero a la fila: `ventas[0]["categoria"]` |
| El diccionario de totales queda incompleto | La clave se crea dentro de la suma, sin el `if` previo | Crea la clave en `0` antes de sumar |

## ➡️ Siguiente tema

**Módulo 03, funciones y código modular:** envuelves el recorrido en una función que reciba la lista y devuelva los totales.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega el diccionario de totales, el traceback del ejercicio 3 y la clave que confundiste.

---
![[meme-python-set-dictionary.webp]]