---
tipo: tema
modulo: "01 Python básico y lógica"
capa: dominio inicial
herramienta: Python
resultado: "guardar valores con nombre y escribir tu primera comparación que devuelve True o False"
tiempo-lectura: 20 min
prerrequisitos: []
tags: [tema, python, fundamentos]
---

# Variables, tipos y booleanos

![[python.svg|64]]

> [!tip] En una frase
> Una **variable** es un nombre que le pones a un valor para no tener que repetirlo, y un **booleano** es el valor más pequeño que existe: solo puede ser `True` o `False`.

> [!info] Cómo leer este tema — 20 minutos
> Este es el primer tema del programa y no da nada por conocido. Si algo no cierra, escribe `#duda` en esa línea.
>
> **Este tema es lento a propósito.** Son cuatro cosas y ninguna se puede saltear: guardar un valor, saber de qué tipo es, compararlo, y obtener un resultado que es verdadero o falso. Todo lo demás del curso se apoya en eso.

## 🤔 Para qué sirve

Tienes una venta de 3,50 y otra de 2,00. Quieres el total.

Podrías escribir `3.50 + 2.00` y obtener `5.5`. Pero mañana los precios son otros, y el mes que viene son cincuenta mil ventas. **Escribir los números en el código no escala.**

Una variable resuelve eso: le pones un nombre al valor y trabajas con el nombre. Y cuando los datos cambian, cambias el valor, no la cuenta.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Guardar cinco valores con su tipo correcto e imprimirlos con `type()` |
| Después | Escribir comparaciones que devuelvan `True` o `False` y guardarlas en variables |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Ejemplo |
|---|---|---|
| **Variable** | un nombre que apunta a un valor | `precio = 3.5` |
| **Tipo** | de qué clase es ese valor | `3.5` es número, `"3,5"` es texto |
| **Booleano** | un valor que solo puede ser verdadero o falso | `True` / `False` |
| **Comparación** | una operación que devuelve un booleano | `precio > 3` → `True` |

## 🧭 Qué no es

- **`=` no es "es igual a".** `=` guarda; `==` compara. Es el error número uno de todo el que empieza.
- **Un booleano no es un número**, aunque `True` se parezca a `1`. Son tipos distintos y Python lo distingue.

## 🧠 Conceptos

1. **Una variable tiene nombre, valor y tipo** — y el tipo lo decide el valor, no el nombre.
2. **Cada tipo se comporta distinto** — `3 + 2` suma; `"3" + "2"` pega texto y da `"32"`.
3. **Una comparación produce un booleano** — y ese booleano es lo que después va a decidir un `if`.

## 📖 Teoría

### 1. Guardar un valor con nombre

```python
precio = 3.50
cantidad = 2
total = precio * cantidad

print(total)        # 7.0
```

Cuatro líneas, tres variables. Se lee así: "`precio` vale 3.50; `cantidad` vale 2; `total` vale el producto de los dos; imprime `total`".

El nombre lo eliges tú. Se escribe en minúsculas, sin espacios, y describe qué es. `p` funciona pero al día siguiente no sabes qué es. `precio_unitario` sí.

### 2. Los cuatro tipos que vas a usar siempre

```python
edad = 30                  # int    -> número entero
precio = 3.50              # float  -> número con decimales
nombre = "Harina P.A.N."   # str    -> texto, siempre entre comillas
disponible = True          # bool   -> verdadero o falso
```

Para saber de qué tipo es algo:

```python
print(type(precio))     # <class 'float'>
print(type(nombre))     # <class 'str'>
```

> [!warning] El tipo lo decide el valor, no el nombre
> `precio = "3.50"` con comillas es **texto**, aunque parezca un número. Y con texto no se puede multiplicar:
> ```python
> precio = "3.50"
> print(precio * 2)    # 3.503.50   <- pegó el texto dos veces
> print(precio + 1)    # TypeError
> ```
> Esa es la primera trampa del curso, y viene de acá.

### 3. Operar con números: los seis operadores

Con los números se opera, y hay seis operadores. Los cuatro primeros los conoces de la calculadora; los otros dos son los que sorprenden.

| Operador | Qué hace | Ejemplo | Resultado |
|---|---|---|---|
| `+` | suma | `3 + 2` | `5` |
| `-` | resta | `3 - 2` | `1` |
| `*` | multiplica | `3 * 2` | `6` |
| `/` | divide, y **siempre devuelve decimal** | `7 / 2` | `3.5` |
| `//` | divide y **descarta los decimales** | `7 // 2` | `3` |
| `%` | devuelve **lo que sobra** de la división | `7 % 2` | `1` |

```python
precio = 3.50
cantidad = 2

print(precio * cantidad)      # 7.0   <- el total de la venta
print(10 / 4)                 # 2.5   <- la división normal devuelve decimal
print(10 // 4)                # 2     <- la división entera se queda con el 2
print(10 % 4)                 # 2     <- el módulo dice cuánto sobra
print(2 ** 3)                 # 8     <- potencia: 2 elevado a 3
```

> [!warning] La división que más confunde
> `10 / 4` da `2.5` aunque los dos números sean enteros: en Python la división normal **siempre** devuelve decimal. Si quieres el cociente entero, es `//`. Y si quieres saber si un número es par, es `% 2 == 0`: si el resto es cero, es par.
>
> ```python
> print(10 / 4)        # 2.5
> print(10 // 4)       # 2
> print(10 % 2 == 0)   # True   <- 10 es par
> print(7 % 2 == 0)    # False  <- 7 es impar
> ```

**Por qué importa:** el primer cálculo de todo el curso es `total = precio * cantidad`. Esa línea es este operador. Y cuando más adelante haya que promediar, `sum(x) / len(x)` solo da el resultado correcto porque `/` devuelve decimal: con `//` el promedio de 7 y 8 daría 7 en vez de 7.5.

El orden también importa: la multiplicación se resuelve antes que la suma, igual que en matemáticas. Si dudas, usa paréntesis.

```python
print(2 + 3 * 4)      # 14, no 20
print((2 + 3) * 4)    # 20
```

### 4. Comparar: de aquí salen los booleanos

Una comparación **no** devuelve un número: devuelve `True` o `False`.

```python
print(3 > 2)          # True
print(3 < 2)          # False
print(3 == 3)         # True    (== compara)
print(3 != 3)         # False   (!= significa "distinto de")
print(3 >= 3)         # True
```

Y con variables:

```python
precio = 3.50
print(precio > 3)         # True
print(precio == 3.5)      # True
print(precio != 3.5)      # False
```

Ese `True` o `False` **es un valor**, igual que `7.0` o `"Harina"`. Se puede guardar:

```python
es_caro = precio > 3
print(es_caro)            # True
print(type(es_caro))      # <class 'bool'>
```

> [!tip] Por qué importa tanto
> La próxima vez que veas un `if`, vas a entender qué pasa: un `if` no es magia, es una pregunta que devuelve `True` o `False` y decide por ahí. Sin entender los booleanos, un `if` es una fórmula que hay que copiar. Con ellos, es una decisión.

### 5. Combinar booleanos

```python
precio = 3.50
cantidad = 2

print(precio > 3 and cantidad > 1)    # True  (las dos son verdaderas)
print(precio > 5 or cantidad > 1)     # True  (alcanza con una)
print(not precio > 5)                 # True  (niega: "no es cierto que...")
```

- `and` — verdadero solo si **ambas** lo son.
- `or` — verdadero si **alguna** lo es.
- `not` — da vuelta el resultado.

## ⚠️ El error más común

> [!warning] `TypeError: unsupported operand type(s) for +: 'int' and 'str'`
> ```
> Traceback (most recent call last):
>   File "ventas.py", line 3, in <module>
>     total = precio + cantidad
> TypeError: unsupported operand type(s) for +: 'int' and 'str'
> ```
> **Qué significa:** uno de los dos valores es texto y el otro no. Python no adivina: si no le dices el tipo, no opera. **Cómo se lee:** la última línea dice **qué** pasó y con qué tipos. La de arriba dice **dónde**. **Cómo se arregla:** mira el valor antes de operar con `print(type(precio))`. Si es `str`, conviértelo con `float(precio)` — pero primero entiende por qué llegó como texto.

El otro error, más silencioso y peor: `=` donde ibas a poner `==`.

```python
precio = 3.5      # guarda
precio == 3.5     # compara, devuelve True (no guarda nada)
```

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces en tu terminal o en VS Code. Pega la salida en "Mi resultado".

1. Crea cinco variables: `precio`, `cantidad`, `producto`, `disponible` y `cliente`. Guárdale a cada una un valor del tipo que le corresponde. Imprime cada una con `print(valor, type(valor))`.
2. Escribe cinco comparaciones sobre esas variables (una con `>`, una con `<`, una con `==`, una con `!=`, una con `>=`) y guarda cada resultado en una variable booleana. Imprime las cinco.
3. Provoca el error a propósito: escribe `print("3.50" + 1)` y lee el traceback completo. Después arregla la línea.

### Reto — opcional

1. Escribe una sola expresión con `and` que sea `True` solo si el precio es mayor que 3 **y** la cantidad es par. Usa `%` para lo segundo y compruébala con dos combinaciones distintas.
2. Guarda en una variable el resultado de `not (precio > 3)`. Explícalo en un comentario, con tus palabras.

## ✅ Criterios de logro

- [ ] Tu programa corre y muestra cinco valores con su tipo
- [ ] Tienes cinco variables booleanas, una por comparación, y las imprimiste
- [ ] Puedes explicar la diferencia entre `=` y `==` sin mirar
- [ ] Sabes decir de qué tipo es `"3,50"` y por qué no se suma
- [ ] Leíste un traceback completo y encontraste la línea del error

## 🔗 Para profundizar

- [Python: tipos incorporados](https://docs.python.org/es/3/library/stdtypes.html) — la lista completa, para consultar
- [Tutorial de Python en español](https://docs.python.org/es/3/tutorial/introduction.html) — capítulo 3

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `SyntaxError: invalid syntax` | Falta un paréntesis, unas comillas o dos puntos | Mira la línea que marca y la anterior |
| `NameError: name 'precio' is not defined` | Usaste la variable antes de crearla, o está mal escrita | Definirla arriba; Python distingue mayúsculas |
| `TypeError` con `int` y `str` | Estás operando un número con un texto | `print(type(...))` y convertir con criterio |
| `3.503.50` en vez de `7.0` | El precio quedó como texto | Sacar las comillas o convertir con `float()` |
| No pasa nada al escribir `==` | `==` compara, no guarda | Para guardar va `=` |

## ➡️ Siguiente tema

**Condicionales:** con un booleano ya se puede decidir. Un `if` no hace más que mirar un `True` o un `False` y elegir un camino.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega las cinco variables con su tipo, las cinco comparaciones con su resultado, y el traceback del ejercicio 3 tal como te salió.

---
![[meme-dev-var.jpg]]