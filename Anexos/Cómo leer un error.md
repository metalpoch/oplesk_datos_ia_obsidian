---
tipo: anexo
modulo: "Anexos"
capa: referencia
herramienta: "transversal"
resultado: "saber qué te está diciendo Python antes de tocar el código"
tiempo-lectura: 12 min
prerrequisitos: []
tags: [anexo, errores, depuracion]
---

# Cómo leer un error

> [!tip] En una frase
> **Un error no es un fracaso: es Python diciéndote exactamente en qué línea se detuvo.** El mensaje está escrito para ti, y se lee de abajo hacia arriba.

> [!info] Lectura previa — 12 minutos
> Esta página no enseña a programar. Enseña a leer el papelito que Python te deja cuando algo sale mal. Es la página que más veces vas a volver a abrir en todo el programa.

## 🤔 Para qué sirve

Escribes tres líneas, ejecutas, y la pantalla se llena de texto rojo. Lo primero que hace casi todo el mundo es **cambiar algo al azar** para ver si se va. A veces se va; casi siempre vuelve, o aparece otro peor.

El texto rojo es un **traceback**: la ruta que siguió Python hasta romperse. No es ruido. Tiene todo lo que necesitas para arreglarlo, en un orden muy específico.


## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Identificar en un traceback el archivo, la línea y el tipo de error |
| Después | Saber que la última línea es lo primero que se lee, y qué hacer después |
## 🧭 Qué no es

- **No es una calificación.** No dice que seas malo programando; dice que una línea hizo algo que no se puede hacer.
- **No es el final del programa, es el punto exacto donde se detuvo.** Antes de esa línea, todo funcionó.
- **No se arregla adivinando.** Cambiar cosas al azar hasta que desaparezca es cómo se aprende a romper programas.

## 📖 Las cuatro partes de un traceback

```text
Traceback (most recent call last):          ← 1. el encabezado
  File "ventas.py", line 4, in <module>     ← 2. dónde
    total = precio + cantidad               ← 3. la línea tal como la escribiste
TypeError: unsupported operand type(s)      ← 4. QUÉ pasó
for +: 'int' and 'str'
```

1. **`Traceback (most recent call last)`** — "el último es el más reciente". Siempre está.
2. **`File "ventas.py", line 4`** — el archivo y **el número de línea**. Es lo primero que abres.
3. **La línea copiada tal cual** — con una flecha `^^^^` debajo que señala la parte exacta que falló.
4. **La última línea** — el tipo de error y qué esperaba Python. **Es la que se lee primero.**

> [!tip] La regla que resuelve el 80 %
> **Lee la última línea primero.** Dice *qué* pasó. Después sube a la línea del archivo: dice *dónde*. Nunca al revés.

## 📖 Los cinco errores que vas a ver

| Última línea | Qué significa | Lo primero que revisas |
|---|---|---|
| `SyntaxError: expected ':'` | Falta un carácter obligatorio; el programa ni siquiera arrancó | La línea que marca: `if`, `for` y `def` terminan en `:` |
| `IndentationError: expected an indented block` | Falta la sangría del bloque | Las 4 espacios debajo de la línea anterior |
| `NameError: name 'x' is not defined` | Usaste un nombre que no existe | Si la variable está definida **antes**, y si está bien escrita |
| `TypeError: unsupported operand type(s) for +: 'int' and 'str'` | Operas un número con un texto | `print(type(variable))` en el valor que entra |
| `KeyError: 'bebidas'` | La clave no existe en el diccionario | Si la clave está escrita igual, y si primero hay que crearla |

Los tres primeros son de **forma**: el programa no llega a correr. Los dos últimos son de **dato**: el programa corre y se rompe con el valor que le diste.

## 📖 Cómo buscarlo

1. **Copia la última línea, no el traceback entero.** `TypeError: unsupported operand type(s) for +` encuentra respuesta; veinte líneas de ruta no.
2. **Quita lo que es tuyo.** Si el mensaje dice `ventas.py`, bórralo: no le sirve a nadie.
3. **Busca en inglés.** La documentación y las respuestas están en inglés, y `python typeerror int str` da resultados que `error de python` nunca va a dar.
4. **Pregunta al agente, pero pídele que explique, no que arregle.** "Explícame qué significa este error" enseña; "arréglalo" te deja sin aprender nada.

## 📖 Antes de pedir ayuda

Cuatro cosas, en este orden. Cuestan dos minutos y resuelven casi todo:

1. ¿En qué **línea** se detuvo? Ábrela.
2. ¿Qué dice la **última línea** del mensaje?
3. ¿Qué **tipo** tiene el valor con el que trabaja esa línea?
4. ¿Qué **cambié** desde la última vez que funcionó?

Si después de eso sigue trabado, pega **el mensaje completo** en el chat del curso. Un mensaje recortado no se puede diagnosticar: el dato que importa casi siempre está en la última línea.

## ⚠️ El error de los errores

> [!warning] Borrar la línea que falla para que el error desaparezca
> El programa corre, la pantalla queda limpia y **el resultado está mal**. Eso es peor que el error: un error se ve, un resultado mal calculado no.
>
> Un `except: pass` hace exactamente eso. Atrapa el problema y sigue como si nada.
>
> **Cómo se arregla:** entender qué dato llegó mal antes de decidir qué hacer con él.

## 🏋️ Ejercicios

### Base — 20 min

1. Escribe `print(precio)` sin haber definido `precio` y ejecuta. Copia la última línea, identifica el número de línea y el tipo de error.
2. Escribe un `if` sin los dos puntos al final. Ejecuta y compara: este error **no** trae traceback completo. ¿Por qué crees que es?
3. Escribe `print("3.50" + 1)` y lee el `TypeError`. ¿Cuál de los dos valores es el culpable? ¿Cómo lo sabes sin adivinar?

### Reto — opcional

1. Provoca un `KeyError` con un diccionario propio y arréglalo **sin** borrar la línea: creando la clave antes, o comprobando con `.get()`.
2. Toma un error cualquiera de la [[Chuleta de errores]] y explica en dos líneas qué te diría la última línea del mensaje antes de mirar la causa.

## ✅ Criterios de logro

- [ ] Identificas en un traceback el archivo, la línea y el tipo de error, sin ayuda
- [ ] Sabes que la última línea es lo primero que se lee
- [ ] Distingues un error de forma (`SyntaxError`) de uno de dato (`TypeError`, `KeyError`)
- [ ] Buscas un error sin pegar el traceback completo
- [ ] Puedes explicar por qué borrar la línea que falla es peor que el error

## 🔗 Para profundizar

- [Errores y excepciones — Python en español](https://docs.python.org/es/3/tutorial/errors.html) — la referencia completa
- [[Chuleta de errores]] — los mensajes concretos del programa, con su causa
- [[01 · Tu entorno de trabajo]] — cómo ejecutar el archivo y ver la salida
