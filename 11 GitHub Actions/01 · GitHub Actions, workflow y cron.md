---
tipo: tema
modulo: "11 GitHub Actions"
capa: familiaridad
herramienta: GitHub Actions
resultado: "workflow que corre en cada push y por horario, con la hora en UTC y en Caracas a la vista"
tiempo-lectura: 16 min
prerrequisitos: ["Neon, Postgres en la nube"]
tags: [github-actions, cron, automatizar]
---

# GitHub Actions: workflow, job y cron en UTC

![[githubactions.svg|64]]

> [!tip] En una frase, antes de empezar
> **GitHub Actions es el robot de GitHub: corre tu código en una máquina que no es la tuya, solo, cuando pasa algo —un `push`, o la hora que tú fijas con un `cron`.**

> [!info] Lee esto antes de empezar — 16 minutos
> Léelo completo y marca `#duda` donde no te cierre. **No está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

El pipeline funciona porque lo corres tú, a mano, cada mañana. Un día te acuerdas a las once, otro día no te acuerdas, y el dashboard muestra el dato de anteayer sin que nada avise. Si estás de viaje o se va la luz, el "script de todos los días" no corre: vive en tu laptop.

También está el otro extremo: lo automatizas con el `cron` de tu computadora, y ahora depende de que la computadora esté prendida a esa hora. Si la apagas a la noche, el cron de las 7 nunca corre.

**Eso es lo que resuelve Actions:** un robot en la nube que corre cuando pasa un evento —un `push` a tu rama, una hora del día— y te deja el log de qué hizo. Tu laptop puede estar apagada.

> [!note] Qué no es
> No contratas nada ni administras una máquina: cada job arranca en una máquina nueva, hace su trabajo y se borra. Por eso tampoco es un lugar para guardar archivos: lo que quede ahí se va con ella.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Tener un workflow en verde en cada push, y leer en el log qué step corrió y en qué orden |
| **Después** | Ponerle un `schedule` al mismo workflow y calcular a qué hora de Caracas corre |

## 🧠 Conceptos

1. **workflow → job → step: la jerarquía** — un archivo `.yml`, adentro uno o más jobs, adentro de cada job los steps. Los jobs corren en máquinas distintas; los steps comparten la misma.
2. **El cron está en UTC, siempre** — Caracas es UTC-4. `"0 7 * * *"` **no** es a las 7 de la mañana: es a las 3 de la madrugada.
3. **Se lee el primer step rojo, no el último** — el log termina en el step que se cayó; la causa está antes o en ese mismo.

## 📖 Teoría

### 1. workflow, job y step

```yaml
name: prueba
on:
  push:
    branches: [main]
jobs:
  hola:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - name: Saludar
        run: echo "corriendo en $(date -u)"
```

- **`on`**: cuándo. Acá, cada push a `main`.
- **`jobs`**: un job es una máquina nueva. Dos jobs son dos máquinas en paralelo, que no comparten archivos.
- **`steps`**: dentro del job, en orden. Si un step falla, los siguientes no corren.
- **`uses` vs `run`**: `uses` trae una acción hecha; `run` ejecuta un comando de shell.

Casi todos los "no me encuentra el archivo" son un archivo que existía en el job de al lado: `actions/checkout` es lo que baja tu código a la máquina. La versión mayor cambia cada tanto; mírala en el Marketplace.

### 2. cron en UTC: la cuenta que se equivoca todo el mundo

El cron tiene cinco campos: minuto, hora, día del mes, mes y día (`0` = domingo). Y GitHub lo interpreta **en UTC**.

```yaml
on:
  push:
  schedule:
    - cron: "0 11 * * *"   # 11:00 UTC = 07:00 en Caracas (UTC-4)
  workflow_dispatch:        # botón "Run workflow" para probar sin esperar
```

| cron | UTC | Caracas |
|---|---|---|
| `"0 7 * * *"` | 07:00 | 03:00 |
| `"0 11 * * *"` | 11:00 | 07:00 |
| `"30 1 * * 1"` | lunes 01:30 | **domingo** 21:30 |

Ese último es el que rompe cabezas: cuando el UTC es de madrugada, en Caracas todavía es el día anterior. **Regla práctica: 7 de la mañana en Caracas = `0 11 * * *`.**

### 3. Leer el log

Un run tiene tres niveles de log y tú miras el de abajo: el step. La regla es buscar **el primer rojo**, no el último. Con la CLI: `gh run list --limit 5` te dice qué corrió y cómo terminó; `gh run view --log-failed` te muestra solo lo que falló.

El truco para entender qué hora era: el step del ejemplo imprime `date -u`. Si tu cron dice una cosa y el log dice otra, no adivines: el log tiene la respuesta.

## ⚠️ El error más común

> [!warning] `Invalid workflow file`
> ```text
> Invalid workflow file
> You have an error in your yaml syntax on line 12
> ```
> **Qué significa:** el archivo no es YAML válido, así que GitHub **no lo ejecuta**: ni siquiera aparece un job con rojo, aparece este cartel. **Cómo se lee:** mira la línea que dice y **la de arriba**. El 80 % de las veces el error está antes: en YAML la indentación es la estructura. **Cómo se arregla:** dos espacios por nivel, **nunca tabs**. En VS Code, "Convert Indentation to Spaces" y revisa que los guiones de `steps:` queden alineados. Valida antes de pushear con `python -c "import yaml; yaml.safe_load(open('.github/workflows/prueba.yml'))"`.

> [!caution] El otro error frecuente no falla: sale verde
> `"0 7 * * *"` corrió perfecto... a las 3 de la mañana de Caracas. Nada rojo, nada avisado, y el informe de las 7 nunca llegó a tiempo. Si esperabas las 7 y el run fue a las 3, el bug es la zona horaria, no el cron.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Crea `.github/workflows/prueba.yml` con un job que corra en cada push a `main`. Tiene que quedar en verde.
2. Agrégale un step que imprima la hora UTC y la de Caracas (`date -u` y `TZ=America/Caracas date`). Copia las dos líneas del log.
3. Agrégale `schedule` con un cron **y escribe la cuenta**: qué hora UTC, qué hora Caracas. Después rompe el YAML a propósito (un tab o un `- name:` corrido) y guarda el mensaje exacto.

### Reto — opcional

1. Pon `workflow_dispatch` y corre el workflow a mano, sin tocar el código.
2. Agrega un step que falle a propósito (`exit 1`) y comprueba que los steps siguientes **no** corren. Explica por qué el último step del archivo no aparece en el log.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] El run del último push está en verde y puedes nombrar sus steps en orden
- [ ] Escribiste la conversión UTC → Caracas de tu cron y el log la confirma
- [ ] De un run en rojo, señalas el primer step que falló sin dudar

## 🔗 Para profundizar

- [Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) — la sección `schedule`: rama por defecto, retrasos y los 60 días
- [Workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) — la referencia de `on`, `jobs` y `steps`
- [crontab.guru](https://crontab.guru/) — pegas tu cron y te dice las próximas ejecuciones (en UTC)

## 🧭 Problemas frecuentes

- **El cron no corre nunca** → los workflows programados corren **solo en la rama por defecto**. Si el archivo vive en una rama, no dispara.
- **Corre, pero tarde** → pasa: en horas de mucha carga el `schedule` se retrasa varios minutos. No es tu YAML.
- **Corrió y un mes después dejó de correr** → en repositorios públicos GitHub desactiva los workflows programados tras 60 días sin actividad. Se vuelven a habilitar desde la pestaña Actions.
- **Duda de costos** → en repos públicos los runners estándar son gratis e ilimitados; en privados descuentan de la cuota mensual incluida. Si el repo es público, esto no te cuesta nada hoy (y las condiciones cambian: miralas antes de decidir).

## ➡️ Siguiente tema

Le vas a pasar un secreto al workflow sin escribirlo en el archivo, y a encadenar dos jobs para que el segundo avise cuando el primero se cae → [[02 · GitHub Actions, secretos y alertas]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la cuenta UTC → Caracas, el nombre del primer step rojo y el texto del `Invalid workflow file` que provocaste. El siguiente tema retoma lo que quede acá.
