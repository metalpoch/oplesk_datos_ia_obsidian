---
tipo: tema
modulo: "Anexos"
capa: familiaridad
herramienta: "un agente de código"
resultado: "bitácora de una propuesta rechazada, con el test que la desmiente"
tiempo-lectura: 17 min
prerrequisitos: ["Telegram, el bot de alertas"]
tags: [agentes, verificacion, ia]
---

# Agentes: el agente como compañero (y como sospechoso)

> [!tip] En una frase, antes de empezar
> **Un agente de código es un programa de IA al que le pides una tarea en tu proyecto y que trabaja en bucle con herramientas: lee archivos, corre comandos, mira el resultado y vuelve a intentar.** Tú revisas lo que hizo.

> [!info] Antes de empezar — 17 minutos
> Léelo completo y marca `#duda` donde no te cierre. **No está acá:** la solución del ejercicio; eso se construye al hacerlo.

## 🤔 Para qué sirve

Llevas dos horas con el mismo `UnicodeDecodeError` y no hay nadie despierto a quien preguntarle. Le pegas el error a un chat de IA, te devuelve un bloque de código, lo pegas en tu archivo y funciona —o arregla eso y rompe otra cosa tres pasos más adelante.

Copiar y pegar respuestas ya lo haces. El agente es lo mismo, pero con acceso a tus archivos: puede leer el script, correr el comando y ver el error por su cuenta.

**Eso es lo que resuelve, y por eso hay que cuidarlo:** te ahorra el ida y vuelta de copiar y pegar, y al mismo tiempo puede reescribir el archivo entero con total seguridad. Lo que no cambia es quién verifica: tú.

> [!note] Qué no es
> El autocompletado sugiere una línea y se detiene; el agente actúa. Y no es un compañero que sabe más que tú: no puede comprobar su propio trabajo ni te va a decir "esto no lo puedo verificar". Esa decisión es tuya y no se delega.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Pedirle un cambio al agente, ejecutarlo, y comprobar con `pytest` que su propuesta rompe algo |
| **Después** | Rehacer el pedido con restricciones y criterio de aceptación, y verificar que esta vez pasa |

Este es el primer tema del bloque de IA. Empieza por lo incómodo: el agente se equivoca, y sin verificación no te enteras.

## 🧠 Conceptos

1. **No es autocompletado: es un ciclo** — razona, usa una herramienta (leer un archivo, correr un comando), mira el resultado y vuelve a razonar. Por eso puede cambiar tu código y por eso puede romperlo rápido.
2. **Cuándo NO usarlo** — cuando no puedes verificar el resultado, cuando el dato es sensible, cuando el costo supera el beneficio. Tres condiciones, y basta que falle una.
3. **Verificar es ejecutar** — que el diff "se vea bien" no es evidencia. Un test que falla es la única prueba de que el agente se equivocó.

## 📖 Teoría

### 1. Qué es un agente

El autocompletado predice la próxima palabra y se detiene. Un agente tiene **herramientas** y las usa en bucle: lee el archivo, corre el test, ve el error, edita de nuevo.

```text
pedido → razona → llama herramienta → observa resultado → razona → ...
```

**Por qué importa:** el ciclo es lo que lo hace útil y lo que lo hace peligroso. Un autocompletado te sugiere una línea; un agente puede reescribir el archivo entero, y lo va a hacer con total seguridad.

### 2. Cuándo no usarlo

- **No puedes verificar el resultado** → si no sabes cómo comprobar que está bien, no delegues. Primero escribe la comprobación.
- **El dato es sensible** → nombres, cédulas, teléfonos, credenciales de la tienda no van al prompt. Ni "solo para probar".
- **El costo supera el beneficio** → si escribir el pedido con criterio te lleva más que hacerlo, hazlo tú.

**Por qué importa:** el agente nunca te va a decir "esto no lo puedo verificar". Esa decisión es tuya y no se delega.

### 3. Verificar por ejecución

```python
# tests/test_limpieza.py
from limpieza import limpiar

def test_fecha_dia_primero():
    fila = {"fecha": "03/04/2024", "total": "3,50"}
    assert limpiar(fila)["fecha"] == "2024-04-03"
```

Ese assert dice una cosa muy concreta: `03/04/2024` es **3 de abril**, no 4 de marzo. Si el agente "mejora" el parseo y lo interpreta al revés, el script sigue corriendo y el test se pone rojo. **Esa es la alarma.**

> [!tip] El curso ya te dio las tres piezas
> `assert` ([[01 · Python funciones, errores y asserts]]), `pytest` y el `.env` ([[02 · Python modular y tests]]), y el diff del PR que aprendiste a leer ([[01 · Git y GitHub]]). No necesitas nada nuevo para no ser víctima de esto: necesitas **usarlo antes de aceptar**.

## ⚠️ El error más común

> [!warning] Le pediste "optimiza este script" y `pytest` contesta esto
> ```text
> $ pytest -q
> F.F.                                                                    [100%]
> =========================== FAILURES ===========================
> ___________________ test_fecha_dia_primero _____________________
>     def test_fecha_dia_primero():
>         fila = {"fecha": "03/04/2024", "total": "3,50"}
> >       assert limpiar(fila)["fecha"] == "2024-04-03"
> E       AssertionError: assert '2024-03-04' == '2024-04-03'
> 2 failed, 2 passed in 0.19s
> ```
> **Qué significa:** el agente reescribió el parseo de fechas y ahora lee día/mes al revés. Dos tests en rojo, el mismo problema en dos filas distintas. El script **corre igual**: el error no aparece hasta que alguien lee el dashboard. **Cómo se lee:** el test te da las dos cosas: qué esperabas (`2024-04-03`) y qué salió (`2024-03-04`). Acá se recorta el segundo fallo; pytest los muestra todos. Sin test, estas líneas no existen y el bug viaja. **Cómo se arregla:** no parchees el test para que pase. Vuelve atrás el cambio (`git restore`), y rehaz el pedido con restricciones: "no toques el parseo de fechas; cambia solo X; criterio: estos 4 tests en verde". Recién ahí aceptas.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Pídele al agente "optimiza este script" sobre tu `limpiar.py`, sin darle más contexto. Guarda el diff.
2. Corre `pytest` y anota cuántos tests pasan y cuántos fallan. Si no tienes tests, ese es el hallazgo del día.
3. Rechaza la propuesta: vuelve al commit anterior y rehaz el pedido con **dos restricciones y un criterio de aceptación**. Guarda el segundo diff y compara qué tocó de más el primero.

### Reto — opcional

1. Escribe el test **antes** de pedir el cambio, muestra que falla, y después pídele al agente que lo haga pasar.
2. Mide el costo: minutos y cantidad de intentos. ¿Te ahorró tiempo o te lo comió? Anota el número.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Tu bitácora tiene el pedido, el diff y el nombre del test que falló
- [ ] Revertiste el cambio en vez de "arreglar" el test
- [ ] El segundo pedido incluye una restricción explícita y un criterio verificable
- [ ] Puedes explicar en una frase cuándo NO le darías tu dataset a un agente

## 🔗 Para profundizar

- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — el ciclo y los patrones, sin humo
- [Claude Code best practices](https://www.anthropic.com/engineering/claude-code-best-practices) — cómo pedir bien, con ejemplos de pedidos y restricciones
- [pytest: asserts](https://docs.pytest.org/en/stable/how-to/assert.html) — por qué el `assert` pelado alcanza y cómo se lee el fallo

## 🧭 Problemas frecuentes

- **"Me reescribió todo y le pedí un cambio chico"** → pídele un diff o un parche, revísalo antes de aplicar, y trabaja con el repo limpio (`git status`) para poder volver.
- **"Corre, pero el número cambió"** → no hay test que lo cubra. Ese es el problema, no el agente.
- **`NameError` o `ImportError` de algo que el agente nombró** → inventó una función o un import. Corre el archivo antes de creerle.
- **No puedo comprobar si está bien** → entonces no lo delegues. Escribe primero la comprobación; el agente después.

## ➡️ Siguiente tema

Vas a dejar de repetir el mismo pedido: escribes una instrucción reusable —una skill— con formato y criterio de aceptación, y la usas dos veces → [[Agentes, skills]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la bitácora: el pedido, el diff que rechazaste y el test que falló. En el siguiente tema arrancamos por lo que quedó acá.
