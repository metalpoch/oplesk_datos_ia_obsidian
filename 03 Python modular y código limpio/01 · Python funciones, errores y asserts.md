---
tipo: tema
modulo: "03 Python modular y código limpio"
capa: dominio inicial
herramienta: Python
resultado: "validar_ventas.py con la función que devuelve el total y la validación por asserts, más la salida con fila y campo"
tiempo-lectura: 15 min
prerrequisitos: ["Listas y diccionarios"]
tags: [tema, python, funciones, errores, fundamentos]
---

# Python: funciones, errores y asserts

![[python.svg|64]]

> [!tip] En una frase, antes de empezar
> **Una función es un pedazo de código con nombre: le entregas algo, hace su trabajo y te devuelve un resultado, y desde cualquier otro archivo la llamas por ese nombre.** `def` la define y `return` hace la entrega; `assert` es un chequeo que la hace fallar si el dato llegó mal.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea. Esas dudas son el punto de partida.
>
> **Lo que NO está acá:** la solución del ejercicio. Se construye en la práctica, paso a paso; si ya viene resuelta, la nota no sirve.

## 🤔 Para qué sirve

Tu suma de ventas del tema anterior funcionó. La copiaste al script de otro informe, y a un tercero para la prueba de un compañero. Ahora hay tres archivos con la misma suma copiada adentro:

- Encuentras un error en la suma. Lo arreglas en uno. **Los otros dos siguen mal** y nadie se acuerda de que existen.
- El script imprime el total pero no lo devuelve, así que ese total no sirve para nada más. Para el IVA lo vuelves a calcular, con otro `for` copiado.
- Un archivo llega con `total` en `"4,20"` y el programa explota. O peor: un `except: pass` se lo come y el total sale mal **sin que nada falle**.

**Eso es lo que resuelven las funciones y los `assert`.** Una función es una pieza con nombre y contrato: entra la lista, sale el número, y se arregla en un solo lugar para los tres scripts. Un `assert` es la misma idea al revés: si el dato no cumple lo que prometió, el programa se detiene **ahí**, con un mensaje que dice qué fila y qué campo.

> [!note] Qué no es
> - **`print` no es `return`.** Imprimir muestra el valor en la terminal y deja la función devolviendo `None`; lo único que se puede usar tres líneas más abajo sale por `return`.
> - **`try/except` no arregla el error.** El `if` decide sobre un dato que ya tienes; el `except` atrapa algo que ya se rompió, y sirve para decir qué fila falló y relanzar, no para tapar.
> - **`assert` no es un validador de formularios.** Se escribe para errores de programación; si el dato lo escribe un usuario final, el chequeo desaparece con `python -O` y el programa sigue de largo con lo que llegó.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Envolver la suma del tema anterior en `total_ventas(ventas)`: recibe la lista, **devuelve** el número, y lo imprimes desde otro archivo |
| **Después** | Correr `validar_venta(venta, numero)` y que falle con un mensaje que diga **cuál fila** y **qué campo** está mal, no un `AssertionError` pelado |

La suma que escribiste a mano se vuelve una pieza que se llama desde cualquier lado y que **avisa** cuando los datos están mal.

## 🧠 Conceptos

1. **Una función es un contrato: entra algo, sale algo** — `def` dice qué recibe, `return` dice qué entrega. Imprimir por dentro no es devolver.
2. **`try/except` es para controlar el error, no para esconderlo** — se atrapa para reportar cuál fila falló; `except: pass` convierte un bug en un número mal.
3. **`assert` es un contrato que grita** — falla temprano, con tu mensaje, en la línea del problema. Sin mensaje, `AssertionError` no te cuenta nada.

## 📖 Teoría

### 1. `def` y `return`: imprimir no es devolver

```python
def con_return(numeros):
    return sum(numeros)      # devuelve 3.5
def con_print(numeros):
    print(sum(numeros))      # imprime 3.5 y devuelve None
print(con_return([3.5]))     # 3.5
print(con_print([3.5]))      # imprime 3.5 y después None
```

Sin `return`, toda función devuelve `None`, y `None` no se suma ni se indexa: explota más tarde y más lejos. Imprimir sirve para mirar; devolver, para construir: tu `total_ventas` de este tema tiene que devolver.

### 2. `try/except`: atrapar para reportar

```python
ventas = [{"categoria": "panaderia", "total": "2,0"}]
total = 0
for numero, venta in enumerate(ventas, start=1):
    try:
        total = total + venta["total"]
    except TypeError as error:
        print(f"fila {numero} ({venta['categoria']}): {error}")
        raise
print("total:", total)
```

Imprime `fila 1 (panaderia): unsupported operand type(s) for +: 'int' and 'str'` y después el traceback: el `raise` relanza la excepción y el programa **termina** en vez de seguir con un total incompleto. Atrapas para decir la fila, no para tapar. Cambia esas líneas por `except: pass` y el mismo programa imprime `total: 0`, avisa que todo salió bien y se comió una venta en silencio.

### 3. `assert`: el contrato con mensaje

```python
def validar_venta(venta, numero):
    assert venta["total"] > 0, f"fila {numero}: el campo total debe ser > 0, vino {venta['total']}"

validar_venta({"categoria": "bebidas", "total": 3.5}, 1)   # pasa en silencio
validar_venta({"categoria": "bebidas", "total": -1}, 3)    # AssertionError: fila 3: el campo total debe ser > 0, vino -1
```

La coma después de la condición es el mensaje: `assert condición, "qué pasó"`. Para el dato sucio que no es número, el chequeo es `isinstance(valor, (int, float))`. Los `assert` se apagan con `python -O`: son para errores de programación, no para validar lo que tipea un usuario final.

## 🧠 Dos piezas que aparecen aquí por primera vez

> [!info] `f"..."` y `enumerate()`
> Hasta ahora imprimiste separando con comas: `print("fila", numero, ":", error)`. Eso funciona, pero mezcla texto y valores de forma incómoda.
>
> Un **f-string** es un texto con valores adentro, y se escribe poniendo una `f` antes de las comillas y el valor entre llaves:
>
> ```python
> numero = 3
> categoria = "bebidas"
> print(f"fila {numero} ({categoria}): el total no cuadra")
> # fila 3 (bebidas): el total no cuadra
> ```
>
> `enumerate()` devuelve el **número de vuelta** además del valor, sin que tengas que llevar un contador a mano:
>
> ```python
> ventas = ["pan", "café", "harina"]
> for numero, venta in enumerate(ventas, start=1):
>     print(f"{numero}. {venta}")
> # 1. pan
> # 2. café
> # 3. harina
> ```
>
> Con `start=1` la cuenta empieza en uno, que es como la lee una persona. Sin eso empieza en cero, como los índices de una lista.

## ⚠️ El error más común

> [!warning] `AssertionError` sin mensaje
> ```
> Traceback (most recent call last):
>   File "ventas.py", line 10, in <module>
>     validar_venta({"categoria": "bebidas", "total": -1}, 3)
>   File "ventas.py", line 8, in validar_venta
>     assert venta["total"] > 0
>            ^^^^^^^^^^^^^^^^^^
> AssertionError
> ```
> **Qué significa:** la condición del `assert` dio `False` y no le pusiste mensaje: el programa hizo bien en parar, pero no te dijo *qué* dato estaba mal. **Cómo se lee:** la última línea dice **qué** pasó; la de arriba, **dónde** (archivo y línea). Con `AssertionError` solo tienes el lugar, no el dato. **Cómo se arregla:** mensaje con fila, campo y valor. Arreglado se lee `AssertionError: fila 3: el campo total debe ser > 0, vino -1`: ya sabes qué fila abrir.

Y el primo hermano: la función se olvidó el `return`, devolvió `None` y explotó tres líneas después.

```
Traceback (most recent call last):
  File "caja.py", line 11, in <module>
    print(total_con_iva(ventas))
  File "caja.py", line 8, in total_con_iva
    return total_ventas(ventas) * 1.16
           ~~~~~~~~~~~~~~~~~~~~~^~~~~~
TypeError: unsupported operand type(s) for *: 'NoneType' and 'float'
```

`total_ventas` **no aparece** en el traceback: acusa a la línea 8, pero el bug está en la función que no figura porque terminó "bien". Cuando veas `NoneType`, mira la función de la línea anterior y pregúntale si tiene `return`.

Este no es un error de adorno: es el que más se repite si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o en VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. **(8 min)** En `total_ventas.py`, envuelve tu suma del tema anterior en `def total_ventas(ventas):`. Debe **devolver** el total, no imprimirlo. Llámala con dos listas distintas y con `[]`, e imprime los tres resultados desde afuera.
2. **(7 min)** Borra el `return` a propósito y corre el script. Copia el traceback y anota en una frase: ¿qué línea te acusa y en qué función está el problema real?
3. **(10 min)** Escribe tu propia `validar_venta(venta, numero)`: que exista el campo `total`, que sea número, que sea mayor a 0. Cada `assert` con mensaje que incluya **fila**, **campo** y **valor recibido**. Pruébala con `{"total": "4,20"}` y pega el `AssertionError` completo.

### Reto — opcional

1. Haz `revisar_ventas(ventas)` que devuelva `{"total": ..., "descartadas": ...}` en vez de imprimir, y decide si cortas en la primera fila mala o juntas todas para reportar al final. Deja la razón en un comentario.
2. Envuelve la llamada en el `main` con `try/except AssertionError` para imprimir el mensaje sin traceback. Prohibido `except: pass`.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `python3 total_ventas.py` corre sin traceback y muestra tres totales, `0` en la lista vacía
- [ ] `print(type(total_ventas(ventas)))` imprime `<class 'float'>`: **devuelve**, no imprime
- [ ] Tu `AssertionError` incluye número de fila, nombre del campo y el valor que vino
- [ ] Explicas en una frase por qué `except: pass` es peor que no poner nada

## 🔗 Para profundizar

- [Errores y excepciones — tutorial oficial de Python en español](https://docs.python.org/es/3/tutorial/errors.html) — las secciones de `try`/`except` y de lanzar excepciones
- [La sentencia `assert` — referencia del lenguaje](https://docs.python.org/es/3/reference/simple_stmts.html#the-assert-statement) — una página corta, explica lo de `python -O`
- [Definir funciones — tutorial oficial](https://docs.python.org/es/3/tutorial/controlflow.html#defining-functions) — valores por defecto y `return`, en español
- Anexo del vault: [[Chuleta de errores]]

## 🧭 Problemas frecuentes

- **Imprime el total "y además un `None`"** → falta el `return`, o está dentro del `for` y devuelve en la primera vuelta. Sácalo del bucle.
- **`NameError: name 'total' is not defined`** → usas desde afuera una variable creada adentro de la función. Lo de adentro no sale: sale el `return`.
- **`IndentationError: expected an indented block`** → falta indentar el cuerpo del `def` o del `try`. Cuatro espacios, nunca tabs.
- **El `assert` no salta aunque el dato está mal** → la condición está invertida, o corriste con `python -O`. Prueba `python3 archivo.py` a secas.

Si algo no te cierra y nada de esto lo resuelve, anota el traceback completo en "Mi resultado".

## ➡️ Siguiente tema

Vas a meter todo esto en un repo con commits, ramas y un pull request, para que cada cambio tenga un punto de retorno.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (los tres totales, el `AssertionError` con mensaje o el traceback que no pudiste resolver). En el siguiente tema se retoma por lo que quedó acá.
