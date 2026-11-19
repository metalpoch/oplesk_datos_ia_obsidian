---
tipo: tema
modulo: "02 Git y GitHub"
capa: dominio básico
herramienta: Git y GitHub
resultado: "repo en GitHub con README.md, .gitignore y un pull request mergeado a main"
tiempo-lectura: 18 min
prerrequisitos: ["Listas y diccionarios", "Python funciones, errores y asserts"]
tags: [tema, git, github, fundamentos]
---

# Git y GitHub: control de versiones desde cero

![[git.svg|64]] ![[github.svg|64]]

> [!tip] En una frase, antes de empezar
> **Git es un programa que guarda una foto de tu carpeta cada vez que se lo pedís, y te deja volver a cualquiera de esas fotos.** Nada más que eso. Todo lo demás son detalles de cómo se pide la foto y cómo se comparte.

> [!info] Lee esto antes de empezar — 18 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea. Esas dudas son el punto de partida.
>
> **Lo que NO está acá:** la solución del ejercicio. Se construye en la práctica, paso a paso; si ya viene resuelta, la nota no sirve.

## 🤔 Para qué sirve

Abre la carpeta donde tienes tus scripts y míralos. Si llevas algún tiempo programando, probablemente tengas algo así:

```
analisis.py
analisis_v2.py
analisis_final.py
analisis_final_bueno.py
analisis_final_bueno_ESTE.py
analisis_corregido_por_el_profe.py
```

Y ahora las preguntas incómodas:

- ¿Cuál de todos esos **funciona**? Ya no lo sabes.
- Querías probar algo nuevo, rompiste el que funcionaba y **no hay vuelta atrás**.
- ¿Qué cambiaste la última vez? Lo hiciste en el archivo, se perdió, y no te acuerdas.
- Se lo mandaste a alguien por WhatsApp. Ahora hay dos versiones y **nadie sabe cuál es la buena**.

**Eso es lo que Git resuelve.** No es una herramienta de gente avanzada: nace de ese problema, y lo resuelve en tres cosas:

1. **Guarda todas las versiones**, sin que tengas que inventar nombres de archivo.
2. **Te dice qué cambió, cuándo y por qué** — con tu propio mensaje, no con un nombre críptico.
3. **Te deja volver atrás** cuando rompes algo que funcionaba.

> [!tip] La analogía que sirve
> Es como los **puntos de guardado de un videojuego**: cuando la cosa funciona, guardas, y si después metes la pata puedes volver a ese punto. La diferencia es que Git además te cuenta qué pasó entre un punto y el otro.
>
> A diferencia de «Guardar» de Word: Word conserva **la versión actual** y descarta la anterior. Git conserva **todas**.

> [!note] Qué no es
> - **Git no es GitHub.** Git es el programa que corre en tu computadora y funciona sin conexión ni cuenta; GitHub es el sitio donde vive una copia. Puedes tener la historia completa y ningún remoto.
> - **No es un backup automático.** La foto existe cuando la pides con `commit`, y sale de tu máquina cuando haces `push`. Lo que no commiteaste no está en ninguna parte.
> - **No reemplaza la revisión del código.** `push` solo sube; el pull request existe para leer el diff antes de que entre a `main`, sea otro o seas tú al día siguiente. El merge es el último clic, nunca el primero.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Tener tu carpeta `tienda/` como repo, con un primer commit y un `.gitignore` que deja `__pycache__/` y `.env` fuera: `git status` ya no los nombra |
| **Después** | Abrir la rama `feature/limpieza`, subirla, leer el diff de un PR en GitHub línea por línea y mergearlo a `main` |

Hasta ahora tu código es un archivo suelto. En este tema gana **historia**: cada cambio con vuelta atrás, y una revisión antes de entrar a `main`.

## 🧠 Conceptos

1. **Un commit es una foto que se queda para siempre** — una marca en el tiempo con un mensaje que dice qué cambió. Es a lo que vuelves.
2. **Tu carpeta está en tu computadora; GitHub es una copia en internet** — Git trabaja en tu máquina, sin conexión. `push` sube, `pull` baja. `main` es el nombre de la línea principal.
3. **Una rama es un carril aparte, y el PR es la revisión antes de integrarla** — pruebas sin tocar lo que ya funciona; si sale bien se integra, si sale mal se tira.

## 📖 Teoría

### 1. El commit: la foto que se queda

Cuando Git está mirando una carpeta, esa carpeta pasa a llamarse **repositorio** (o "repo"). "Mirar" significa que existe una carpeta oculta llamada `.git` adentro: ahí van las fotos. **Si borras `.git`, borraste la historia** (tu código sigue ahí, pero sin versiones).

```bash
cd tienda             # entra a la carpeta que quieres versionar
git init              # le dice a Git: "empieza a mirar esta carpeta"
git add .             # elige qué archivos van a entrar en la foto
git commit -m "suma de ventas funcionando"
git log --oneline     # muestra las fotos que existen, una por línea
```

Léelo como una receta de tres pasos:

| Paso | Comando | Qué significa en lenguaje normal |
|---|---|---|
| 1 | `git init` | "Empieza a llevar la historia de esta carpeta" |
| 2 | `git add .` | "Prepárame todo lo que cambió para la próxima foto" |
| 3 | `git commit -m "..."` | "Toma la foto y ponle esta nota al dorso" |

El mensaje **no es decoración**: es lo único que vas a tener dentro de tres meses para saber por qué cambiaste algo. `"arreglos"` no sirve. `"valida filas antes de sumar: rompía con los montos en texto"` sí.

Y volver atrás:

```bash
git restore analisis.py    # devuelve el archivo a como estaba en la última foto
```

Eso es todo. Un punto de retorno, con nombre.

#### `git status`: tu único comando obligatorio

```bash
git status
```

Git te contesta tres cosas: en qué rama estás, qué archivos cambió y **no** están en la foto, y si hay algo esperando para entrar. Si te quedas con un solo comando de este tema, que sea este: **ante la duda, `git status`.** Te dice dónde estás sin que tengas que recordar nada.

### 2. Tu computadora y la copia en internet (Git no es GitHub)

Son dos cosas distintas, y casi todos las mezclan el primer día:

| | **Git** | **GitHub** |
|---|---|---|
| Qué es | un **programa** en tu computadora | un **sitio web** que guarda una copia |
| Necesita internet | **No** | Sí |
| Para qué | llevar la historia de tu carpeta | compartirla, respaldarla, trabajar en equipo |
| Quién lo hizo | la comunidad (software libre) | una empresa (Microsoft) |

La forma más simple de decirlo: **Git es el álbum de fotos; GitHub es el álbum en la nube que puedes compartir con un link.**

```
   TU COMPUTADORA                                GITHUB
 ┌──────────────────┐      git push      ┌──────────────────────┐
 │ tienda/          │  ───────────────▶  │ tu-usuario/tienda    │
 │ (repo local)     │                    │ (repo remoto)        │
 │  .git/           │  ◀───────────────  │                      │
 └──────────────────┘      git pull      └──────────────────────┘
```

- **`push`** = subir tus fotos nuevas al sitio web.
- **`pull`** = bajar las que hay allá y no tienes acá.
- **`origin`** = el apodo de esa dirección. Cuando escribes `origin` en un comando, te refieres a "mi repo remoto". Es un nombre convencional, no una palabra mágica.
- **`main`** = el nombre de la **línea principal** de fotos. Antes se llamaba `master`; en GitHub hoy es `main`.

Para conectar tu carpeta con el sitio web (una sola vez por proyecto):

```bash
git remote add origin https://github.com/tu-usuario/tienda.git
git branch -M main
git push -u origin main
```

La URL no te la inventas: te la da GitHub al crear el repo, en el botón verde **Code**.

#### `.gitignore`: lo que nunca sube

En la raíz de tu repo, un archivo de texto llamado `.gitignore` le dice a Git qué **no** mirar:

```text
__pycache__/
*.pyc
.env
```

- `__pycache__/` y `*.pyc` son cachés que Python genera solo: ensucian el diff y no aportan nada.
- **`.env` guarda contraseñas y tokens.** Este es el importante.

> [!warning] Un repo se hace público con un clic
> Si subiste el `.env` con una clave, **esa clave ya está filtrada**, aunque borres el archivo después: queda en la historia y en las copias que alguien ya descargó. Lo único que sirve es **rotarla** en el servicio (generar una nueva y anular la vieja).
>
> Si lo commiteaste por error: `git rm --cached .env`, agrega la línea al `.gitignore` y commitea de nuevo. Eso lo saca de las fotos futuras, no del pasado.

### 3. Una rama es un carril aparte, y el PR es la revisión

Hasta acá tienes una sola línea de fotos: `main`. **Una rama** es abrir una línea paralela para probar algo sin ensuciar la principal.

```bash
git switch -c feature/limpieza   # crea la rama y se mueve a ella
```

Piensa en una rama como **un carril aparte**: trabajas en `feature/limpieza`, y `main` sigue como estaba. Si el experimento sale bien, se integra. Si sale mal, borras el carril y no pasó nada. **Ese es el valor: puedes equivocarte sin costo.**

Cuando terminas:

```bash
git add .
git commit -m "limpieza: valida filas antes de sumar"
git push -u origin feature/limpieza
```

Ahora en GitHub aparece un botón **Compare & pull request**. Un **pull request** (PR) es una solicitud: *"estos cambios míos, ¿los integramos a `main`?"*. Se abre con título y dos líneas explicando qué cambia y por qué.

Y después viene la parte que casi nadie hace al principio: entras a **Files changed** y **lees el diff**. Cada línea con `+` es algo que entra; cada `-` es algo que sale. Verde y rojo.

> [!tip] Si prefieres no usar la terminal
> Todo esto —commit, rama, pull request, diff— también se hace con **GitHub Desktop**, una aplicación gráfica. En este curso no la usamos: el objetivo es entender qué pasa por debajo, y eso se ve escribiendo los comandos. Si más adelante te resulta más cómoda, la historia es exactamente la misma y no tienes que aprender nada nuevo.

## ⚠️ El error más común

> [!warning] `! [rejected] main -> main (fetch first)`
> ```
> To github.com:tu-usuario/tienda.git
>  ! [rejected]        main -> main (fetch first)
> error: failed to push some refs to 'github.com:tu-usuario/tienda.git'
> hint: Updates were rejected because the remote contains work that you do not
> hint: have locally. This is usually caused by another repository pushing to
> hint: the same ref. If you want to integrate the remote changes, use
> hint: 'git pull' before pushing again.
> hint: See the 'Note about fast-forwards' in 'git push --help' for details.
> ```
> **Qué significa:** la copia en internet (el remoto) tiene fotos que tu copia no tiene. Git no pisa trabajo ajeno: rechaza tu push en lugar de decidir por ti. Traducido: *"primero bájate lo que hay allá, después subes"*. **Cómo se lee:** el `! [rejected]` y el `error:` dicen que el push falló; los `hint:` dicen la causa (el remoto está adelante) y el remedio (`git pull`). El `main -> main` aclara en qué rama choca. **Cómo se arregla:** `git pull --rebase` y después `git push`. Si tocaste los mismos archivos, `git pull` te avisa: los conflictos **no se resuelven en este tema**, van a la guía asincrónica.

El otro mensaje frecuente es de GitHub, no de la terminal: **`There isn't anything to compare`** al abrir el PR. Casi siempre es que comparas `main` contra `main` y no hay commits nuevos de un lado. Se revisa en «Problemas frecuentes».

Este no es un error de adorno: es el que más se repite si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o en VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. **(8 min)** En `tienda/`: `git init`, crea el `.gitignore` con esas tres líneas, `git add .` y primer commit. Después `git status`: no tienen que aparecer ni `__pycache__` ni `.env`.
2. **(7 min)** Crea el repo en GitHub (`New repository`), copia la URL que te da, conéctalo con `git remote add origin <esa URL>` y sube con `git push -u origin main`. Agrega un `README.md` con tres cosas: qué hace tu script, cómo se corre y qué falta.
3. **(10 min)** Crea la rama, cambia algo chico (por ejemplo una función de conteo), commitea y sube la rama. Abre el PR y anota **tres cosas** que ves en el diff. El merge va después, cuando ya hayas leído el diff.

### Reto — opcional

1. Corrige el mensaje de tu último commit con `git commit --amend -m "..."` **antes** de pushear, y escribe en una línea por qué no se hace después.
2. Deja un comentario en una línea del diff (pasando el mouse aparece el `+`), como haría un revisor. Explica qué preguntarías antes de aprobar.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `git log --oneline` muestra al menos dos commits con mensajes que se entienden
- [ ] `git status` dice `working tree clean` y no lista ni `.env` ni `__pycache__`
- [ ] El repo en GitHub abre con el `README.md` renderizado y tu `total_ventas.py` adentro
- [ ] El PR figura como **Merged** en la pestaña Pull requests
- [ ] **Explicas en una frase la diferencia entre Git y GitHub**, sin mirar la tabla
- [ ] Explicas en una frase por qué no commiteas directo a `main`

## 🔗 Para profundizar

- [Pro Git, el libro oficial en español](https://git-scm.com/book/es/v2) — capítulos 1 (qué es Git, en serio) y 2 (lo básico); el resto lo vas a necesitar recién en el proyecto final
- [Documentación de `git switch`](https://git-scm.com/docs/git-switch) — crear y cambiar de rama sin `checkout`
- [GitHub: ignorar archivos](https://docs.github.com/es/get-started/getting-started-with-git/ignoring-files) — los patrones de `.gitignore` con ejemplos
- [GitHub: crear un pull request](https://docs.github.com/es/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) — el flujo completo con capturas
- El primer archivo que entra al repo es el del tema de Python básico: [[04 · Listas y diccionarios]]

## 🧭 Problemas frecuentes

- **`Author identity unknown`** → Git no sabe quién eres: `git config --global user.name "Tu Nombre"` y `git config --global user.email "tu@correo"`, y repite el commit.
- **`fatal: your current branch 'main' does not have any commits yet`** → no hay nada commiteado: te falta `git add .` y `git commit -m "..."`.
- **`There isn't anything to compare`** → estás parado en `main` o no hay commits nuevos: mira `git branch --show-current` y confirma el `git push` de la rama.
- **`error: src refspec main does not match any`** → tu rama local se llama `master` y le pediste pushear `main`. Renómbrala con `git branch -M main` y repite el push.
- **`remote origin already exists`** → ya lo habías conectado. Cambia la URL con `git remote set-url origin <URL nueva>`.
- **No entiendo nada de lo que me pide la terminal** → corre `git status`. Siempre. Es gratis y te dice dónde estás.

Si algo no te cierra y nada de esto lo resuelve, anota el mensaje completo en "Mi resultado".

## ➡️ Siguiente tema

Vas a ordenar el código en carpetas que se importan entre sí, leer el `.env` sin claves escritas en el archivo y hacer que un test te avise si algo se rompió.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la salida de `git log --oneline`, la URL de tu repo o el mensaje que no pudiste resolver.
