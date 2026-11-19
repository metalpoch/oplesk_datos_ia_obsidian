---
tipo: tema
modulo: "Proyecto y portafolio"
capa: dominio inicial
herramienta: "README + rúbrica de 10 criterios (ninguna herramienta nueva)"
resultado: "repo reproducible desde cero, rúbrica autoevaluada y los dos arreglos más caros aplicados"
tiempo-lectura: 15 min
prerrequisitos: ["Proyecto, integración"]
tags: [proyecto, reproducibilidad, revision-entre-pares]
---

# Reproducibilidad: que otro lo corra sin preguntarte nada

![[github.svg|64]]

> [!tip] En una frase, antes de empezar
> **Reproducible es una prueba, no una sensación**: alguien más clona tu repo, sigue tu README y llega a tu misma cifra sin hablarte.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea. Esas dudas marcan el punto de partida.
>
> **Lo que NO está acá:** el arreglo de tu repo ni el veredicto del repo de tu compañero. Eso se hace con el repo abierto y el cronómetro andando.

## 🤔 Para qué sirve

Funciona en tu máquina. Lo probaste diez veces, funciona perfecto.

Después alguien clona el repo, corre el primer comando y muere: falta una librería, la ruta era `C:\Users\kurbil\Descargas\ventas.csv`, el `.env` no estaba. **"A mí me funciona"** es la frase más cara de la programación, y al presentar el proyecto es la que decide la nota.

Eso es lo que este tema resuelve: que tu trabajo **pueda correr sin ti**. No es prolijidad ni una formalidad del curso; es la diferencia entre un proyecto y una demo.

> [!note] Qué no es
> - **No es "tener el código en GitHub".** Eso es respaldo. Reproducibilidad es que otro llegue al mismo resultado.
> - **No es un README lindo.** Es un README que alguien **siguió** y funcionó.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Auditar el repo de un compañero con la rúbrica de 10 criterios y devolver 3 hallazgos accionables, ordenados por costo |
| **Después** | Dejar tu repo corriendo desde cero siguiendo tu propio README, sin explicar nada de viva voz |

## 🧠 Conceptos

1. **Reproducible es una prueba, no una sensación** — un tercero llega al mismo número siguiendo el README, sin hablarte. Si necesitó preguntarte algo, no lo era.
2. **Los arreglos se ordenan por costo, no por facilidad** — primero lo que rompe el proyecto (credenciales, duplicados, cifras), después lo que lo embellece.
3. **Una revisión sirve si es específica** — "está bien, felicitaciones" no es una revisión: es un saludo. Hallazgo, evidencia, arreglo.

## 📖 Teoría

### Qué hace reproducible a un proyecto

Una sola prueba, y es la que vale: **alguien que no habló contigo clona el repo, sigue el README y llega a tu misma cifra.** Ni un mensaje, ni una llamada, ni "pásame el CSV". El README tiene que responder cuatro cosas, en este orden:
1. **Entorno**: qué versión de Python y qué instalar. `pip install -r requirements.txt` no alcanza si no dices la versión.
2. **Datos**: o el CSV está en el repo, o hay un comando que lo genera con la misma semilla.
3. **Un comando**: el que corre todo, de la fuente al dashboard. Si son cinco scripts, están los cinco y en orden.
4. **La cifra de control**: el número que el lector tiene que ver para saber que le salió igual que a ti.

> [!tip] La prueba de las 3 preguntas
> Contesta sin mirar el código: ¿qué comando corro?, ¿qué número tengo que ver?, ¿qué parte es manual? Si dudas en alguna, el README está incompleto.

### La rúbrica de 10 criterios: Sí o No, sin "más o menos"

| # | Criterio | Ejemplo de violación |
|---|---|---|
| 1 | **Un comando de punta a punta** | el README dice `python main.py`, pero antes hay que correr 4 scripts que no menciona |
| 2 | **Dependencias fijadas** | `pandas` sin versión; el día de la demo salió pandas 3 y cambió el dtype de texto |
| 3 | **Datos incluidos o generables** | el script lee `C:/Users/kurbil/Descargas/ventas.csv`, que no está en el repo |
| 4 | **Credenciales fuera del repo** | `DATABASE_URL = "postgresql://user:clave@ep-...neon.tech/db"` commiteado en `db.py` |
| 5 | **`.env.example` versionado** | hay `.env` en el repo y ningún ejemplo: nadie sabe qué variables necesita |
| 6 | **Carga idempotente** | correr el pipeline dos veces deja 100 k filas donde había 50 k |
| 7 | **Números que cuadran** | la app muestra 1,2 M de ingresos, Neon dice 800 k y nadie lo notó |
| 8 | **Secretos en CI, no en el YAML** | el token del bot escrito en `pipeline.yml`: queda en el log, en claro |
| 9 | **Rutas relativas** | `path = "/home/kurbil/proyecto/data"` en tres archivos distintos |
| 10 | **Límites declarados** | el README promete "pipeline completo" y tiene 3 pasos manuales que no figuran |

### Cómo hacer una revisión entre pares que sirva

Antes de opinar, **corre el repo del otro**. Una revisión sin haber ejecutado nada es una opinión sobre el código, y lo que se rompe no es el código: es la corrida. Cada hallazgo se escribe con la misma fórmula de tres partes:

> **Vi** `X` en `archivo.py` línea `N`. **Riesgo:** `lo que puede pasar`. **Arreglo:** `qué haría yo`.

| No sirve | Sirve |
|---|---|
| "Está muy bien, felicitaciones" | "Corrí el README y falla en el paso 2; el CSV no está en el repo" |
| "Revisa la seguridad" | "La cadena de Neon está en `db.py:12`; hay que rotarla, no solo borrar la línea" |
| "Faltan tests" | "Si corro dos veces, el conteo pasa de 50 k a 100 k: falta un delete antes del insert" |

Tres hallazgos, como máximo, del más caro al más barato. Diez observaciones sueltas no se arreglan; tres priorizadas sí.

### Los arreglos más caros primero

El orden no tiene que ver con cuánto tarda cada arreglo, sino con cuánto rompe:

1. **Credenciales expuestas.** Rota la clave **ya**; después limpia el repo. Borrar la línea no alcanza: el valor quedó en el historial de git y en cada clon.
2. **Carga no idempotente.** Es la que ensucia las cifras en silencio. Se arregla con `DELETE` + `INSERT` por partición o un `UPSERT` por clave, y se verifica corriendo dos veces y comparando el `count(*)`.
3. **Números que no cuadran.** El más caro de encontrar: no hay error, hay dos verdades. Se arregla con **una** cifra de control comparada en cada eslabón. Lo demás (README, `requirements.txt`, rutas) es barato y va después: si el pipeline duplica filas, un README perfecto solo documenta mejor el error.

## ⚠️ El error más común

> [!warning] `FileNotFoundError: [Errno 2] No such file or directory: 'C:\\Users\\kurbil\\Descargas\\ventas.csv'`
> ```
> FileNotFoundError: 'C:\\Users\\kurbil\\Descargas\\ventas.csv'
> ```
> **Qué significa:** hay una ruta absoluta escrita a mano. En tu máquina existe; en la de cualquiera que clone el repo, no. Delata que el proyecto nunca se corrió desde cero. **Cómo se lee:** la ruta lo dice todo. Si empieza por `C:\`, `/Users/` o `/home/`, no es del repo: es de alguien. Búscala con `grep -rn "Descargas" .` **Cómo se arregla:** rutas relativas al archivo, y los datos versionados o generados.
> ```python
> RAIZ = Path(__file__).resolve().parent.parent
> VENTAS = RAIZ / "data" / "ventas.csv"
> assert VENTAS.exists(), f"falta el dato: corre 'make datos' ({VENTAS})"
> ```

Este no es un error de adorno: es el que se lleva el tema si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, el tema completo

1. Aplica la rúbrica de 10 criterios a tu repo y anota el resultado en una tabla Sí/No.
2. Arregla **los dos más caros** de tu lista. No los más fáciles: los más caros.
3. Pídele a alguien del grupo que corra tu repo desde cero sin preguntarte nada. Anota cada pregunta que igual te hizo: cada una es un defecto del README.

### Reto — opcional, para quien quiera más

1. Reproduce el pipeline en una máquina limpia o en un contenedor y cronometra el tramo desde cero hasta la cifra de control.

## ✅ Criterios de logro

- [ ] Alguien que no habló contigo corrió tu repo y llegó a tu cifra de control
- [ ] Tu rúbrica de 10 criterios está completa, con los dos arreglos caros aplicados
- [ ] Correr el pipeline dos veces deja **el mismo** estado (conteo y cifras)

## 🔗 Para profundizar

- [GitHub — secretos en Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) — para el criterio 8
- Anexo del vault: [[Chuleta de errores]] — varias de esas entradas son defectos de reproducibilidad
- Nota anterior: [[01 · Proyecto, integración]]
- Nota siguiente: [[03 · Proyecto, defensa]]

## 🧭 Problemas frecuentes

- **"A mí me funciona"** → es el síntoma, no la defensa. Borra tu carpeta, clona de nuevo y corre el README al pie de la letra.
- **El repo del compañero no corre y no quieres ser duro** → reporta el criterio, no a la persona. "Falla el criterio 3" es objetivo; "no sabes empaquetar" no.
- **Arreglar la idempotencia te da miedo** → empieza con el `count(*)` antes y después de dos corridas. Si el número no cambia, ya está.

## ➡️ Siguiente tema

Vas a defender este mismo repo en 8 minutos: primero el problema, después la demo, y las decisiones con su alternativa descartada. Todo lo que arregles acá es lo que sostiene la defensa.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá tu rúbrica autoevaluada, el conteo de las dos corridas y la pregunta que te hizo quien reprodujo tu repo.
