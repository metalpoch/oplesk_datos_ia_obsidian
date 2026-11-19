---
tipo: guia
modulo: "00 Antes de empezar"
capa: dominio básico
herramienta: VS Code + Jupyter
resultado: "un archivo propio que se ejecuta y muestra un resultado"
tiempo-lectura: 25 min
prerrequisitos: []
tags: [inicio, entorno, vscode, terminal]
---

# Tu entorno de trabajo

![[jupyter.svg|64]] ![[python.svg|64]]

> [!tip] Resumen
> VS Code es el programa donde se escribe y se ejecuta el código. La terminal es la misma ventana, pero se le dan órdenes escribiendo. En este curso las dos están en el mismo lugar.

> [!info] Lectura previa: antes de empezar — 25 minutos
> Esta nota no enseña a programar: prepara el entorno donde se va a trabajar. Sin esto, el arranque se consume en instalaciones. Cualquier punto que no quede claro se marca con `#duda` y se revisa antes de seguir.

## 🤔 Para qué sirve

Alguien envía un archivo llamado `ventas.py` y pide ejecutarlo.

- Con un procesador de texto (Word, Google Docs) se abre como texto plano, con el formato roto, y al guardar se corrompe.
- Con el Bloc de notas se puede leer, pero no ejecutar.
- Con doble clic se abre una ventana negra que se cierra de inmediato.

Falta una herramienta que permita escribir, ejecutar y ver el resultado en el mismo lugar. Eso es un **editor con terminal integrada**, y en este curso es VS Code.

Hay un segundo punto que suele quedar implícito: un archivo de código no se ejecuta solo. Necesita un programa que lo interprete (Python) y una instrucción que indique qué archivo ejecutar. Esa instrucción se escribe en la terminal.


## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| Primero | Dejar el editor instalado y abrir la carpeta del proyecto |
| Después | Correr tu primer archivo `.py` y ver la salida en la terminal |
## 🧠 Qué es cada cosa

| Componente | Función | Se instala |
|---|---|---|
| **VS Code** | Editor: donde se escribe el código y se ve el resultado | Sí |
| **Python** | Intérprete: el programa que ejecuta el código | Sí |
| **Terminal** | Interfaz de texto para dar órdenes: ejecutar, instalar, moverse entre carpetas | No, ya viene en el sistema |

La terminal se abre dentro de VS Code. No son dos aplicaciones: es un panel del editor.

## 🧭 Qué no es

- **No es un cuaderno de apuntes.** Para teoría y notas se usa Obsidian (este vault). VS Code es para ejecutar.
- **Python no es el editor.** Es el intérprete. Se puede ejecutar Python sin VS Code; lo que falta en ese caso es la comodidad de ver el código y la salida juntos.
- **La terminal no es una aplicación aparte.** En este curso es un panel dentro del editor.

## 🗺️ El mapa de VS Code

La primera vez que se abre, la cantidad de paneles desconcierta. Son cinco zonas:

```
┌──┬──────────────────────────────────────────┐
│  │  pestañas:   ventas.py  ×                │
│B │                                          │
│a │  EDITOR                                  │
│r │  (aquí se escribe el código)             │
│r │                                          │
│a ├──────────────────────────────────────────┤
│  │  TERMINAL INTEGRADA                      │
│  │  (aquí se ejecuta: python ventas.py)     │
└──┴──────────────────────────────────────────┘
     barra de estado: intérprete · rama · errores
```

- **Barra de actividad** (izquierda): los iconos. El primero es el explorador de archivos (`Ctrl+Shift+E`). Debajo aparecen búsqueda, control de versiones y extensiones.
- **Editor** (centro): el archivo abierto, con pestañas en la parte superior.
- **Terminal integrada** (abajo): se abre y se cierra con `Ctrl + ñ` (en teclados en inglés, `Ctrl + backtick`). Es la terminal del sistema, abierta dentro del editor.
- **Barra de estado** (abajo): a la derecha indica qué intérprete de Python está seleccionado. Si no indica ninguno, no hay intérprete elegido y el código no se ejecuta.
- **Paleta de comandos** (`Ctrl+Shift+P`): el buscador de acciones. Si no se encuentra una función del editor, está ahí.

### Atajos que conviene memorizar

| Atajo | Acción |
|---|---|
| `Ctrl+Shift+P` | Paleta de comandos: busca cualquier acción del editor |
| `Ctrl + ñ` | Abre y cierra la terminal |
| `Ctrl+S` | Guarda |
| `Ctrl+Shift+E` | Explorador de archivos |
| `Shift+Enter` | En un notebook: ejecuta la celda y pasa a la siguiente |

## 📓 Notebooks y scripts

| | **Script** (`.py`) | **Notebook** (`.ipynb`) |
|---|---|---|
| Ejecución | Completa, de arriba abajo | Celda por celda |
| Resultado | En la terminal | Debajo de cada celda |
| Uso | Código que debe ejecutarse solo | Explorar datos y revisar resultados |
| En este curso | A partir del código modular y los tests | En los primeros temas |

Los dos son Python y los dos se abren en VS Code. La diferencia es la forma de ejecutarlos.

> [!warning] Un notebook recuerda lo que se ejecutó, no el orden en que está escrito
> Si las celdas se ejecutan en desorden, el resultado en pantalla puede no corresponder al código visible. Cuando una cifra no cuadre, se reinicia el kernel y se ejecuta todo desde arriba.

## ⚙️ Cómo se ejecuta cada cosa

**Un script**

1. Abrir la carpeta del proyecto: `Archivo → Abrir carpeta`.
2. Abrir la terminal con `Ctrl + ñ`.
3. Escribir `python ventas.py` y presionar Enter.

**Un notebook**

1. Abrir el archivo `.ipynb` desde el explorador.
2. Arriba a la derecha, seleccionar el intérprete de Python.
3. Escribir en una celda y presionar `Shift + Enter`.

## 🖥️ La terminal: cinco comandos

| Comando | Acción |
|---|---|
| `pwd` | Indica la carpeta actual |
| `ls` (Mac/Linux) · `dir` (Windows) | Lista el contenido de la carpeta |
| `cd carpeta` · `cd ..` | Entra a una carpeta · vuelve a la anterior |
| `python archivo.py` | Ejecuta un script |
| `pip install paquete` | Instala una librería |

> [!tip] La mayoría de los errores de terminal son de ubicación
> Cuando aparece `No such file or directory` o `can't open file`, lo habitual es que la terminal esté en otra carpeta. Conviene ejecutar `pwd` antes de modificar el comando.

## 📦 Instalación

- [ ] **VS Code** desde [code.visualstudio.com](https://code.visualstudio.com)
- [ ] **Python 3.12 o superior** desde [python.org](https://www.python.org/downloads/). En Windows, marcar **«Add Python to PATH»** en la primera pantalla del instalador.
- [ ] En VS Code, instalar las extensiones **Python** y **Jupyter** (de Microsoft).
- [ ] Verificar en la terminal: `python --version` y `pip --version`.
- [ ] Abrir la carpeta del curso: `Archivo → Abrir carpeta`.

> [!warning] El error más frecuente del curso
> `python no se reconoce como un comando...` en Windows significa que Python no quedó en el PATH. La solución es reinstalar y marcar **«Add Python to PATH»**. No hay otro camino.

## ✅ Comprobación del entorno

Si esto funciona, el entorno está listo. Guardar como `prueba.py` y ejecutar con `python prueba.py`:

```python
import pandas as pd

datos = pd.DataFrame({"categoria": ["bebidas", "panaderia"], "total": [3.5, 2.0]})
print(datos.groupby("categoria")["total"].sum())
```

La salida esperada es:

```
categoria
bebidas      3.5
panaderia    2.0
Name: total, dtype: float64
```

Si falla `import pandas`, falta instalar la librería: `pip install pandas`.

## 🧭 Problemas frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `python no se reconoce...` | Python instalado sin PATH | Reinstalar marcando la casilla |
| VS Code no encuentra Python | No hay intérprete seleccionado | `Ctrl+Shift+P` → `Python: Select Interpreter` |
| `pip` no se reconoce | `pip` fuera del PATH | `python -m pip install pandas` |
| La terminal abre en otra carpeta | La ruta por defecto del editor | `pwd`, y `cd` hasta el proyecto |
| El notebook no ejecuta | No hay kernel seleccionado | Elegir `Python 3.x` arriba a la derecha |

## ☂️ Alternativa: Colab en el navegador

Si el equipo no permite instalar, o la instalación se extiende demasiado, [colab.research.google.com](https://colab.research.google.com) abre un notebook en el navegador sin instalar nada. Es el mismo archivo `.ipynb`.

Sirve para los primeros temas. A partir del código modular, los tests y git se necesita VS Code local: Colab no tiene terminal real ni sistema de archivos propio.

> [!warning] Colab no es una alternativa a Jupyter
> Es una versión **dependiente**: el código se ejecuta en servidores de Google, así que requiere conexión estable y una cuenta. Al cerrar el notebook, los archivos se pierden si no se guardaron en Drive. Se usa como alternativa, no como base del curso.

## 🔗 Para profundizar

- [Documentación de VS Code](https://code.visualstudio.com/docs) — sección «Get Started»
- [Atajos de teclado](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf) — una hoja para imprimir
- [Jupyter en VS Code](https://code.visualstudio.com/docs/datascience/jupyter-notebooks) — notebooks paso a paso
- [Tutorial de Python en español](https://docs.python.org/es/3/tutorial/) — para consultar

## ➡️ Siguiente tema

[[04 · Listas y diccionarios]] — primer tema de contenido. Con el entorno listo, se empieza por el contenido y no por la instalación.

---
![[setup-meme.jpg]]