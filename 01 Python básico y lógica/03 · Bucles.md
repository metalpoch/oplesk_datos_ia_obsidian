---
tipo: tema
modulo: "01 Python básico y lógica"
capa: dominio inicial
herramienta: Python
resultado: "recorrer una lista con for, sumar con un acumulador y filtrar filas con un if dentro del bucle"
tiempo-lectura: 20 min
prerrequisitos: ["Condicionales"]
tags: [tema, python, fundamentos]
---

# Bucles

![[python.svg|64]]

> [!tip] En una frase
> Un **bucle** repite un bloque sin copiarlo: recorre una colección elemento por elemento (`for`) o se repite mientras una condición sea verdadera (`while`).

> [!info] Cómo leer este tema — 20 minutos
> Ya viste un `for` de paso, en el tema anterior, para decidir por fila; aquí se explica completo. Lo demás se usa igual: variables con su tipo, comparaciones, `and`/`or`/`not` e `if`/`else`.
>
> **Dos formas de repetir y una costumbre.** `for` recorre una colección; `while` repite mientras algo sea verdadero; la costumbre es el acumulador, que se crea antes del bucle y crece adentro. Las listas aparecen solo como la colección que se recorre: tienen su tema propio después.

## 🤔 Para qué sirve

Tienes cuatro precios y quieres imprimir cada uno. Cuatro `print` lo resuelven, hasta que la lista crece a cuatrocientos y hay que corregir las cuatrocientas líneas si el mensaje cambia.

Excel ya resuelve esto: escribes la fórmula en la primera fila y arrastras el cuadro hacia abajo. **Arrastrar hacia abajo es un bucle.** En Python ese arrastre se escribe con código y no depende del mouse.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Recorrer una lista con `for` y acumular un total que cuadre |
| Después | Usar `range()` y un `while` que se corte solo, sin bucle infinito |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Ejemplo |
|---|---|---|
| **`for`** | repite una vez por cada elemento de una colección | `for venta in ventas:` |
| **`range()`** | números consecutivos, sin incluir el final | `range(1, 4)` → `1`, `2`, `3` |
| **`while`** | repite mientras la condición sea `True` | `while stock > 0:` |
| **Acumulador** | una variable que crece dentro del bucle | `total = total + venta` |
| **Cuerpo** | las líneas indentadas que se repiten | cuatro espacios bajo el `for` |

## 🧭 Qué no es

- **No reemplaza a un `if`.** El `for` repite; el `if` decide. Sin `if` adentro, hace lo mismo con todas las filas.
- **No siempre hace falta.** `sum(ventas)` suma la lista entera sin bucle, y sobre una tabla se usa Pandas (biblioteca de análisis de datos), que opera la columna completa. Revisa si la operación ya existe antes de escribir el `for`.

## 🧠 Conceptos

1. **`for` recorre; `while` repite mientras** — el `for` sabe cuántos elementos hay; el `while` no sabe cuántas vueltas dará y necesita algo que cambie su condición.
2. **La indentación es sintaxis** — lo indentado bajo el `for` es lo que se repite, ni una línea más.
3. **Lo creado antes del bucle sobrevive; reiniciarlo adentro borra lo anterior** — de ahí sale el acumulador y de ahí su error clásico.

## 📖 Teoría

### 1. `for` sobre una lista

```python
ventas = [3.50, 2.00, 5.25]
for venta in ventas:
    print("Vuelta:", venta)     # Vuelta: 3.5 | 2.0 | 5.25
print("Fin")                    # Fin
```

Por cada elemento de `ventas`, Python lo guarda en `venta` y ejecuta la línea indentada: tres elementos, tres vueltas. `print("Fin")` no está indentado y corre una sola vez, al final. `[3.50, 2.00, 5.25]` es una **lista**: valores entre corchetes separados por comas, y aquí se usa solo como la colección que se recorre. El `for` termina en dos puntos, el cuerpo va con **cuatro espacios** y `venta` es un nombre que eliges tú, en singular: no hay contador, el `for` avanza solo.

### 2. `range()`: repetir un número de veces

```python
for numero in range(5):
    print(numero)               # 0, 1, 2, 3, 4
for numero in range(1, 4):
    print(numero)               # 1, 2, 3
```

`range(5)` entrega cinco números, del `0` al `4`: empieza en `0` y **no incluye el final**. Con dos valores se elige el inicio, y el segundo tampoco: para contar del `1` al `5` se escribe `range(1, 6)`.

### 3. El acumulador

```python
ventas = [3.50, 2.00, 5.25]
total = 0
for venta in ventas:
    total = total + venta
    print("Voy en", total)      # 3.5 | 5.5 | 10.75
print("Total:", total)          # Total: 10.75
```

`total = total + venta` se lee de derecha a izquierda: "toma lo que valía `total`, súmale `venta` y guarda el resultado". Como la variable se crea antes, el valor no se pierde entre vueltas.

> [!warning] El error que no avisa
> ```python
> ventas = [3.50, 2.00, 5.25]
> for venta in ventas:
>     total = 0
>     total = total + venta
> print(total)      # 5.25
> ```
> Con `total = 0` **dentro** del bucle, cada vuelta empieza de cero y descarta lo anterior: imprime `5.25`, el último elemento, y no `10.75`. No hay mensaje y el número es creíble. La regla: **se inicializa antes, se acumula adentro**.

### 4. `while`: repetir mientras la condición sea verdadera

```python
stock = 3
while stock > 0:
    print("Quedan", stock)      # Quedan 3 | 2 | 1
    stock = stock - 1
print("Sin stock")              # Sin stock
```

`while` revisa la condición **antes** de cada vuelta: si es `True` ejecuta el cuerpo; si es `False`, sigue con la línea de abajo. El cuerpo tiene que cambiar algo que la condición mire: `stock = stock - 1` es lo que la vuelve falsa. Sin esa línea, `stock` vale `3` para siempre y el `print` se repite sin fin: la terminal no vuelve y hay que cortar con `Ctrl + C`. `for` cuando sabes cuántas vueltas quieres; `while` cuando el final depende de una condición.

### 5. `for` con `if`: recorrer y decidir fila por fila

Es la operación más común del análisis de datos, y lo que le falta al esquema del tema anterior es un acumulador adentro del `if`:

```python
ventas = [3.50, 2.00, 5.25]
total_caras = 0
for venta in ventas:
    if venta > 3:
        total_caras = total_caras + venta
print(total_caras)              # 8.75
```

Dos niveles de indentación: el `if` dentro del `for` se revisa en cada vuelta; el acumulador dentro del `if` corre solo cuando la condición se cumple. Si desindentas el acumulador un nivel, suma las tres ventas y da `10.75`; contar filas es el mismo esquema con `caras = caras + 1`.

## ⚠️ El error más común

```python
ventas = [3.50, 2.00, 5.25]
for venta in ventas:
print(venta)
```

> [!warning] `IndentationError: expected an indented block after 'for' statement on line 2`
> ```text
>   File "ventas.py", line 3
>     print(venta)
>     ^
> IndentationError: expected an indented block after 'for' statement on line 2
> ```
> **Qué significa:** el `for` espera un bloque indentado y lo encontró alineado con él. **Cómo se lee:** no hay `Traceback` porque el programa no llegó a ejecutarse: es un error de sintaxis. La primera línea trae la ruta del archivo y el número de línea; aquí la ruta va abreviada, en tu máquina muestra la carpeta completa. La flecha `^` marca dónde se notó el problema. **Cómo se arregla:** indentar el cuerpo con cuatro espacios.

El segundo error aparece cuando el acumulador es un número y un elemento es texto:

```python
ventas = ["3.50", 2.00, 5.25]
total = 0
for venta in ventas:
    total = total + venta
```

```text
Traceback (most recent call last):
  File "ventas.py", line 4, in <module>
    total = total + venta
            ~~~~~~^~~~~~~
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

`"3.50"` con comillas es texto. La última línea nombra los dos tipos que no se pueden sumar y los `~~~^~~~` señalan la operación exacta. Se corrige el dato, no la suma, después de confirmar con `print(type(venta))` cuál de los tres elementos sobra.

Los dos mensajes son salidas reales de Python 3.12. El tercer error no muestra ninguno: es el `total = 0` dentro del bucle del apartado 3. Imprime `5.25` en lugar de `10.75` y no avisa; se detecta comparando con una cuenta hecha a mano.

## 🏋️ Ejercicios

### Base — obligatorio

1. Crea una lista con cuatro precios y recórrela con `for`, un precio por línea. Después del bucle, imprime `Fin` y comprueba que sale una sola vez.
2. Suma esa lista con un acumulador: parcial adentro, total al final. Luego mueve `total = 0` adentro del bucle, ejecuta y anota la diferencia.
3. Con `range(1, 6)`, imprime `Fila 1` hasta `Fila 5`. Después quita la indentación de una línea del cuerpo, ejecuta y lee el mensaje completo; arregla el archivo.

### Reto — opcional

1. Escribe un `while` con `stock = 5` que reste de a `2` e imprima el stock en cada vuelta, mientras haya stock. Comprueba que no imprime ningún negativo y explica por qué se detuvo.
2. Con una lista de seis precios, cuenta cuántos pasan de `4` y suma solo esos, con dos acumuladores en el mismo `for`. Escribe un comentario que explique por qué hay dos.

## ✅ Criterios de logro

- [ ] Tu `for` recorre una lista e imprime un valor por elemento
- [ ] Sabes cuántos números entrega `range(5)` y por qué el `5` no aparece
- [ ] Tu acumulador se inicializa antes del bucle y el total coincide con la cuenta a mano
- [ ] Provocaste el error de `total = 0` adentro y explicas qué imprime y por qué
- [ ] Tu `while` termina solo, sin `Ctrl + C`
- [ ] Leíste un `IndentationError` completo y ubicaste la línea que lo causaba

## 🔗 Para profundizar

- [La sentencia `for` y `range()`](https://docs.python.org/es/3/tutorial/controlflow.html#for-statements)
- [`while`](https://docs.python.org/es/3/tutorial/introduction.html#first-steps-towards-programming)
- [`sum()`](https://docs.python.org/es/3/builtins/functions.html#sum) — suma una colección sin bucle

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `IndentationError: expected an indented block...` | El cuerpo del bucle quedó vacío | Indentar con 4 espacios |
| `IndentationError: unexpected indent` | Se indentó una línea que no va en el bucle | Alinear la línea con su bloque |
| `TypeError` con `'int'` y `'str'` | Un elemento de la lista es texto | `print(type(venta))` y corregir el dato |
| El total da el último valor | `total = 0` está dentro del bucle | Inicializar antes del `for` |
| La terminal no vuelve | `while` sin algo que cambie la condición | `Ctrl + C` y revisar el cuerpo |
| El `print` sale una sola vez | Quedó fuera del bucle | Indentarlo bajo el `for` |

## ➡️ Siguiente tema

**Listas y diccionarios:** una lista guarda muchas filas y un diccionario describe una sola con un nombre en cada campo, así que el bucle deja de recorrer números sueltos y empieza a recorrer registros.

## 📝 Mi resultado

> [!note] Tu espacio
> Pega la salida de los tres ejercicios base: los cuatro precios con `Fin` al final, el total con los dos resultados (correcto y con `total = 0` adentro), y el `IndentationError` tal como te salió.
