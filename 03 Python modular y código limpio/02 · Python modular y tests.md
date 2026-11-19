---
tipo: tema
modulo: "03 Python modular y código limpio"
capa: familiaridad
herramienta: Python + pytest
resultado: "paquete ordenado en src/ que corre desde dos carpetas distintas, con .env.example versionado y un test en verde"
tiempo-lectura: 15 min
prerrequisitos: ["Python funciones, errores y asserts", "Git y GitHub"]
tags: [tema, python, pytest, rutas, fundamentos]
---

# Python modular y tests: carpetas, rutas y pytest

![[python.svg|64]] ![[pytest.svg|64]]

> [!tip] En una frase, antes de empezar
> **Ordenar el código en módulos es repartirlo en varios archivos que se importan entre sí, y un test es un `assert` guardado en un archivo que se puede volver a correr cuando quieras.** `pytest` es el programa que los ejecuta todos a la vez y te dice cuáles pasan.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez, y cuando algo no te cierre escribe `#duda` en esa línea. Esas dudas son el punto de partida.
>
> **Lo que NO está acá:** la solución del ejercicio. Se construye en la práctica, paso a paso; si ya viene resuelta, la nota no sirve.

## 🤔 Para qué sirve

Tu proyecto ya son cuatro archivos y funciona. En tu máquina y desde tu carpeta. El día que lo abres desde otro lado, o se lo mandas a un compañero, aparece esto:

- `python reporte.py` funciona y `python tienda/reporte.py` no. El error dice `FileNotFoundError` y el CSV **está ahí**: lo que cambió es desde dónde lo corriste.
- La ruta `datos/ventas.csv` está escrita a mano dentro del script: el programa funciona por casualidad, porque el archivo está justo al lado.
- Cambias una función para arreglar algo y **rompes otra cosa** que funcionaba. Te enteras dos días después, cuando el informe sale mal.

**Eso es lo que resuelven los módulos y los tests.** Un **módulo** es un archivo que se deja importar, y el paquete es la carpeta: el código queda ordenado y las rutas se arman desde el propio archivo, no desde donde estés parado. Un **test** es un `assert` que corre solo: se guarda, se vuelve a correr con un comando y te avisa si tocaste algo y rompiste lo de al lado.

> [!note] Qué no es
> - **No es una revisión a mano.** Abrir el CSV y mirar que el total tenga sentido no se vuelve a correr nunca; un test queda escrito y avisa cuando algo se rompió.
> - **No prueba que el programa esté bien.** Prueba que ese caso no se rompió: el test valida lo que alguien creyó correcto al escribirlo, y si esa idea estaba mal, el verde no dice nada.
> - **`venv` no es Docker.** Aísla las librerías de ese proyecto; no aísla el sistema, ni la versión de Python, ni la base de datos, y no se comparte: cada uno arma el suyo.
> - **`.env` no es un lugar seguro.** Sirve para que la clave no entre al repo, no para protegerla: quien tiene el archivo tiene el valor, y no es sitio para datos de clientes.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Correr el reporte con `python reporte.py` desde `tienda/` y con `python tienda/reporte.py` desde la carpeta de arriba, y obtener **el mismo resultado**: la ruta del CSV sale del archivo, no de donde estás parado |
| **Después** | Correr `python -m pytest` y ver un test en verde, después de haberlo visto en rojo, con el `.env.example` versionado y el `.env` real fuera del repo |

En este tema el código deja de ser "un archivo que funciona en mi máquina" y pasa a ser una carpeta ordenada que otra persona puede correr.

## 🧠 Conceptos

1. **Un módulo es un archivo que se deja importar; el paquete es la carpeta** — `from src.limpieza import ...` funciona si corres desde la raíz; si el script vive en una subcarpeta, se lanza con `python -m`.
2. **La ruta se arma desde el archivo, no desde donde estás parado** — `Path(__file__).parent` hace que el script no dependa de tu carpeta actual; los secretos van en `.env`.
3. **Un test es un assert que corre solo** — rojo primero, verde después: el verde sin haber visto el rojo no enseña nada.

## 📖 Teoría

### 1. Módulo, import y `python -m`

```text
tienda/
├── src/          <- limpieza.py + un __init__.py vacío
├── datos/        <- ventas.csv
├── reporte.py    <- importa src/ y arma la ruta del CSV
└── tests/        <- test_limpieza.py
```

Dentro de `src/` va un archivo vacío, `__init__.py`: eso le dice a Python que la carpeta es un paquete. Y en `reporte.py`, al lado de las carpetas:

```python
from src.limpieza import total_ventas

print(total_ventas([{"total": 3.5}, {"total": 2.0}]))   # 5.5
```

El comando importa: `python reporte.py` desde `tienda/` encuentra `src/` porque la carpeta del script está en el camino. Si en cambio lo metes en `scripts/`, `python scripts/reporte.py` revienta y hay que lanzarlo con `python -m scripts.reporte`: `-m` pone la carpeta actual donde Python busca módulos, el otro comando pone la del archivo.

### 2. Rutas que no dependen de dónde estás

```python
from pathlib import Path

RAIZ = Path(__file__).parent.parent       # sube de src/ a tienda/
RUTA = RAIZ / "datos" / "ventas.csv"
print(RUTA.name, RUTA.exists())           # ventas.csv True
```

`__file__` es la ruta del archivo donde escribes, `parent` sube una carpeta y el `/` de `Path` une sin conflicto con las barras de Windows: así el script encuentra `datos/ventas.csv` lo lances desde donde lo lances. Los secretos, en cambio, no van en el código: van en un `.env` (`CLAVE=valor` por línea) y se leen con `os.getenv("TIENDA_ENTORNO", "local")`, donde el segundo valor es el default. El `.env` **no se versiona** —por eso el `.gitignore` del tema anterior— y al repo va un `.env.example` con los nombres y sin los valores.

### 3. pytest: rojo antes de verde

```python
# tests/test_ejemplo.py
def doble(n):
    return n * 2

def test_doble():
    assert doble(2) == 4
```

Parado en `tienda/`, `python -m pytest` busca archivos `test_*.py` y funciones `test_*`, corre los `assert` y te cuenta: con eso imprime `1 passed`. Si cambias el `4` por `5`, el test falla y pytest te muestra `assert 4 == 5`: lo que vino y lo que esperabas. No hay clases ni `unittest` de por medio.

**Por qué importa:** un `assert` suelto lo corres una vez; un test lo corre cualquiera y te avisa si tocaste algo y rompiste otra cosa.

## ⚠️ El error más común

> [!warning] `ModuleNotFoundError: No module named 'src'`
> ```
> Traceback (most recent call last):
>   File "scripts/reporte.py", line 1, in <module>
>     from src.limpieza import total_ventas
> ModuleNotFoundError: No module named 'src'
> ```
> **Qué significa:** Python buscó una carpeta `src` en el camino desde donde ejecutaste y no la encontró: estás parado en una carpeta donde `src` no existe. **Cómo se lee:** mira la ruta del `File`. Si dice `scripts/reporte.py` y tú hiciste `cd scripts`, ahí está el problema: el nombre del módulo (`'src'`) dice qué no encontró, no qué archivo tiene el bug. **Cómo se arregla:** vuelve a la raíz (`cd tienda`) y corre `python -m scripts.reporte`. Confirma también que `src/__init__.py` existe: sin ese archivo la carpeta no es un paquete.

Su compañero es el `FileNotFoundError: [Errno 2] No such file or directory: 'datos/ventas.csv'`: el archivo existe, pero la ruta relativa se resolvió contra la carpeta desde donde corriste. Se arregla como en el concepto 2.

Este no es un error de adorno: es el que más se repite si no lo tienes visto.

## 🏋️ Ejercicios

### Base — obligatorio, 28 min

Los haces **en tu terminal o VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. **(9 min)** Arma la estructura de la teoría (con `src/__init__.py` vacío), pasa la función que armaste en el tema de funciones a `src/limpieza.py` e impórtala desde `reporte.py`. Corre `python reporte.py` desde `tienda/`. Después mueve `reporte.py` a una carpeta `scripts/` y anota qué pasa con `python scripts/reporte.py` y por qué `python -m scripts.reporte` sí funciona; al final devuélvelo a la raíz.
2. **(8 min)** Pasa la ruta del CSV a `Path(__file__)` y corre el reporte desde `tienda/` y desde la carpeta de arriba (`python tienda/reporte.py`). Las dos salidas tienen que ser idénticas.
3. **(11 min)** Crea `.env` con `TIENDA_ENTORNO=local`, léelo con `os.getenv` y escribe `.env.example` sin el valor. Commitea el ejemplo y verifica con `git status` que el `.env` real no aparece. Después escribe `tests/test_limpieza.py`: córrelo, cámbialo para que falle y vuélvelo a verde.

### Reto — opcional

1. Haz que el `.env` sea opcional: lee la variable con default desde `src/config.py`, y corre los tests **sin** `.env` para probar que no dependen de él.
2. Escribe un test que documente el dato sucio de la tienda (la fila con `"4,20"`): `import pytest` y `with pytest.raises(AssertionError):`. El test pasa cuando la función falla como debe.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `python reporte.py` desde `tienda/` imprime el total y no tira traceback
- [ ] `python tienda/reporte.py` desde la carpeta de arriba da exactamente la misma salida
- [ ] `git status` muestra `.env.example` y **no** muestra `.env`
- [ ] `python -m pytest` termina en `passed`, y sabes cuál fue el cambio que lo puso en rojo
- [ ] Explicas en una frase la diferencia entre `python -m scripts.reporte` y `python scripts/reporte.py`

## 🔗 Para profundizar

- [Módulos y paquetes — tutorial oficial de Python en español](https://docs.python.org/es/3/tutorial/modules.html) — la sección "Paquetes" explica `__init__.py` y `-m`
- [pathlib — rutas como objetos](https://docs.python.org/es/3/library/pathlib.html) — `parent`, `/` y `exists()`, con ejemplos
- [pytest: primeros pasos](https://docs.pytest.org/en/stable/getting-started.html) — instalar y correr; la documentación está en inglés, el código no
- [python-dotenv en PyPI](https://pypi.org/project/python-dotenv/) — cómo cargar el `.env` en una línea (se instala con `pip install python-dotenv`)
- Anexo del vault: [[Chuleta de errores]]

## 🧭 Problemas frecuentes

- **`ModuleNotFoundError: No module named 'pytest'`** → pytest no está instalado en este Python: `python -m pip install pytest` y vuelve a intentar.
- **`no tests ran`** → el archivo no empieza con `test_`, o la función no se llama `test_...`. pytest no adivina: se lo dice el nombre.
- **`ModuleNotFoundError: No module named 'src'`** → estás corriendo desde adentro de `scripts/` o con `python scripts/reporte.py`. `cd tienda` y `python -m scripts.reporte`.
- **`FileNotFoundError` con la ruta del `.csv`** → la ruta es relativa a tu carpeta actual. Ármala con `Path(__file__).parent.parent` y no vuelve a pasar.

Si algo no te cierra y nada de esto lo resuelve, anota el traceback completo en "Mi resultado".

## ➡️ Siguiente tema

Vas a leer las ventas desde un archivo CSV real en vez de copiarlas a mano, y a decidir qué hacer con las filas que no se pueden sumar.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la salida de `python -m pytest`, el error de import que no pudiste resolver o la prueba de que el reporte corre desde las dos carpetas.
