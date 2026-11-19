---
tipo: tema
modulo: "Proyecto y portafolio"
capa: dominio inicial
herramienta: "la demo y el guion (ninguna herramienta nueva)"
resultado: "demo de 8 minutos cronometrada, slides de apoyo y links al repo y a la app publicada"
tiempo-lectura: 12 min
prerrequisitos: ["Proyecto, reproducibilidad"]
tags: [proyecto, defensa, comunicacion]
---

# La defensa: 8 minutos, una demo y dos decisiones bien contadas

![[python.svg|64]]

> [!tip] En una frase, antes de empezar
> **La defensa es tu demo del proyecto frente a otros: 8 minutos para mostrar que corre, contar dos decisiones con su costo y decir qué no hace.** No es un examen oral de teoría.

> [!info] Lee esto antes de empezar — 12 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea. Esas dudas marcan el punto de partida.
>
> **Lo que NO está acá:** tu presentación armada ni tus diapositivas. Acá está el guion, el criterio y las preguntas. El ensayo se hace cronometrado.

## 🤔 Para qué sirve

Tienes el pipeline funcionando y llegas con las diapositivas llenas de tecnologías. A los cuarenta segundos el jurado ya no sabe qué problema resolviste; a los seis minutos vas por la mitad y la demo todavía no apareció. Te preguntan de dónde salió una cifra y no la tienes a mano.

Lo difícil no fue el código: fue contarlo. Y sin eso, un proyecto que funciona se lee como un trabajo que no sabes explicar.

**Eso es lo que resuelve la defensa:** un guion fijo —problema, datos, demo, decisiones, límites— que se ensaya con cronómetro, y evidencia abierta en pestañas para las tres preguntas que te van a hacer.

> [!note] Qué no es
> No es un examen de definiciones ni una tesis: nadie te va a preguntar qué es un `JOIN`. Y no gana quien promete más: decir qué no hace y qué te costó cada decisión es lo que te hace creíble.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Presentar tu demo completa en 8 minutos cronometrados, con dos rondas y devolución de un compañero |
| **Después** | Defender dos decisiones técnicas con su alternativa descartada y su costo, y responder las 3 preguntas del jurado sin improvisar |

## 🧠 Conceptos

1. **La demo manda** — el jurado recuerda lo que vio funcionar, no lo que le contaste. Si no se vio, no pasó.
2. **Una decisión se defiende con lo que descartaste** — sin alternativa y sin costo, no es una decisión: es una preferencia.
3. **Los límites declarados suman** — decir qué no hace te hace creíble. Que lo descubra el jurado te lo resta.

## 📖 Teoría

### El guion de 8 minutos, minuto por minuto

El orden no es libre: problema → datos → qué hiciste → demo → decisiones → límites. La demo va en el medio, cuando el jurado ya sabe qué está mirando y todavía no se cansó.

| Minuto | Qué muestras | Qué dices |
|---|---|---|
| **0:00–0:45** | Nada, o el título con una cifra grande | "En la tienda de la esquina se venden X al día y nadie sabe qué se vende ni cuándo. Esto es lo que armé para responderlo." |
| **0:45–1:30** | El CSV crudo, 10 segundos, con las filas sucias a la vista | "50 mil filas con fechas en dos formatos, precios con coma y la misma categoría escrita de cuatro maneras." |
| **1:30–3:00** | El diagrama de los 8 eslabones | "De la libreta al dashboard en menos de 3 minutos: limpieza, Parquet, base, alerta y app." |
| **3:00–6:00** | **La demo**: app en incógnito, un run de Actions, el mensaje de Telegram | "Cambio el filtro y mira cómo se mueve el número. Este run trae el dato de ayer y acá llega la alerta." |
| **6:00–7:15** | Una tarjeta con dos decisiones, tres líneas cada una | "Usé DuckDB para verificar y Neon para servir. Descarté X porque Y, y me costó Z." |
| **7:15–7:45** | La lista de límites, escrita | "No tiene login, es de un solo usuario, el plan free duerme y el cron puede demorar." |
| **7:45–8:00** | El repo y la URL, en pantalla | "Está todo acá: el README corre de cero y la app está publicada." |

Los tres minutos de demo son intocables. Todo lo demás se recorta antes que eso.

### Cómo se defiende una decisión técnica

Cuatro campos, siempre los mismos, y en ese orden:

> **Decisión:** qué elegiste, en una frase. **Alternativa:** la otra opción real que descartaste, no un espantapájaros. **Por qué:** el criterio que decidió (costo, tiempo, la restricción real, lo que sabías). **Qué te costó:** el precio de haber elegido eso.

El cuarto campo es el que casi todos se saltean, y es el que más credibilidad da. Ejemplo, con el stack del curso:

> **Decisión:** Neon como base de la app, en vez de PostgreSQL local. **Alternativa:** un Postgres en Docker en mi máquina, que era más rápido y gratis. **Por qué:** el dashboard se publica en Streamlit Cloud y tiene que leer desde la nube, no desde mi laptop. **Qué me costó:** el arranque en frío: la primera consulta tarda 8 segundos, así que cacheé con `ttl` y caliento la base antes de la demo.

Si no puedes nombrar el costo, probablemente no hayas decidido nada: te tocó.

### Las 3 preguntas que el jurado va a hacer

No son trampas. Son las tres formas de preguntar "¿esto lo hiciste tú y sabes lo que hace?".

1. **"¿Por qué esta herramienta y no la otra?"** → contestas con los cuatro campos, no con una lista de ventajas. Si la única razón es "era la del curso", dilo: es honesto y te ahorra una pregunta peor.
2. **"¿Cómo sé que las cifras son correctas?"** → muestras la cifra de control repetida en DuckDB, Neon y la app, y el paso del pipeline que la compara. La evidencia está en el repo; tenla a mano.
3. **"¿Qué pasa si el pipeline falla un domingo a las 3 de la mañana?"** → muestras la alerta de Telegram, el run rojo provocado a propósito y el `assert` que cortó. Prepara la captura: en vivo no vas a poder romperlo sin arruinar la demo.

Preparar la respuesta es preparar la **evidencia**, no el discurso: las tres se contestan abriendo algo.

> [!important] Y una cuarta que ya está en la política del curso
> "¿Qué hizo la IA y cómo lo verificaste?" No es hostil: está en [[01 · Proyecto, integración]]. Ten a mano el caso donde el agente se equivocó y cómo lo detectaste.

## ⚠️ El error más común

> [!warning] Arrancar por el stack: "Buenas, mi proyecto usa Python 3.12, pandas, DuckDB, Parquet, Neon, Actions, Telegram y Streamlit…"
> **Qué significa:** empezaste por las herramientas porque es lo que más trabajaste. Es comprensible y es el error más común de la cohorte. **Cómo se lee (desde el jurado):** una lista de compras. A los 40 segundos ya no sabe qué problema resuelves, y el resto de la presentación lo escucha comparando con la lista. **Cómo se arregla:** los primeros 45 segundos son del problema y de la cifra. El stack aparece recién cuando una decisión lo necesita, y ahí sí suma: "usé DuckDB porque necesitaba SQL sobre archivos sin servidor".
> ```
> Mal:  "Usé Python, pandas, DuckDB, Parquet, Neon, Actions, Telegram y Streamlit."
> Bien: "La tienda no sabe qué se vende ni cuándo. Armé un pipeline que se actualiza
>        solo cada día y avisa si las ventas caen. Para eso usé DuckDB y Neon."
> ```

Este no es un error de adorno: es el que se lleva la defensa si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, el tema completo

1. Ensaya el guion completo con cronómetro, grabándote. Anota en qué minuto estabas al terminar cada bloque: ahí se ve qué se te fue.
2. Escribe tus dos decisiones con los cuatro campos, en una tarjeta que puedas leer.
3. Prepara las 3 respuestas **con su evidencia** abierta en pestañas: la app, el run rojo y la consulta de la cifra de control.

### Reto — opcional, para quien quiera más

1. Graba un video de respaldo de la demo de 3 minutos, por si la red falla en vivo.
2. Responde las 3 preguntas del jurado en 90 segundos cada una, en voz alta, sin leer.

## ✅ Criterios de logro

- [ ] La demo completa entra en 8 minutos, cronometrada, y la demo dura 3
- [ ] Los primeros 45 segundos hablan del problema, no del stack
- [ ] Hay dos decisiones escritas con alternativa descartada y costo
- [ ] Las 3 respuestas del jurado se contestan abriendo una pestaña, no recitando

## 🔗 Para profundizar

- [Streamlit — conceptos de caché](https://docs.streamlit.io/develop/concepts/architecture/caching) — para cuando te pregunten por el arranque en frío
- Nota anterior: [[02 · Proyecto, reproducibilidad]]
- Anexo: [[Chuleta de errores]]

## 🧭 Problemas frecuentes

- **Te fuiste a 11 minutos** → no hables más rápido: corta. Primero el bloque de "qué hiciste", después un límite, después una decisión. La demo no se toca.
- **La demo falla en vivo** → sigue con la captura o el video. Dilo en una frase ("esto ya lo corrí, acá está la salida"), no pidas disculpas tres veces.
- **No te acuerdas una cifra** → ten una hoja de una página con las 5 cifras que importan. Mirarla no resta; inventarla, sí.
- **El jurado pregunta algo que no sabes** → "no lo medí, lo haría así". Es una respuesta completa; improvisar un número no.

## ➡️ Siguiente tema

No hay tema siguiente: esto es el cierre. Lo que sigue lo eliges tú, y el mapa está en los anexos del vault: [[Polars en 25 minutos]] y [[Equivalencias Pandas y Polars]] si quieres seguir por el lado de los motores.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá el tiempo real de tu ensayo, tus dos decisiones en cuatro campos y la pregunta del jurado que peor te salió. Empieza por esa.
