---
tipo: tema
modulo: "11 GitHub Actions"
capa: dominio inicial
herramienta: GitHub Actions
resultado: "en un run en rojo, señalar el primer step que falló, nombrar la causa entre los cuatro fallos habituales y volver al verde con un push"
tiempo-lectura: 20 min
prerrequisitos: ["01 · GitHub Actions, workflow y cron"]
tags: [tema, github-actions, depuracion]
---

# Actions: leer el log y depurar

![[githubactions.svg|64]]

> [!tip] En una frase
> **El run en rojo no se adivina: se lee. Se baja por el log hasta el primer step que falló, porque lo que aparece debajo es consecuencia, no causa.**

> [!info] Cómo leer este tema — 20 minutos
> Este tema no da nada por sabido: ni la pantalla de un run ni lo que es un log. Si algo no cierra, escribe `#duda` en esa línea.
>
> **No está acá:** cómo se escribe un workflow desde cero ni cómo se manejan los secretos; eso está en los dos temas anteriores del módulo. Acá se trabaja sobre un workflow que ya existe y ya se cayó.

## 🤔 Para qué sirve

El workflow viene funcionando sin fallar desde hace tiempo y una mañana el dashboard muestra el dato de anteayer. Abres la pestaña Actions y hay un run en rojo.

Entras, y lo que aparece es una pared de texto gris. Bajas hasta el final, encuentras un renglón que dice que el proceso terminó con un código distinto de cero, y eso no explica nada.

Desde ahí empieza lo caro: re-ejecutar por si acaso, agregar un `pip install` que quizá falte, cambiar una ruta a ver si era eso. Varios intentos después el run sigue rojo y no sabes más que al principio.

En Excel, un dato mal referenciado se ve al instante. En un pipeline, un archivo que no existe se ve recién cuando alguien lee el log.

**Depurar es un procedimiento, no intuición:** saber dónde está el log de cada nivel, leer el primer rojo en vez del último, reconocer cuatro fallos que se repiten siempre y usar dos comandos para ver lo que la máquina ve. Eso es la diferencia entre diagnosticar y adivinar.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Abrir un run en rojo, ubicar el job y el step que fallaron, y señalar la línea del log donde está la causa |
| **Después** | Nombrar cuál de los cuatro fallos habituales es, corregirlo con un push y decidir si re-ejecutar todo o solo lo que falló |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Dónde se ve |
|---|---|---|
| **run** | una ejecución completa del workflow, disparada por un evento | la lista de la pestaña Actions, con el commit que la disparó |
| **job** | una máquina (runner) que ejecuta un grupo de steps, con su propio log | la barra lateral izquierda del run |
| **step** | un bloque del job, sea `- uses:` o `- run:` | un bloque plegable dentro del log del job |
| **log** | el texto que el runner imprimió al ejecutar | dentro de cada step, expandido |
| **intento** | cada re-ejecución del mismo run | el menú `Latest`, a la derecha del nombre del run |

## 🧭 Qué no es

- **No es reescribir el workflow.** Casi nunca falla la lógica: falla el entorno, una ruta o una dependencia.
- **No es leer de abajo hacia arriba.** El final del log dice cómo terminó, no por qué.
- **Un run en verde no prueba que el trabajo quedó hecho.** Puede terminar sin escribir nada y no avisar.
- **No es probar hasta que pase.** Re-ejecutar sin cambiar nada repite la misma pregunta al mismo sistema.

## 🧠 Conceptos

1. **run → job → step: tres niveles y tres logs** — el run no tiene un log único: se lee el log del job y, dentro, el del step.
2. **El primer rojo, no el último** — cuando un step falla, los siguientes no corren; el final del log no es la causa, es la consecuencia.
3. **Si local funciona, la diferencia es el entorno** — misma versión de Python, misma carpeta, mismas variables. Mientras no coincidan, el código no es el sospechoso.
4. **Re-ejecutar repite el mismo commit** — el botón no toma tus cambios locales. Sin push, el segundo intento es el primero otra vez.

## 📖 Teoría

### 1. Anatomía del run: tres niveles y tres logs

Un workflow se ve así en el archivo, y cada bloque aparece en el log del job en ese mismo orden:

```yaml
jobs:
  diario:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7        # step 1: baja tu código al runner
      - name: Extraer                    # step 2
        run: python extraer.py
```

La página del run tiene tres zonas. Arriba, el nombre del workflow, el commit y el resultado. A la izquierda, los jobs: cada uno es una máquina distinta y tiene su propio log, así que el log de un job no muestra nada del otro. En el centro, el log del job elegido, partido en un bloque por step, y cada bloque se expande por separado.

Hay dos steps que no escribiste y siempre están: `Set up job` y `Complete job`; el primero imprime la imagen del runner y el enlace a las herramientas preinstaladas, que es el primer lugar donde mirar cuando sospechas del entorno. En `Summary` quedan los artefactos y las anotaciones, y el log completo del job se descarga con el menú de engranaje del panel de log, `Download log archive`.

### 2. El primer rojo, no el último

Los steps de un job corren en orden. El shell por defecto en Linux es `bash -e {0}`: dentro de un step, el shell se detiene en el primer comando que falla, y los steps siguientes ya no corren, quedan salteados, no rojos.

Eso cambia por completo dónde se busca. No se busca el último mensaje: se busca **el primer step marcado como fallido** y se lee **desde arriba de ese bloque**. Ahí está el comando que se ejecutó, su salida y el error.

El renglón con el que el runner cierra un step fallido, `Process completed with exit code 1.`, es el resultado, no la causa: informa que el proceso terminó mal, no por qué. Un traceback de Python se lee al revés de como se lee el log: la última línea dice el tipo de error y el mensaje; las de arriba, dónde ocurrió.

Si el run tiene varios jobs, los que dependen con `needs` de un job fallido quedan salteados, en gris. No son fallos: son consecuencias. El rojo que importa es uno solo.

> [!warning] El verde que esconde un fallo
> Con el shell por defecto, `bash -e {0}`, **no hay `pipefail`**: en una tubería, el estado que cuenta es el del último proceso. Un step que hace `python extraer.py | tail -5` puede terminar en verde aunque Python haya fallado. Si escribes `shell: bash`, el comando interno es `bash --noprofile --norc -eo pipefail {0}` y el fallo sí se detecta.

### 3. Los cuatro fallos que vas a tener

**Falta una dependencia.** El step que importa la librería muere con `ModuleNotFoundError: No module named 'psycopg'`, y en tu computadora el mismo script corre. La causa es que el runner arranca limpio: instala lo que dice `requirements.txt` y nada más. Si falta una línea ahí, no existe. Se arregla en el archivo y se pushea; re-ejecutar no sirve, porque el commit es el mismo.

**La ruta no existe en el runner.** El síntoma es `FileNotFoundError: [Errno 2] No such file or directory: 'datos/ventas.csv'`. Las causas habituales: el archivo nunca se commiteó, está en `.gitignore`, el nombre tiene otra mayúscula, o el step corre desde una carpeta distinta de la que supones. La ruta útil es la relativa a la raíz del repo, que es también `GITHUB_WORKSPACE`.

**Un secreto no está definido.** Acá no hay error de Actions: la expresión `${{ secrets.DATABASE_URL }}` se resuelve como texto vacío y el step sigue. El fallo llega después y por otro lado, con un `KeyError: 'DATABASE_URL'` al leer la variable, o con un error de conexión del driver. Se revisa el nombre exacto en *Settings → Secrets and variables → Actions* y se comprueba sin imprimir el valor, con `if [ -z "$DATABASE_URL" ]; then echo "falta DATABASE_URL"; fi`.

**El comando corrió bien y el archivo quedó vacío.** El run sale en verde y el archivo existe, con cero filas. Hay dos casos y no se ven igual: si el archivo tiene solo el encabezado, el que lo lee obtiene un DataFrame vacío y nadie se queja; si tiene cero bytes, el que lo lee revienta con `pandas.errors.EmptyDataError: No columns to parse from file`. La causa típica es un filtro que no dejó filas, o un script que escribió en otra carpeta y el archivo se fue con la máquina. Se corrige verificando en el mismo step y cortando a propósito antes de escribir: `if df.empty: raise SystemExit("ventas.csv no trajo filas")`.

### 4. `echo` y `set -x`: ver lo que el runner ve

Antes de tocar el script, conviene imprimir el entorno. Un step de diagnóstico antes del step que falla responde casi todas las dudas:

```yaml
      - name: Qué hay en el runner
        run: |
          set -x
          pwd
          ls -la
          ls -la datos || true
          python --version || python3 --version
```

`pwd` dice desde qué carpeta corre el step; el runner ubica tu repo en `/home/runner/work/<repo>/<repo>`. `ls -la` dice qué archivos hay de verdad ahí, con mayúsculas incluidas. La versión de Python suele explicar la diferencia entre tu computadora y la máquina. El `|| true` del `ls` sobre `datos` está a propósito: con `-e`, una carpeta que no existe cortaría el step antes de terminar el diagnóstico.

`set -x` imprime cada comando antes de ejecutarlo, con un `+` adelante, y debajo la salida de ese comando. Sirve para saber **qué se ejecutó exactamente**, incluidas las variables ya expandidas. Vale solo para su step: cada step corre en su propio proceso, así que no contamina a los demás.

> [!caution] `set -x` y secretos, en el mismo step, no
> `set -x` imprime el comando con las variables ya reemplazadas. El enmascarado de GitHub cubre el valor exacto del secreto, no una versión recortada o transformada. Un step que maneja credenciales se diagnostica con `echo` del nombre de la variable, nunca con `set -x`.

### 5. La regla de oro y el segundo intento

**Si local funciona y en el runner no, la diferencia es el entorno, no el código.** No hay excepciones útiles: el código es el mismo archivo. Lo que cambia es la máquina.

Las diferencias que explican casi todo: la versión de Python, las librerías instaladas, los archivos que llegaron al runner, las variables y secretos disponibles, el sistema operativo, la zona horaria (el runner está en UTC) y los permisos de escritura. Se descubren comparando, no adivinando: los mismos `pwd`, `ls -la` y `python --version` en tu terminal y en el step de diagnóstico, línea por línea.

Después de arreglar, hay dos botones en el menú `Re-run jobs`, arriba a la derecha del run. **`Re-run failed jobs`** repite solo los jobs que fallaron, más los que dependen de ellos; los demás conservan su resultado. **`Re-run all jobs`** repite todo. El intento usa el mismo commit y la misma referencia, tiene su propio log y queda como un attempt distinto bajo el menú `Latest`; hay un techo de cincuenta re-ejecuciones y una ventana de treinta días.

La regla práctica: si cambiaste un archivo del repo, push y run nuevo, porque el botón no ve tu laptop. Si lo que cambió fue un secreto o una condición de la red, `Re-run failed jobs` alcanza y ahorra tiempo. Y si el fallo fue un timeout del runner, re-ejecutar todo es lo razonable, porque no hubo nada que arreglar.

## ⚠️ El error más común

> [!warning] Re-ejecutar sin haber pusheado
> El botón repite el mismo commit, así que el segundo intento corre exactamente el mismo código y falla en el mismo step, con el mismo log. Se comprueba así: el encabezado del run muestra el commit que se ejecutó, y `git log -1 --format=%h` en tu terminal tiene que dar el mismo. Si no coincide, todavía no probaste nada.

El otro silencio, más caro porque no avisa, es el run verde con el archivo vacío: el pipeline "funcionó" y el dashboard muestra cero. Un run en verde solo prueba que los comandos terminaron bien, no que hicieron el trabajo.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Rompe el run por una dependencia: quita del `requirements.txt` la línea de una librería que tu script importe y pushea. Copia el primer step rojo y la última línea del mensaje. Si la imagen del runner ya trae esa librería instalada, haz lo mismo con un archivo propio que no commiteaste: el mensaje es de la misma familia y el ejercicio sirve igual.
2. Agrega antes del step que falla un step de diagnóstico con `set -x`, `pwd`, `ls -la` y la versión de Python. Copia del log tres líneas: una que empiece con `+`, una del listado de archivos y la de la versión.
3. Provoca el verde vacío: en el step que escribe el CSV, pon un filtro que no deje ninguna fila. El step sale en verde. Comprueba el tamaño con `wc -c salida.csv` en el mismo step, y después vacía el archivo a propósito en ese step con `: > salida.csv` y corre el que lo lee. Anota qué pasa en cada uno de los dos casos.

### Reto — opcional

1. Arregla el fallo con un push y comprueba que el run nuevo sale en verde. Después usa `Re-run failed jobs` sobre el run viejo y explica en dos líneas por qué sigue rojo aunque el repo ya esté arreglado, y dónde se ve que es un intento distinto.
2. Escribe el criterio completo: qué miras primero en un run rojo, en qué orden, y cuándo re-ejecutas solo lo que falló en vez de todo. Aplícalo a un caso en el que el fallo fue un timeout del runner y el código no cambió.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Señalas el primer step rojo de un run y la línea del log donde está la causa, sin recorrer el archivo completo
- [ ] Nombras cuál de los cuatro fallos habituales es cada uno de los que provocaste
- [ ] Tu step de diagnóstico imprime la carpeta, los archivos y la versión de Python del runner
- [ ] Explicas por qué un re-run sin push falla igual, y qué hay que hacer en su lugar
- [ ] Distingues un run verde vacío de un run que hizo el trabajo

## 🔗 Para profundizar

- [Using workflow run logs](https://docs.github.com/en/actions/how-tos/monitor-workflows/use-workflow-run-logs) — los steps `Set up job` y `Complete job`, la búsqueda y la descarga del log
- [Re-running workflows and jobs](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/re-run-workflows-and-jobs) — qué re-ejecuta cada botón, qué conserva y sus límites
- [Enabling debug logging](https://docs.github.com/en/actions/how-tos/monitor-workflows/enable-debug-logging) — la casilla al re-ejecutar y las variables `ACTIONS_STEP_DEBUG` y `ACTIONS_RUNNER_DEBUG`
- [Variables](https://docs.github.com/en/actions/reference/workflows-and-actions/variables) — `GITHUB_WORKSPACE` y el resto de las rutas del runner

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| El log termina en un código de salida y nada más | Se está leyendo la última línea como si fuera la causa | Subir al primer step rojo y leer su bloque desde arriba |
| `ModuleNotFoundError` en el runner y no en local | Librería instalada a mano en tu computadora | Agregarla a `requirements.txt` y pushear |
| `FileNotFoundError` con una ruta que en tu computadora existe | Archivo no commiteado, en `.gitignore` o con otra mayúscula | `ls -la` en un step de diagnóstico; `git ls-files` para ver qué se subió |
| El secreto llega vacío y el workflow no avisa | Nombre mal escrito o el environment no está declarado en el job | Probar con `if [ -z "$VAR" ]`; revisar Settings |
| Step en verde y archivo de cero filas o cero bytes | Un filtro que no dejó filas, o el archivo se escribió en otra carpeta | `wc -c` en el mismo step y salir con error si no hay filas |
| Un job quedó en gris, no en rojo | `needs` sobre un job que falló | No es un fallo: es una consecuencia; el rojo está arriba |
| Hay dos intentos y no sabes cuál mirar | Una re-ejecución | Menú `Latest`: cada intento tiene su propio log |

## ➡️ Siguiente tema

El diagnóstico deja de depender de que estés mirando la pestaña Actions: el resultado del run va a llegar a tu teléfono con un bot de Telegram → [[01 · Telegram, el bot de alertas]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega el primer step rojo con la línea que explica la causa, las tres líneas del step de diagnóstico y el tamaño del archivo vacío. El siguiente tema retoma lo que quede acá.
