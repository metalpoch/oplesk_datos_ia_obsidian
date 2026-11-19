---
tipo: tema
modulo: "11 GitHub Actions"
capa: familiaridad
herramienta: GitHub Actions
resultado: "dos jobs encadenados con `needs`, secreto fuera del repo y alerta cuando el pipeline falla"
tiempo-lectura: 16 min
prerrequisitos: ["GitHub Actions, workflow y cron"]
tags: [github-actions, secretos, automatizar]
---

# GitHub Actions: secretos, `needs` y alertas

![[githubactions.svg|64]]

> [!tip] En una frase, antes de empezar
> **Un secreto de GitHub Actions es un valor —una contraseña, un token, una cadena de conexión— que guardas en GitHub y que el workflow usa sin quedar escrito en el archivo.**

> [!info] Lee esto antes de empezar — 16 minutos
> Léelo completo y marca `#duda` donde no te cierre. **No está acá:** la solución de los ejercicios; eso se construye en la práctica.

## 🤔 Para qué sirve

Para que el job escriba en la base, le pegas la cadena de conexión en el `.yml`. Funciona. Pero el repo es público, y desde ese momento tu contraseña vive en el historial: borrarla del archivo no la borra del pasado, y cualquiera que haya clonado el repo la tiene.

Al mismo tiempo pasa lo otro: el run se cae y nadie se entera. Queda un rojo en una pestaña que nadie abre, hasta que alguien pregunta por qué el número de ayer no llegó. El pipeline llevaba días callado.

**Eso es lo que resuelven estas dos piezas:** el secreto guarda la credencial fuera del archivo y la inyecta en el momento de correr, y el job de alerta avisa cuando algo se cae, sin que tengas que estar mirando.

> [!note] Qué no es
> La variable se lee tal cual y sirve para rutas y nombres; el secreto se enmascara en el log y sirve para lo que no puede salir de ahí. Atención: el enmascarado protege el log, no los archivos. Un secreto escrito en un archivo o subido como artefacto sale en claro.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Encadenar un job que extrae y otro que carga con `needs`, y pasarle la credencial desde los secretos del repo |
| **Después** | Provocar un fallo a propósito y ver la alerta dispararse en el run |

## 🧠 Conceptos

1. **Secretos vs. variables** — ambos se configuran en el repo; el secreto se enmascara en el log y la variable se lee tal cual. La diferencia no es de comodidad: es de quién puede verla.
2. **`needs`: el orden entre jobs** — dos jobs son dos máquinas distintas. `needs: extraer` hace que `cargar` espere, y que no corra si `extraer` falló.
3. **`if: failure()`: avisar cuando algo se cae** — sin eso, el pipeline rojo se queda callado hasta que alguien abre GitHub por casualidad.

## 📖 Teoría

### 1. Secretos y variables

| | Secret | Variable |
|---|---|---|
| En el log | aparece como `***` | aparece tal cual |
| Para qué | contraseñas, tokens | rutas, nombres, banderas |

```yaml
    steps:
      - name: Cargar a Neon
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: python cargar.py
```

El secreto entra como **variable de entorno**, no como texto en el archivo. Tu script de carga no cambia: sigue leyendo `os.environ["DATABASE_URL"]`. Configuración: *Settings → Secrets and variables → Actions*.

**Por qué importa:** si la cadena vive en el `.yml`, cualquiera con acceso al repo tiene tu base. Y atención a esto: **el enmascarado protege el log, no los archivos.** Un secreto escrito en un archivo, subido como artefacto o transformado (base64, un pedazo, URL-encoded) sale en claro.

### 2. `needs`: encadenar jobs

```yaml
jobs:
  extraer:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - run: python extraer.py
  cargar:
    needs: extraer                 # espera a extraer; si falla, no corre
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - name: Cargar a Neon
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: python cargar.py
  avisar:
    needs: [extraer, cargar]
    if: failure()                  # solo si algo de arriba se cayó
    runs-on: ubuntu-latest
    steps:
      - run: echo "se cayó el pipeline"
```

Dos jobs = dos máquinas. **Un archivo que creaste en `extraer` no existe en `cargar`**: si lo necesitas, va como artefacto o los dos pasos van en el mismo job.

### 3. `if: failure()`: el aviso

Cada job tiene un estado (`success`, `failure`, `cancelled`). Por defecto, un job con `needs` se saltea si el anterior falló; con `if: failure()` lo fuerzas a correr **justo en ese caso**. Es el job más importante del pipeline: el único que te avisa sin que estés mirando.

## ⚠️ El error más común

> [!warning] Run en verde, y este archivo adentro del artefacto
> ```text
> $ unzip artefacto.zip && cat config.json
> {
>   "host": "ep-cool-name-123456.us-east-2.aws.neon.tech",
>   "password": "npg_8Xk2vQr7Tb1Zm4"
> }
> ```
> **Qué significa:** el secreto salió del almacén y quedó escrito en un archivo que cualquiera con acceso al repo puede descargar. **No hay traceback: hay un checkmark verde.** **Cómo se lee:** no busques rojo, busca el valor. Revisa cada artefacto y cada línea de log que escriba configuración; si en el log ves `***` estás bien, si en el archivo ves `npg_...` ya está expuesto. **Cómo se arregla:** no escribas secretos en archivos. Pásalos por `env`, y si de verdad necesitas un archivo, genéralo en `$RUNNER_TEMP` y no lo subas. Y aunque borres el workflow: **rota la credencial** en Neon, porque una contraseña que estuvo en un artefacto ya está comprometida.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Crea el secret `DATABASE_URL` en el repo y úsalo en un step con `env`. En el log tiene que aparecer `***` y en ningún lado tu contraseña.
2. Separa el pipeline en dos jobs (`extraer` y `cargar`) unidos con `needs`, y comprueba que si `extraer` falla, `cargar` no corre.
3. El ejercicio caro, a propósito: en `cargar`, escribe el secreto en un archivo y súbelo como artefacto. Descárgalo y mira el contenido. **Después borra ese step y rota la credencial.**

### Reto — opcional

1. Dos jobs de aviso: uno con `if: success()` y otro con `if: failure()`, con mensajes distintos. El run tiene que mostrar uno solo.
2. Que `cargar` falle si el conteo de filas no coincide con el esperado, y que el mensaje del aviso diga cuántas filas llegaron.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Hay dos jobs encadenados y el log muestra que `cargar` esperó a `extraer`
- [ ] El secreto aparece como `***` y puedes señalar el lugar exacto del log
- [ ] Puedes explicar en una frase por qué el enmascarado no te salva de un archivo

## 🔗 Para profundizar

- [Secrets](https://docs.github.com/en/actions/concepts/security/secrets) — qué enmascara y qué no
- [Using secrets in GitHub Actions](https://docs.github.com/en/actions/how-tos/security-for-github-actions/security-guides/using-secrets-in-github-actions) — environments y límites
- [Expresiones y funciones de estado](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions) — `failure()`, `always()`, `cancelled()`

## 🧭 Problemas frecuentes

- **`Unrecognized named-value: 'secrets'`** → estás usando `secrets` donde no existe (por ejemplo en el `if` de un job). Pásalo a `env` y compáralo ahí.
- **El secreto llega vacío** → el nombre no coincide (distingue mayúsculas) o está en un *environment* y el job no declara `environment:`.
- **`needs` no comparte el archivo** → son máquinas distintas. Artefacto, o un solo job con dos steps.
- **El aviso no dispara** → `if: failure()` en un step solo mira los steps del mismo job. Si falló otro job, el `if` va en el job, junto con `needs`.

## ➡️ Siguiente tema

El aviso va a dejar de ser un `echo` y va a llegar a tu teléfono: un bot de Telegram que cuenta en tres mensajes si el pipeline salió bien, con aviso o con error → [[01 · Telegram, el bot de alertas]]

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá el log con el `***`, la captura del run con la alerta y el contenido del artefacto del ejercicio 3 (con la contraseña tapada). El siguiente tema retoma lo que quede acá.
