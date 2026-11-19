---
tipo: guia
modulo: "00 Antes de empezar"
tiempo-lectura: 20 min
tags: [guia, obsidian, herramientas]
---

# Guía de Obsidian: por qué tu cuaderno es una herramienta de trabajo

![[obsidian.svg|64]]

> [!tip] En una frase, antes de empezar
> **Obsidian es un programa que abre una carpeta con tus notas en archivos de texto y te deja conectarlas entre sí con enlaces.** Esas notas son archivos `.md` en tu disco: el programa las muestra bonito, las busca todas a la vez y te dice qué conecta con qué.

> [!info] Esto no es relleno del curso
> Vas a pasar los próximos meses acumulando decisiones: qué filtraste, qué descartaste, por qué usaste una herramienta y no otra. Si eso vive en tu cabeza, se pierde. Si vive en un cuaderno con enlaces, se convierte en tu portafolio y en tu memoria.
>
> Obsidian es gratis, funciona sin internet y los archivos son **tuyos** para siempre.

## 🤔 Para qué sirve

Repasa dónde están hoy las cosas que aprendes: una foto del pizarrón en el carrete del celular, un audio de WhatsApp, dos cuadernos a medio usar, una pestaña que cerraste sin leer y una hoja de cálculo que abriste una vez. Nada de eso está roto. El problema es otro:

- Querías recordar **por qué** filtraste esos 40 registros y no otros. Lo pensaste, no lo anotaste, y hoy no lo sabes.
- Buscas "esa nota que explicaba el `GROUP BY`". Está en algún lado de tu computadora y no la   encuentras en veinte minutos.
- Un compañero pregunta cómo resolviste el error de ayer. Lo resolviste, no quedó escrito, y lo vuelves a resolver desde cero.

**Eso es lo que resuelve Obsidian.** No te hace más inteligente: **guarda lo que ya pensaste**, en archivos que no dependen de ninguna empresa. Un **vault** es solo la carpeta con esas notas; Obsidian la abre, la busca entera y convierte cada `[[nombre de nota]]` en un enlace que funciona desde cualquier lado.

> [!tip] Con qué se lo confunde
> Con **la app de notas del teléfono**, que guarda texto y ya. La diferencia no es la cantidad de funciones: es que acá cada nota es un archivo `.md` tuyo, y se puede enlazar con otra.
>
> Y no hace falta el vault para todo: un documento suelto que vas a imprimir y olvidar se escribe en cualquier lado. El vault es para lo que vas a volver a buscar.


## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Moverte por el material y encontrar un tema en dos clics |
| Después | Marcar una duda y volver a encontrarla cuando la resuelvas |
## 🧩 Por qué Obsidian y no un Google Doc

Un documento es una línea: escribes arriba, lees abajo. Un vault es una **red**: cada cosa que aprendes se conecta con las otras. La diferencia se nota cuando quieres recordar qué decidiste al principio sobre el formato de los datos.

Cuatro razones concretas:

1. **Los archivos son Markdown en tu disco.** No hay formato propietario ni cuenta que se pueda cerrar. En cinco años los abres con cualquier editor.
2. **Los enlaces son la función, no un adorno.** `[[03 · SQL, JOIN y granularidad]]` te lleva a esa nota desde cualquier lado, y la nota sabe quién la menciona.
3. **Funciona sin internet.** Importante cuando la conexión falla justo el día que estudias.
4. **Te sirve fuera del curso.** Un vault de decisiones técnicas es lo que un equipo espera de alguien senior. Empiezas a construirlo hoy, con material real.

## 🎓 Lo que tienes que aprender (5 cosas, 15 minutos)

### 1. Un vault es una carpeta

Este vault es una carpeta con archivos `.md` adentro. Nada más. Puedes abrirla con el Explorador de archivos de tu sistema y ver todo. Obsidian solo la muestra bonito.

### 2. Enlaces: `[[así]]`

Escribiendo `[[` te aparece un buscador de notas. Eliges una y queda el enlace. El enlace **funciona por el nombre del archivo**, no por la carpeta.

> [!tip] El atajo que más vas a usar
> `Ctrl+O` (o `Cmd+O`) abre el buscador rápido. Escribes tres letras del nombre y entras. Esto reemplaza andar buscando archivos en el explorador.

### 3. Callouts: las cajas de color

```markdown
> [!warning] Cuidado con esto
> Acá va el texto.

> [!tip] Un atajo
> Acá va el texto.

> [!info] Contexto
> Acá va el texto.
```

Hay más tipos: `note`, `important`, `question`, `example`. En este curso usamos `> [!warning]` para los errores y `> [!info]` para los avisos de lectura.

### 4. Propiedades: los datos de la nota

Arriba de cada nota hay un bloque entre `---` con campos:

```yaml
---
capa: familiaridad
herramienta: Streamlit
resultado: URL pública
---
```

Eso no es decoración: es información que se puede buscar y filtrar. En el trabajo real, "¿qué resultados tengo pendientes?" se responde filtrando, no leyendo.

### 5. Buscar es la habilidad

`Ctrl+Shift+F` busca en **todo** el vault. Buscar `#duda` te devuelve todas las líneas donde marcaste una duda. Empieza a usar esto desde el principio: es la diferencia entre releer todo y encontrar lo que buscabas.

## 🗓️ Cómo vamos a usar el vault en este curso

| Momento | Qué haces acá |
|---|---|
| **Antes de empezar** | Lees la nota del tema (10-20 min). Marcas `#duda` |
| **Después** | Haces los ejercicios en tu terminal y pegas el resultado en "Mi resultado" |
| **Cuando algo falla** | Buscas el error en [[Chuleta de errores]] |
| **Al final del curso** | Tu vault es el índice del proyecto: cada decisión técnica con su por qué |

> [!warning] El error que casi todos cometen
> Leer la nota como si fuera un libro, de corrido y sin marcar nada. Cuando vuelven a la nota, no se acuerdan de nada y creen que "no les da la cabeza". La nota no se lee: se **interroga**. Cada vez que dice algo, pregúntate "¿esto lo podría hacer yo ahora?". Si la respuesta es no, marca `#duda`.

## 📦 Instalación

1. Descarga Obsidian de [obsidian.md](https://obsidian.md/) (Windows, macOS o Linux; gratis).
2. Al abrirlo, elige **"Abrir carpeta como vault"** y selecciona la carpeta de este curso.
3. Listo. No hace falta configurar nada: el vault viene armado.

**En el celular:** la app de Obsidian es gratis, pero sincronizar tiene costo. Para este curso no hace falta: alcanza con la computadora. Si quieres leer en el teléfono, sincroniza la carpeta con tu método de siempre (Drive, Syncthing, git).

## 🔌 Plugins: lo que recomendamos y lo que NO

**Viene funcionando con lo que trae.** Eso es suficiente para el curso.

Si quieres ir un poco más (opcional, una vez que domines lo básico):

- **Templates** (ya viene activo) — para crear notas con la plantilla del vault.
- **Calendar** — ver qué escribiste cada día.
- **Dataview** — consultas sobre tus propias notas. Potente, pero no lo necesitas para el curso.

> [!warning] No instales plugins al principio
> Cada plugin nuevo es una forma de romper el vault o de distraerte del contenido. Primero aprende a enlazar, buscar y marcar dudas. Después, si te hace falta, agregas.

## 🌍 Obsidian fuera del curso

Esto es lo que te va a servir en un trabajo, más allá de la analítica:

- Un vault por proyecto, con una nota de decisiones: qué se probó, qué se descartó y por qué.
- Una nota por reunión, enlazada a las personas y a los temas.
- Un "diario de bugs": qué rompió, qué se probó, qué funcionó. La segunda vez que falle lo encuentras en 30 segundos.

Nada de eso necesita que seas experto en Obsidian. Necesita que escribas diez líneas por día.

## 🔗 Para profundizar

- [Ayuda oficial de Obsidian](https://help.obsidian.md/) — corta y en español
- [Sintaxis de Markdown](https://help.obsidian.md/Editing+and+formatting/Basic+formatting+syntax)
- Siguiente paso del curso: [[01 · Tu entorno de trabajo]]
---
![[meme-obsidian.jpeg]]]