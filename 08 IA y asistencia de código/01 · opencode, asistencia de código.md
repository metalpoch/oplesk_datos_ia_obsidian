---
tipo: tema
modulo: "08 IA y asistencia de código"
capa: familiaridad
herramienta: opencode
resultado: "un PR con documentación escrita por el agente y verificada por ti"
tiempo-lectura: 18 min
prerrequisitos: ["Git y GitHub", "Python modular y tests", "Pandas, groupby y merge"]
tags: [ia, agentes, opencode]
---

# opencode: asistencia de código

![[opencode.svg|64]] ![[openrouter.svg|64]]

> [!tip] En una frase
> **opencode es un agente que vive en tu terminal: lee los archivos de tu carpeta, propone cambios y los ejecuta cuando se lo permites.** El modelo no cambia; lo que cambia es que ahora tiene acceso a tu proyecto.

> [!info] Lectura previa — 18 minutos
> Este tema cambia el método de trabajo. Todo lo anterior lo escribiste a mano; ahora el agente es parte del trabajo — y también la obligación de verificarlo. Cualquier punto que no quede claro se marca con `#duda`.

## 🤔 Para qué sirve

Tu pipeline funciona. El `README` está desactualizado, las funciones no tienen docstrings, y la revisión de tu propio código la vienes posponiendo desde los primeros temas.

Ese trabajo —documentar, explicar, revisar— es exactamente el que nadie hace y el que hace la diferencia cuando alguien más tiene que mantener tu proyecto. Y es el más caro de hacer a mano, porque no produce nada visible.

Un agente lo hace en minutos **si le das el contexto correcto y verificas lo que devuelve**. Eso es lo que se aprende en este tema: a usarlo para lo que sirve y a no creerle nada sin comprobarlo.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Abrir el agente en tu carpeta, hacer que lea tu código y explicar qué hace el proyecto |
| **Después** | Pedirle documentación con criterio de aceptación, leer el diff y comprobarla con `pytest` |

## 🧠 Qué es cada cosa

| Pieza | Función |
|---|---|
| **opencode** | El agente: lee archivos, propone y ejecuta |
| **El modelo** | El que genera la respuesta, a través de OpenRouter |
| **`plan`** | Modo de **solo lectura**: explora y propone, no edita |
| **`build`** | Modo con acceso completo: edita y ejecuta |
| **El diff** | Lo que el agente propone cambiar, línea por línea |

> [!note] Qué no es
> - **No es un chat.** No le pegas texto: está parado en tu carpeta y ve tus archivos.
> - **No verifica solo.** Dice "listo" igual cuando acertó que cuando no.
> - **No reemplaza tu criterio.** Si no entiendes lo que propone, no lo puedes aceptar.
> - **No escribe tu proyecto por ti.** Hace el trabajo mecánico; las decisiones siguen siendo tuyas.

## 🧠 Conceptos

1. **El agente tiene manos** — lee y ejecuta en la carpeta donde lo abres. Por eso la carpeta importa.
2. **`plan` primero, `build` después** — explorar sin riesgo, editar con red de seguridad.
3. **Lo que devuelve se comprueba ejecutando** — el test, la consulta, la cifra. Nunca leyendo.

## 📖 Teoría

### 1. Abrirlo en el lugar correcto

```bash
cd tienda
opencode
```

El agente ve **solamente** lo que hay en `tienda/`. Si lo abres en la carpeta de arriba, no encuentra tu código; si lo abres en `tienda/`, ve el repo completo.

Dentro de la interfaz, `Tab` cambia entre `plan` y `build`. La primera vez con un proyecto se usa `plan`.

### 2. El ciclo

```
   pedido
     ↓
  lee los archivos  ←───────────┐
     ↓                          │
  propone un cambio             │
     ↓                          │
  lo aplica (si está en build) ─┘
     ↓
  tú verificas ejecutando
```

La flecha que vuelve es la diferencia con un chat: el agente **ve el resultado** de lo que hizo y con eso decide el paso siguiente. Por eso puede corregirse, y por eso puede encadenar errores.

### 3. Documentar con el agente, y verificarlo

El pedido que funciona es el que tiene contrato:

```text
Documenta las funciones públicas de src/. Para cada una:
- Docstring en español, formato Google (Args / Returns / Raises).
- Un ejemplo de uso ejecutable, con datos del CSV del curso.
- Si la función no maneja errores, NO la documentes: repórtala aparte.
Salida: solo el bloque de código, sin explicación alrededor.
```

Y la verificación, que es lo que casi nadie hace:

```bash
python -m pytest        # los tests siguen verdes
git diff                # lees qué cambió, línea por línea
```

> [!warning] El paso que no se salta
> Un PR con documentación generada entra **después** de leer el diff y de correr los tests. Si el diff tiene cambios que no pediste, se rechaza. El agente no sabe qué es importante para ti.

## ⚠️ El error más común

> [!warning] El agente cita un archivo o una función que no existen
> ```
> Revisé src/validaciones.py y la función verificar_nulos() que pediste ya está implementada.
> ```
> **Qué significa:** `src/validaciones.py` no existe en tu repo. El agente completó con algo plausible porque no lo tenía en el contexto. **Cómo se lee:** si la respuesta menciona algo que no recuerdas, abre el archivo. Si no está, la respuesta entera pierde valor: no es un detalle, es un síntoma. **Cómo se arregla:** pedir que cite archivo y línea de cada afirmación, y verificar las que puedas. Lo que no se puede verificar, no se acepta.

## 🏋️ Ejercicios

### Base — obligatorio, 30 min

1. Abre opencode en la carpeta de tu proyecto, en modo `plan`. Pídele que te explique qué hace `src/` en cinco líneas. Comprueba que las menciones coincidan con los archivos reales.
2. En `build`, pídele los docstrings de dos funciones con el contrato de arriba. **Lee el diff completo** antes de aceptar.
3. Corre `pytest` después del cambio. Si algo se puso rojo, revierte con `git restore`.

### Reto — opcional

1. Pídele una mejora que **sepas** que rompe un test (por ejemplo, cambiar el parseo de fechas). Comprueba que el test lo detecta y escribe dos líneas sobre qué habría pasado sin la red de seguridad.
2. Compara el mismo pedido en `plan` y en `build`. Anota la diferencia en una línea.

## ✅ Criterios de logro

- [ ] Abriste el agente en la carpeta correcta y leyó tu propio código
- [ ] Usaste `plan` y `build`, y puedes explicar cuál no edita
- [ ] Leíste un diff completo antes de aceptar un cambio
- [ ] Los tests siguen verdes después del cambio
- [ ] Tienes anotado al menos un caso en el que el agente se equivocó o inventó algo

## 🔗 Para profundizar

- [opencode — documentación](https://opencode.ai/docs) — agentes, configuración y subagentes
- [OpenRouter — claves y crédito](https://openrouter.ai/docs) — dónde se ve el consumo
- Anexos: [[Agentes, skills]] · [[Agentes, subagentes]] — para cuando ya lo domines
- La nota del curso anterior sobre el tema: [[Chuleta de errores]]

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| No encuentra tus archivos | Abriste en otra carpeta | Abrir el agente dentro del proyecto |
| Propone cambios y no los aplica | Estás en `plan` | Cambiar a `build` con `Tab` |
| El cambio rompió un test | El agente no conocía esa restricción | `git restore` y repetir el pedido con el criterio explícito |
| Cita archivos que no existen | Completar un hueco del contexto | Pedir archivo y línea; verificar antes de aceptar |
| Sin crédito | Saldo en cero | Cargar en el panel de OpenRouter |

## ➡️ Siguiente tema

Le sigue **DuckDB**. El pipeline ya está entero y el agente te ayuda a mantenerlo. Ahora el dato crece y hace falta un motor que consulte archivos sin levantar un servidor.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega el diff que aceptaste (o el que rechazaste y por qué), el resultado de `pytest` después del cambio, y una línea con lo que el agente se inventó si te pasó.

---
![[opencode-image.jpg]]