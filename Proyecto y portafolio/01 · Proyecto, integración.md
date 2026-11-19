---
tipo: tema
modulo: "Proyecto y portafolio"
capa: dominio inicial
herramienta: "el pipeline completo (ninguna herramienta nueva)"
resultado: "integrity.md con el checklist de 8 puntos, el link del run y la captura de la app con el dato nuevo"
tiempo-lectura: 12 min
prerrequisitos: []
tags: [proyecto, integracion]
---

# Proyecto: no se aprende nada nuevo, se hace funcionar lo viejo

![[github.svg|64]]

> [!tip] En una frase, antes de empezar
> **Un pipeline es el recorrido completo del dato, de la fuente al dashboard, en scripts que corren solos: cada paso recibe lo que dejó el anterior y lo deja listo para el siguiente.**

> [!info] Lee esto antes de empezar — 12 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea. Esas dudas marcan el punto de partida.
>
> **Lo que NO está acá:** tu pipeline resuelto. Acá está el orden, la verificación y el criterio para recortar.

## 🤔 Para qué sirve

Cada pieza tuya funciona por separado. El script de limpieza corre, el Parquet se escribe, la base recibe las filas, la app muestra el número. Las probaste todas, una por una, en días distintos.

Después las juntas, y algo se rompe: el Parquet es de la corrida vieja, la columna cambió de nombre y la base sigue pidiendo el nombre anterior, la alerta no llega porque el pipeline murió dos pasos antes. Y el día de la demo estás abriendo una terminal, después otra, y la app muestra el número del día anterior mientras explicas.

**Eso es lo que resuelve este tema:** unir los eslabones y demostrar que la cadena entera corre en una sola pasada, de principio a fin, con una cifra de control en cada unión.

> [!note] Qué no es
> Una lista se corre a mano y en el orden que recuerdes; un pipeline deja un número que el paso siguiente comprueba. Y no se arregla agregando tecnología: cuando algo no llega, se recorta el alcance y se declara qué se recortó.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Correr el pipeline completo y llenar el checklist de 8 puntos con 8 verdes en la misma corrida |
| **Después** | Romper un eslabón a propósito y ubicar en menos de 2 minutos dónde se cortó la cadena, leyendo el primer paso rojo |

## 🧠 Conceptos

1. **El pipeline es una cadena, no una lista** — cada eslabón recibe lo que dejó el anterior. Vale lo que vale el más débil, y casi nunca es el que imaginas.
2. **Verde no es lo mismo que correcto** — "no tiró error" es el piso. Cada eslabón tiene un número que lo confirma: un conteo, una cifra, un hash.
3. **Si falta una pieza se recorta el alcance** — nunca se agrega tecnología para tapar el hueco. Sumar una herramienta se come el tiempo disponible y no arregla nada.

## 📖 Teoría

### El objetivo: 3 minutos, de la libreta al dashboard

Desde que disparas el pipeline hasta que el número nuevo se ve en la app, pasan **menos de 3 minutos**. No es capricho: es el presupuesto de tu demo final. Si tarda 20, no lo puedes mostrar en vivo y terminas contando lo que hiciste. Cronometra el tramo que controlas (local + push + el run): **no cronometres el `cron`**, la cola de GitHub no es tu código.

### El diagrama: los 8 eslabones

```
[1] ventas.csv crudo (50 k filas, sucio a propósito)
      ▼ pandas: tipos, fechas, categorías canónicas
[2] limpieza reproducible (antes / después / descartadas)
      ▼
[3] parquet_ventas/ particionado ──► [4] DuckDB: las 3 cifras de control
      ▼
[5] Neon: carga idempotente, sin duplicados
      ▼
[6] Actions: workflow + cron + secretos ──► [7] Telegram: 1 mensaje por evento
      ▼
[8] Streamlit Cloud mostrando el dato nuevo
```

### El checklist de 8 puntos: los 8 verdes en la misma corrida

| # | Eslabón | Verde cuando… | Cómo lo verificas |
|---|---|---|---|
| 1 | **Fuente** | el CSV crudo tiene el conteo que esperas | `wc -l ventas.csv` y el `md5` contra tu manifiesto |
| 2 | **Limpieza** | no queda una fecha inválida sin explicar | `src/limpio.py` imprime filas antes, después y descartadas |
| 3 | **Parquet** | abre y tiene el mismo conteo que la limpieza | `SELECT count(*) FROM 'parquet_ventas/*.parquet'` |
| 4 | **Base local** | las 3 cifras de control coinciden con la app | las 3 consultas en un `.sql`, siempre las mismas |
| 5 | **Neon** | conteo igual a local y 0 duplicados tras cargar dos veces | `count(*)` antes y después de la segunda carga |
| 6 | **Actions** | el run termina en verde y el artefacto se descarga | abre el run, baja el artefacto, mira su conteo |
| 7 | **Telegram** | llega **un** mensaje por evento | corre dos veces seguidas y cuenta los mensajes |
| 8 | **Dashboard** | la URL pública muestra el número nuevo | incógnito, recarga, compara con la cifra de Neon |

Ocho verdes en **la misma corrida**: en corridas distintas se cuela el eslabón que solo funciona cuando el anterior ya estaba caliente. Si no puedes decir qué entra y qué sale de una casilla, ese eslabón no está.

### La regla de oro: se recorta el alcance, no se suma tecnología

Cuando algo no llega, la tentación es agregar: un orquestador, otra librería, otra base "para probar". Eso no arregla el eslabón que falta, agrega uno nuevo que también hay que aprender, conectar y depurar.

| Si no te da… | Recortas… | Y NO agregas… |
|---|---|---|
| la ventana de fechas | fija 90 días y dilo | un orquestador que "lo resuelva" |
| el dashboard multipágina | una página con las 3 métricas | otra librería de gráficos |
| la carga a Neon | muestras DuckDB local y qué falta | una segunda base para comparar |

Recortar se **declara** al presentar: recortado y explicado se lee como criterio.

### El primer paso rojo: dónde se cortó la cadena

1. Corre los 8 puntos de arriba abajo, anotando ✅ o ❌ en `integrity.md`.
2. Quédate con el **primer ❌**. Los de abajo son consecuencia: si la limpieza no escribió el Parquet, la base no tiene qué leer y la app no tiene qué mostrar.
3. Arregla ese y vuelve a correr **todo** desde el paso 1. No sigas de largo con el resto en rojo.

## ⚠️ El error más común

> [!warning] `Binder Error: Referenced column "monto_total" not found in FROM clause!`
> ```
> duckdb.duckdb.BinderException: Binder Error: Referenced column "monto_total"
> not found in FROM clause! Candidate bindings: "total", "fecha", "categoria"
> ```
> **Qué significa:** renombraste una columna en la limpieza y la base (o la app) sigue pidiendo el nombre viejo. Cambiaste un eslabón y no avisaste a los demás. **Cómo se lee:** "Candidate bindings" lista los nombres que **sí** existen. Si ahí ves `total`, ya sabes quién cambió de nombre y quién no. **Cómo se arregla:** un solo lugar donde viven los nombres: `COL_MONTO = "total"` en un módulo, importado por todos, más `assert COL_MONTO in df.columns, f"falta {COL_MONTO}: {list(df.columns)}"`. Si el nombre nuevo es el definitivo, actualiza consultas y vistas en la misma corrida.

Este no es un error de adorno: es el que se lleva el tema si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, el tema completo

1. Crea `integrity.md` con la tabla de 8 puntos y corre el pipeline entero, marcando cada uno con la verificación de la columna derecha.
2. Cronometra el recorrido completo y anota el número real, aunque sea 11 minutos.
3. Rompe un eslabón (renombra una columna, cambia una ruta) y ubica el primer rojo sin mirar el último log.

### Reto — opcional, para quien quiera más

1. Corre el pipeline desde un **fork limpio** y lista los pasos manuales que te obligó a hacer: cada paso de esa lista es deuda.
2. Que comparar el conteo de la app con el de Neon sea un paso del workflow, no algo que miras tú.

## ✅ Criterios de logro

- [ ] Los 8 puntos verdes en la **misma** corrida, con su evidencia anotada
- [ ] El recorrido completo tarda menos de 3 minutos y puedes mostrar el número
- [ ] Puedes romper un eslabón y nombrar el primer rojo en menos de 2 minutos
- [ ] Hay al menos un recorte de alcance declarado por escrito, con su motivo

## 🔗 Para profundizar

- [DuckDB — SQL](https://duckdb.org/docs/sql/introduction) — para las consultas de control
- Anexo del vault: [[Chuleta de errores]] — el 80 % de los rojos de este tema ya está ahí

## 🧭 Problemas frecuentes

- **El run queda en cola** → los `schedule` solo corren en la rama por defecto; para probar usa `workflow_dispatch`.
- **Neon tarda 8 segundos la primera consulta** → arranque en frío. Calienta la base 1 minuto antes y no lo cuentes en los 3 minutos.

## ➡️ Siguiente tema

Vas a dejar ese pipeline en un repo que **otra persona corre desde cero** siguiendo tu README, y a auditar el de un compañero con una rúbrica de 10 criterios.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá tu `integrity.md`, el tiempo del recorrido y el link del run verde. En el siguiente tema arrancamos por el primer punto rojo que quedó acá.
