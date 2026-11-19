---
tipo: tema
modulo: "01 Python básico y lógica"
capa: dominio inicial
herramienta: Python
resultado: "escribir condiciones que deciden por cada fila: qué venta es grande, qué cliente recibe descuento y qué dato llega vacío"
tiempo-lectura: 20 min
prerrequisitos: ["Variables, tipos y booleanos"]
tags: [tema, python, fundamentos]
---

# Condicionales

![[python.svg|64]]

> [!tip] En una frase
> Un **condicional** mira un booleano —un `True` o un `False`— y ejecuta un bloque de código solo si ese valor es `True`.

> [!info] Cómo leer este tema — 20 minutos
> El tema anterior dejó la materia prima: variables, comparaciones (`>`, `<`, `==`, `!=`, `>=`) y operadores lógicos (`and`, `or`, `not`), que ya devuelven `True` o `False`. **Este tema es lento a propósito:** un `if` sin condición, dos puntos o indentación no corre.

## 🤔 Para qué sirve

Escribes `150 > 100` y obtienes `True`, pero ese `True` en pantalla no hace nada: la venta no quedó marcada ni contada. **Un booleano calculado todavía no es una decisión tomada.** Un condicional lo convierte en una acción: imprimir, marcar, sumar o descartar.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Escribir un `if`/`else` que decida según un dato y comprobar los dos caminos |
| Después | Combinar condiciones con `and`/`or` y usarlas dentro de un `for` |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Ejemplo |
|---|---|---|
| **Condición** | expresión que da `True` o `False` | `monto > 100` |
| **`if`** | ejecuta su bloque solo si la condición es `True` | `if monto > 100:` |
| **`else`** | ejecuta su bloque en todos los demás casos | `else:` |
| **`elif`** | otra condición, evaluada solo si las anteriores dieron `False` | `elif monto >= 50:` |
| **Bloque** | las líneas indentadas que pertenecen al `if` | las que llevan 4 espacios |

## 🧭 Qué no es

- **Un `if` no repite nada.** Pregunta una vez por cada pasada; si el valor cambia después, no vuelve a mirar.
- **Un `if` no elige entre todos los caminos.** En cuanto encuentra un `True`, ejecuta ese bloque y salta el resto.
- **`else` no lleva condición.** Va `else:` a secas; `else monto > 100:` es un error de sintaxis.
- **Un condicional no filtra una tabla.** No modifica los datos: decide qué hacer con cada fila mientras la recorre.

## 🧠 Conceptos

1. **Un `if` mira un booleano** y ejecuta o no su bloque. Nada más.
2. **`elif` encadena opciones y el orden decide el resultado**, porque el primer `True` gana.
3. **La indentación es sintaxis**: el bloque va un nivel adentro de lo que lo contiene.

## 📖 Teoría

### 1. La forma mínima: `if` y `else`

```python
monto = 150
if monto > 100:
    print("Venta grande")
else:
    print("Venta normal")
```

La salida es `Venta grande`. La línea indentada es lo que pasa si la condición es `True`; `else:` y su línea cubren el resto. Con `monto = 45` la salida es `Venta normal`.

**La indentación.** Los dos puntos anuncian un bloque, y el bloque son las líneas que empiezan con **4 espacios**; cuando una línea vuelve al margen, el bloque terminó. Python acepta un tabulador si es consistente, pero no acepta mezclarlo con espacios (`TabError: inconsistent use of tabs and spaces in indentation`): por eso PEP 8 fija 4 espacios, y la tecla Tab los inserta sola.

### 2. `elif`: varias opciones en cadena

```python
monto = 85
if monto >= 100:
    print("Grande")
elif monto >= 50:
    print("Media")
else:
    print("Chica")
```

La salida es `Media`: 85 no llega a 100 y sí llega a 50. `elif` se evalúa solo si la condición anterior fue `False`.

**El orden decide el resultado.** Con las condiciones invertidas, un monto de 150 imprime `Chica`: gana el primer `True` y el `elif` de `monto >= 100` nunca se evalúa. El resultado está mal y no hay ningún mensaje: ordena de la condición más restrictiva a la más amplia.

### 3. Combinar condiciones

```python
monto = 150
cliente_nuevo = False
cliente_frecuente = False
if (monto > 100 or cliente_nuevo) and not cliente_frecuente:
    print("Descuento del 10 por ciento")
```

Imprime `Descuento del 10 por ciento`: el paréntesis es `True` (150 pasa de 100) y `not cliente_frecuente` también. `and` exige las dos partes, `or` se conforma con una, `not` invierte; `and` va antes que `or`, y los paréntesis fijan la prioridad.

### 4. `if` dentro de un `for`: decidir por fila

```python
ventas = [45, 150, 12, 320, 90]
for monto in ventas:
    if monto > 100:
        print(monto, "es grande")
    else:
        print(monto, "es normal")
```

Imprime `45 es normal`, `150 es grande`, `12 es normal`, `320 es grande` y `90 es normal`. `for monto in ventas:` toma cada valor de la lista, lo guarda en `monto` y repite el bloque; el `if` se evalúa una vez por cada valor. `ventas = [45, 150, 12, 320, 90]` es una **lista**, y el `for` se explica completo en el tema siguiente: aquí alcanza con leerlo como "para cada monto de la lista". Es la misma estructura que en Pandas aparece como columna nueva y en SQL como `CASE WHEN`: cambiar la herramienta no cambia la lógica.

### 5. Estilo: lo que sobra y el caso de `None`

`if cliente_nuevo == True:` funciona, pero sobra: `cliente_nuevo` ya es un booleano y compararlo con `True` no agrega información. La forma directa es `if cliente_nuevo:`, y para el caso contrario, `if not cliente_nuevo:`.

**`None`** es un valor especial que significa "ningún valor"; no es 0 ni texto vacío, y no apareció en el tema anterior. Se compara con `is None`, nunca con `== None`:

```python
cliente = None
if cliente is None:
    print("Sin cliente asignado")
```

Imprime `Sin cliente asignado`. `is` pregunta si es literalmente ese objeto, y PEP 8 indica usar `is` y `is not`.

## ⚠️ El error más común

> [!warning] `IndentationError: expected an indented block after 'if' statement on line 2` (salida real de Python 3.12)
> ```
>   File "ventas.py", line 3
>     print("Venta grande")
>     ^
> IndentationError: expected an indented block after 'if' statement on line 2
> ```
> **Qué significa:** después de `if monto > 100:` Python esperaba una línea indentada y encontró una al mismo nivel. **Cómo se lee:** la ruta va abreviada; la última línea dice qué pasó y en qué línea está el `if`, y el `^` señala el punto. **Cómo se arregla:** indentar con 4 espacios la línea que sigue al `if`.

El error hermano pierde los dos puntos: `if monto > 100` sin `:` al final se detiene con `SyntaxError: expected ':'`. La causa es la misma: sin los dos puntos no hay bloque. Aplica igual a `if`, `elif` y `else`.

El tercero no muestra ningún mensaje y es el más caro: la cadena con el orden invertido del apartado 2. El programa corre, imprime un resultado y ese resultado está mal; si una rama nunca cambia la salida, sospecha del orden.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Escribe un `if`/`else` con `monto = 150` que imprima `Venta grande` si el monto pasa de 100 y `Venta normal` si no; ejecútalo, cambia el valor a 45 y ejecútalo otra vez sin tocar el `if`.
2. Escribe una cadena `if`/`elif`/`elif`/`else` que clasifique un `monto` en cuatro categorías; ejecútala cuatro veces, una por rama, y guarda las cuatro salidas.
3. Provoca los dos errores: borra los 4 espacios de la línea que sigue al `if` y ejecuta; después devuélvelos, borra los dos puntos y ejecuta. Copia los dos mensajes, arregla cada línea y verifica que vuelve a correr.

### Reto — opcional

1. Un cliente recibe 15 % de descuento si el monto pasa de 500, o si es cliente nuevo con monto mayor que 100, y nunca si tiene deuda pendiente; escribe la condición con `and`, `or`, `not` y paréntesis, y pruébala con tres combinaciones.
2. Con `ventas = [45, 150, 12, 320, 90]`, cuenta cuántas ventas son grandes y cuántas normales usando dos variables contadoras dentro del `for`; imprime los dos totales.

## ✅ Criterios de logro

- [ ] Tu `if`/`else` imprime una cosa u otra según el valor de `monto`, sin tocar el `if`
- [ ] Tu cadena `elif` clasifica cuatro valores distintos en cuatro ramas distintas
- [ ] Explicas por qué `if monto >= 10:` como primera condición deja la otra rama sin ejecutar
- [ ] Reprodujiste el `IndentationError` y el `SyntaxError: expected ':'`, y escribes `if cliente_nuevo:` en lugar de `== True`

## 🔗 Para profundizar

- [Python: la sentencia `if`](https://docs.python.org/es/3/tutorial/controlflow.html#if-statements) y [PEP 8: indentación](https://peps.python.org/pep-0008/#indentation): cuatro espacios, nunca mezclados

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `IndentationError: expected an indented block...` | El bloque del `if` quedó vacío | Indentar la línea siguiente con 4 espacios |
| `SyntaxError: expected ':'` | Falta el `:` al final de `if`, `elif` o `else` | Agregarlo |
| Una rama nunca se ejecuta | El orden de las condiciones la tapa | Poner la más restrictiva primero |

## ➡️ Siguiente tema

**Bucles:** un `if` decide una vez por cada valor que recibe; un bucle repite la decisión, y ahí empieza el trabajo con datos.

## 📝 Mi resultado

> [!note] Tu espacio
> Pega las salidas de los tres ejercicios base y los dos mensajes de error con su versión corregida.

---
![[meme-dev-if.png]]