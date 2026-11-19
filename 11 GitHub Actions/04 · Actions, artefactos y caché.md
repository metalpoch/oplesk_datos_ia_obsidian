---
tipo: tema
modulo: "11 GitHub Actions"
capa: dominio inicial
herramienta: GitHub Actions
resultado: "workflow que deja el CSV de cada ejecución como artefacto verificable y cachea las dependencias para no descargarlas otra vez"
tiempo-lectura: 20 min
prerrequisitos: ["01 · GitHub Actions, workflow y cron"]
tags: [tema, github-actions, artefactos, cache]
---

# GitHub Actions: artefactos y caché

![[githubactions.svg|64]]

> [!tip] En una frase
> Un **artefacto** es el archivo que el workflow deja guardado para que puedas revisarlo después de que la máquina desaparece, y la **caché** es lo que el workflow guarda para no volver a descargarlo: uno deja evidencia, la otra ahorra tiempo.

> [!info] Cómo leer este tema — 20 minutos
> Este tema cierra el pipeline: ya corre solo y ya tiene un job que avisa cuando se cae, y ahora tiene que dejar constancia de lo que produjo. Se apoya en el workflow de [[01 · GitHub Actions, workflow y cron]].
>
> **Los números no están acá a propósito.** Cuánto vive un artefacto, cuánto pesa o cuánta caché admite un repositorio cambia con el plan: lo que se aprende es la conducta y dónde consultar el número.

## 🤔 Para qué sirve

El workflow corre en verde, el log dice que `extraer.py` terminó y la base tiene las filas nuevas, pero abres el run para revisar el CSV y el archivo no está. No se perdió por un error del script: el runner es una máquina que GitHub prepara para ese job y destruye al terminar, y lo que tu código escribió ahí se fue con ella. Si el resultado no sale del runner, no existe.

En Excel el archivo se queda donde lo guardaste, y ese hábito explica la expectativa. Acá no hay carpeta que sobreviva al job: si quieres el archivo, el workflow tiene que subirlo antes de terminar.

El segundo costo es el tiempo: cada ejecución arranca en una máquina vacía, así que `pip install -r requirements.txt` descarga pandas, duckdb y el resto de nuevo, en cada ejecución, sin excepción.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Dejar el CSV de cada ejecución como artefacto descargable y comprobar que llegó con filas |
| **Después** | Cachear las dependencias y demostrar con los tiempos del log que la siguiente ejecución instala menos |

## 🧠 Qué es cada cosa

| Pieza | Qué es | Dónde se ve |
|---|---|---|
| **Runner** | la máquina temporal donde corre el job; existe mientras el job corre | `runs-on: ubuntu-latest` |
| **Artefacto** | archivo que sale del runner y queda atado a esa ejecución | se lista al final de la página del run |
| **Caché** | archivos que GitHub guarda entre ejecuciones y restaura si la clave coincide | `cache-hit` en el log del step |

## 🧭 Qué no es

- **Un artefacto no es almacenamiento ni un respaldo.** No es el lugar donde vive el dato —eso es Neon o el Parquet del repositorio— y cuando vence se borra, sin restauración.
- **La caché no es un artefacto ni sostiene el pipeline.** No se descarga para leerla y no prueba nada; si desaparece, el pipeline solo tarda más. Un workflow que falla sin caché está roto.
- **Un artefacto no se sube por si acaso.** Si el dato ya tiene un lugar con dueño, la copia envejece sola y termina contradiciendo a la base en una discusión.

## 🧠 Conceptos

1. **El runner se borra: lo que quieras conservar tiene que salir de ahí.** No hay carpeta, ni historial, ni segundo intento.
2. **Artefacto es evidencia; caché es velocidad.** Uno lo mira una persona, la otra la usa el workflow sin que nadie la mire.
3. **Un artefacto sin verificar miente.** Un run en verde con un CSV de solo encabezados es peor que un run en rojo.

## 📖 Teoría

### 1. El runner es una máquina desechable

```yaml
jobs:
  extraer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - name: Generar el CSV
        run: python extraer.py
```

Ese job puede terminar en verde y el archivo existió hasta el último segundo: cuando el job termina, la máquina se borra y no queda ningún lugar del run donde descargarlo. Lo mismo pasa entre jobs, que corren en máquinas distintas y no comparten carpeta.

### 2. Subir el resultado con `actions/upload-artifact`

Un artefacto es un archivo que sale del runner y queda asociado a esa ejecución. Se sube con un step al final del job y se descarga desde la sección **Artifacts** de la página del run, que entrega un zip; desde la terminal, `gh run download <run-id> -n ventas-<run-id>`.

```yaml
      - name: Subir el CSV como artefacto
        uses: actions/upload-artifact@v7
        with:
          name: ventas-${{ github.run_id }}
          path: salidas/ventas.csv
          if-no-files-found: error
```

- **`name`**: lo que aparece en la lista del run. Tiene que ser único en esa ejecución, porque dos subidas con el mismo nombre chocan y un artefacto creado no se reemplaza.
- **`path` e `if-no-files-found`**: la ruta va relativa al workspace y acepta un archivo, una carpeta o un patrón, conservando la estructura dentro del zip. Si no encuentra nada, el valor por defecto solo advierte; `error` lo convierte en un run en rojo.

### 3. Verificar el artefacto: que pese, que tenga filas, que abra

`upload-artifact` no revisa el contenido: si el archivo existe lo sube, y si tiene solo la línea de encabezado también. El resultado más caro es un step en verde con un archivo vacío, porque el pipeline parece sano.

```yaml
      - name: Verificar el CSV
        run: |
          ls -lh salidas/ventas.csv
          filas=$(($(wc -l < salidas/ventas.csv) - 1))
          echo "filas de datos: $filas"
          if [ "$filas" -lt 1 ]; then
            echo "El CSV no tiene filas de datos"
            exit 1
          fi
```

El step deja el conteo en el log y falla si no hay filas: con `exit 1` el job se pone en rojo, los steps siguientes no corren y el archivo vacío no se sube. Después de descargarlo, `wc -l` y un `pd.read_csv` confirman que abre y que trae las filas que el log prometió. Falta una verificación que no es de tamaño: que el dato sea de esta ejecución, porque un CSV idéntico al anterior pasa todas las pruebas de forma y solo se distingue por la marca de tiempo del log.

### 4. Retención y límites: la evidencia vence

Todo artefacto tiene un vencimiento: el plazo se configura en el repositorio, en *Settings → Actions → General*, y un step puede sobrescribirlo con `retention-days` sin pasar el máximo del repositorio. Los artefactos también ocupan almacenamiento, que entra en la cuota de GitHub, y los topes están en la referencia de límites: se consultan antes de decidir, no se memorizan.

| Lo que produce el job | ¿Artefacto? | Por qué |
|---|---|---|
| El CSV que ya se cargó a Neon | No | la base es el lugar del dato; el artefacto sería una copia paralela que nadie actualiza |
| El Parquet que el repositorio ya versiona | No | ya tiene dueño y tiene historial; subirlo duplica y confunde |
| Un reporte con conteos, errores y duración | Sí | no existe en ningún otro lado y es lo que se revisa después del run |
| Un archivo con credenciales | Nunca | el artefacto se descarga; el enmascarado del log no cubre archivos |

### 5. Caché: no volver a descargar lo mismo

Cada ejecución arranca en una máquina vacía, así que las dependencias se descargan de nuevo. La caché guarda la carpeta donde quedan esos paquetes y la restaura al empezar, antes de instalar.

```yaml
      - name: Restaurar la caché de pip
        id: cache-pip
        uses: actions/cache@v6
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      - name: Instalar dependencias
        run: pip install -r requirements.txt
```

- **`key` y `restore-keys`**: `hashFiles` mete el hash de `requirements.txt`, y como una caché no se reemplaza, la clave nueva es la que invalida la vieja. Sin coincidencia exacta, `restore-keys` restaura por prefijo una caché anterior.
- **`cache-hit`**: vale `true` solo con coincidencia exacta; con la restauración por prefijo vale `false` y no es un error. Se compara como texto: `'true'`. La primera ejecución nunca acierta: todavía no hay nada que restaurar.
- **La caché es por rama**: la de la rama por defecto la usan las demás; la de una rama no la usa la principal.
- **No se cachea el `.venv`**: tiene rutas absolutas del runner y no es portable. Tampoco los datos ni el CSV de salida, que es artefacto.

| | Artefacto | Caché |
|---|---|---|
| **Para qué** | guardar lo que produjo la ejecución | acelerar la siguiente ejecución |
| **Si falta** | se pierde la evidencia de esa ejecución | el pipeline solo tarda más |

## ⚠️ El error más común

> [!warning] Run en verde y el resumen sin artefactos
> ```text
> Warning: No files were found with the provided path: salidas/ventas.csv. No artifacts will be uploaded.
> ```
> **Qué significa:** el `path` no encontró nada. El CSV quedó en otra carpeta, el script lo escribió con otro nombre, o el step subió antes de generarlo. **Cómo se lee:** es un warning, no un error: el job continúa, el run queda en verde y la lista de artefactos vacía. Un verde sin archivo es la mentira más cara del pipeline. **Cómo se arregla:** imprime la ruta real antes de subir (`ls -R salidas`), corrige el `path` y pon `if-no-files-found: error` para que un archivo ausente se caiga en rojo en lugar de pasar.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Agrega el step de `actions/upload-artifact` a tu workflow, con nombre por ejecución, `path` al CSV y `if-no-files-found: error`. Corre, descarga el artefacto y pega el conteo de filas del log junto al de tu copia local.
2. Provoca el warning: apunta el `path` a una ruta que no existe, corre el workflow, copia la línea exacta y explica por qué el run no se puso en rojo. Después cambia el valor a `error` y comprueba el rojo.
3. Agrega la caché de pip con `key` basada en `hashFiles('requirements.txt')` y corre el workflow dos veces. Pega la duración del step de instalación en cada una y el valor de `cache-hit`.

### Reto — opcional

1. Sube un segundo artefacto con un `reporte.txt` que genere el propio pipeline (filas, columnas, marca de tiempo). Descarga los dos y explica por qué ese reporte sí merece ser artefacto y el CSV no.
2. Agrega `if: steps.cache-pip.outputs.cache-hit != 'true'` al step de instalación y corre. Explica el error que aparece y por qué la caché no reemplaza a la instalación.

## ✅ Criterios de logro

- [ ] Hay un artefacto en el run, con nombre por ejecución, y lo descargaste
- [ ] El log deja el conteo de filas y coincide con el archivo descargado
- [ ] La segunda ejecución muestra `cache-hit` en `true` e instala en menos tiempo que la primera
- [ ] Puedes explicar en una frase por qué el CSV no va como artefacto si ya está en Neon

## 🔗 Para profundizar

- [Workflow artifacts](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflow-artifacts) — qué es un artefacto y en qué se diferencia de la caché
- [Dependency caching](https://docs.github.com/en/actions/concepts/workflows-and-actions/dependency-caching) — alcance por rama, claves y restauración
- [Limits](https://docs.github.com/en/actions/reference/limits) — los números que este tema no cita

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| Run en verde y sin artefactos | el `path` no coincide con la ruta real | `ls -R` antes de subir y `if-no-files-found: error` |
| `409 Conflict` al subir | ya existe un artefacto con ese nombre en la ejecución | nombre con `run_id` y `run_attempt`, o `overwrite: true` |
| `cache-hit` nunca vale `true` | la `key` cambia en cada ejecución o el step va después de instalar | clave estable con `hashFiles`, caché antes de `pip install` |
| `ModuleNotFoundError` con la caché en `true` | se saltó la instalación creyendo que la caché instala | instalar siempre; la caché solo evita la descarga |

## ➡️ Siguiente tema

El pipeline ya deja evidencia y ya no repite descargas, pero nadie se entera de si salió bien: el verde y el rojo siguen esperando a que alguien abra GitHub. Ese aviso, en el teléfono, es un bot de Telegram → [[01 · Telegram, el bot de alertas]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega el conteo de filas del log junto al de tu archivo descargado, la línea del warning que provocaste y los dos tiempos de instalación con `cache-hit`: eso es la evidencia de que el pipeline no miente.
