---
tipo: tema
modulo: "Anexos"
capa: familiaridad
herramienta: "skills (instrucciones reusables)"
resultado: "una skill escrita y usada dos veces en tareas distintas, con salida consistente"
tiempo-lectura: 16 min
prerrequisitos: ["Agentes, el agente como compañero"]
tags: [agentes, skills, ia]
---

# Agentes: skills, la instrucción que no se repite

> [!tip] En una frase, antes de empezar
> Una **skill** es una instrucción guardada, con formato y criterio de aceptación: **no es un mensaje de chat**. Se escribe una vez y el agente la aplica igual todas las veces.

> [!info] Antes de empezar — 16 minutos
> Léelo completo y marca `#duda` donde no te cierre. **No está acá:** la solución del ejercicio; eso se construye al hacerlo.

## 🤔 Para qué sirve

Le pides al agente "documenta estas funciones". Sale bien. Al día siguiente lo pides de nuevo y sale **distinto**: otro formato, otro idioma, a veces con ejemplos y a veces sin ellos.

El problema no es el agente: es que **pediste algo que no tiene contrato**. Si no defines qué significa "bien", no hay forma de saber si lo hizo bien ni de repetir el resultado.

Eso es lo que una skill resuelve: convertir un pedido que se interpreta en una **instrucción que se cumple o no se cumple**, y que puedes mirar y decir "esto está bien" sin discutir.

> [!note] Qué no es
> - **No es un prompt más largo.** El largo no es el punto: el punto es el criterio de aceptación.
> - **No es la memoria del chat.** Una skill vive en un archivo, la escribes tú y la revisas tú.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Escribir una skill con formato de salida y criterio de aceptación, guardada donde el agente la lee |
| **Después** | Usarla dos veces en tareas distintas y comprobar que la salida tiene la misma forma |

En el tema anterior el pedido fue uno. En este tema el pedido se escribe una vez y se usa siempre.

## 🧠 Conceptos

1. **Una skill es una instrucción reusable** — un archivo con formato y criterio, guardado donde el agente lo pueda leer. No un mensaje de chat: un artefacto que puedes versionar y revisar.
2. **El criterio de aceptación es la parte que casi todos olvidan** — sin "cómo sé que está bien", la salida puede ser correcta o basura y no tienes cómo distinguirlo.
3. **Contexto acotado** — dale lo que necesita para esa tarea, no el repo entero. Más contexto no es mejor contexto.

> Una función documentada es para el humano lo que una skill es para el agente: **el mismo contrato, distinto lector.** Qué recibe, qué devuelve, qué falla.

## 📖 Teoría

### 1. Qué es una skill

Un archivo de texto, en el repo, con instrucciones estables. Lo lee el agente antes de trabajar, y lo lee cualquiera del equipo cuando se pregunta por qué la salida siempre tiene la misma forma.

````markdown
---
name: documentar-funcion
description: Documenta una función de Python del proyecto ventas en 4 secciones fijas.
---

# Cuándo usarla
Cuando agregas o cambias una función en `src/`.

# Formato de salida
1. Qué recibe (tipos copiados de la firma real)
2. Qué devuelve
3. Qué puede fallar
4. Un ejemplo que se pueda pegar y correr

# Criterio de aceptación
- La firma coincide con `inspect.signature`, carácter por carácter.
- Si un dato no está en el código, escribe FALTA. No lo inventes.
- El ejemplo corre en la terminal sin editar nada.
````

El nombre del archivo y la ruta dependen de la herramienta; el contenido no. **La forma exacta la resuelve el agente que uses; el contrato lo pones tú.**

### 2. El criterio de aceptación

Compara los dos pedidos:

- "Documenta este código" → texto lindo, formato distinto cada vez, y nadie puede decir si está bien.
- "Devuelve estas 4 secciones; si falta un dato escribe FALTA; el ejemplo tiene que correr" → algo que se revisa en 20 segundos.

**Por qué importa:** el criterio no es burocracia. Es lo que te deja **rechazar** una salida sin discutir con el agente, y lo que hace que dos corridas se puedan comparar.

### 3. Contexto acotado

Si le pasas el repo entero, el agente elige qué mirar sin que tú sepas qué eligió. Si le pasas el archivo y la función, sabes exactamente sobre qué trabajó. **Acotar el contexto es lo que hace verificable la respuesta.**

## ⚠️ El error más común

> [!warning] `TypeError: limpiar_ventas() got an unexpected keyword argument 'fecha_col'`
> ```text
> Traceback (most recent call last):
>   File "<stdin>", line 1, in <module>
> TypeError: limpiar_ventas() got an unexpected keyword argument 'fecha_col'
> ```
> **Qué significa:** el ejemplo de la documentación usa un parámetro que la función no tiene. La instrucción vaga produjo un documento que **describe una función que no existe**. **Cómo se lee:** pega el ejemplo en la terminal y córrelo. Si no corre, la documentación miente; no lo leas, ejecútalo. **Cómo se arregla:** agrega al criterio "la firma se copia de `inspect.signature`, no de memoria" y "si falta un dato, FALTA". Vuelve a generarlo y corre el ejemplo otra vez.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Escribe tu skill: cuándo usarla, formato de salida, criterio de aceptación y un ejemplo de entrada y salida. Guárdala en el repo.
2. Úsala dos veces en tareas distintas (documentar `limpiar()` y documentar `cargar()`) y compara las salidas: ¿tienen la misma forma?
3. Rompe a propósito: quítale el criterio de aceptación y pide lo mismo de nuevo. Anota la diferencia entre las dos salidas.

### Reto — opcional

1. Agrégale a la skill un ejemplo de mala salida ("no hagas esto, y por qué"), y comprueba si mejora.
2. Escribe una segunda skill para otra tarea repetida (el mensaje de commit, el aviso de Telegram) y verifica que las dos no se pisen.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] La skill existe como archivo en el repo, con formato y criterio de aceptación
- [ ] La usaste dos veces y las dos salidas se pueden comparar lado a lado
- [ ] Puedes señalar qué línea del criterio atajó un error real
- [ ] Puedes explicar en una frase por qué la instrucción vaga no alcanza

## 🔗 Para profundizar

- [Agent Skills](https://code.claude.com/docs/en/skills) — el formato de referencia, con frontmatter, rutas y alcance
- [Agent Skills: el anuncio](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — por qué se estandarizó como archivo y no como prompt
- [Claude Code best practices](https://www.anthropic.com/engineering/claude-code-best-practices) — pedidos con contexto y criterio, con ejemplos

## 🧭 Problemas frecuentes

- **La skill no se aplica nunca** → está en una ruta que el agente no lee, o el `description` no dice cuándo usarla. El `description` es el disparador, no un adorno.
- **Se aplica a todo** → hiciste la skill demasiado amplia. Achica el alcance: una tarea, un formato.
- **La salida cambia en cada corrida** → falta formato explícito. Si la salida no se puede comparar, el criterio no está escrito.
- **El ejemplo pesa más que la regla** → el agente imita tu ejemplo aunque diga otra cosa. Pon un ejemplo **bueno** y describe el malo en texto.

## ➡️ Siguiente tema

Vas a delegar una tarea completa a un subagente, y a verificar tres de sus afirmaciones ejecutando código. Cierra el bloque → [[Agentes, subagentes]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la skill, las dos salidas y la diferencia que apareció cuando le sacaste el criterio de aceptación. En el siguiente tema arrancamos por lo que quedó acá.
