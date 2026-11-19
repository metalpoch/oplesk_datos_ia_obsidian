---
tipo: anexo
modulo: "datos"
capa: referencia
herramienta: "archivos del curso"
resultado: "los cuatro archivos listos para trabajar, y el mapa de su suciedad"
tiempo-lectura: 6 min
prerrequisitos: []
tags: [anexo, datos, archivos]
---

# Los datos del curso

Esta carpeta tiene los archivos con los que trabajan todos los temas. **Cópiala dentro de tu carpeta de proyecto** (la que abres en VS Code) y trabaja siempre desde ahí: el código de las notas usa rutas como `datos/ventas.csv`.

> [!important] No edites estos archivos
> Son la fuente de verdad. Si los modificas, las cifras de las notas dejan de cuadrar y no vas a poder comprobar si tu trabajo está bien. Si necesitas trabajar sobre los datos, guarda tu resultado con otro nombre (`ventas_limpias.csv`).

## 📄 Qué hay

| Archivo | Filas | Columnas | Para qué |
|---|---|---|---|
| `ventas.csv` | 50.000 | `id_venta`, `fecha`, `producto`, `categoria`, `cantidad`, `precio_unit`, `total`, `metodo_pago`, `cliente_id` | El archivo principal. Viene sucio a propósito |
| `ventas_latin1.csv` | 50.000 | las mismas | El mismo contenido guardado en `latin-1`. Sirve para provocar el `UnicodeDecodeError` |
| `clima_caracas.csv` | 90 | `fecha`, `temp_max_c`, `lluvia_mm` | El clima de Caracas de los mismos 90 días que las ventas |
| `precios_competencia.csv` | 20 | `producto`, `precio`, `stock` | Precios de la competencia; 2 productos agotados vienen sin precio |

## 🧹 La suciedad de `ventas.csv` no es un descuido

Cada defecto está puesto para que aprendas a encontrarlo. Esto es lo que vas a tener que resolver:

| Columna | Qué tiene |
|---|---|
| `id_venta` | 200 filas duplicadas: el mismo id aparece dos veces |
| `fecha` | dos formatos mezclados (`dd/mm/aaaa` e ISO `aaaa-mm-dd`), una fecha imposible (`31/02/2024`) y una ambigua (`03/04/2024`) |
| `producto` | acentos y ñ, que se rompen si el archivo se lee con la codificación equivocada |
| `categoria` | 20 variantes de escritura para 6 categorías reales: `bebidas`, `BEBIDAS`, `Bebidas ` con espacio final |
| `cantidad` | 246 valores vacíos |
| `precio_unit` | unas 1.500 filas con coma decimal (`3,50`) en vez de punto |
| `total` | vacío donde falta `cantidad` o `precio_unit` |
| `metodo_pago` | 950 vacíos |
| `cliente_id` | 2.503 vacíos y 446 ids huérfanos (apuntan a clientes que no existen) |

## ⚖️ La invariante que te permite verificar tu trabajo

> [!warning] `total = cantidad × precio_unit`, redondeado a 2 decimales
> Vale para **todas las filas que tienen los dos valores**. Son 49.754 de las 50.000.
>
> Es la herramienta de control más útil del curso: si después de limpiar sumas `total` y sumas `cantidad * precio_unit`, los dos números **tienen que dar igual**. Si dan distinto, se colaron filas sucias y ahí está el trabajo.

## 🌡️ Sobre `clima_caracas.csv`

Tiene 90 filas, una por día, del 1 de enero al 30 de marzo de 2024 — los mismos días que cubren las ventas. Viene incluido para que puedas practicar el cruce aunque la API no responda o no tengas internet.

El tema de `httpx` te enseña a generarlo tú mismo desde Open-Meteo. Las dos versiones sirven para el mismo ejercicio: si generas la tuya, compara los totales con esta y explica la diferencia.

## 🏭 Lo que vas a producir tú

Estos archivos **no vienen**: son el resultado de tu trabajo, y por eso no están.

| Archivo | Lo produces en |
|---|---|
| `ventas_limpias.csv` | limpieza de `ventas.csv`: `fecha` como fecha, `total` numérico, categorías normalizadas |
| `ventas.parquet` | el mismo contenido en formato columnar |
| `ventas_diarias_clima.csv` | el cruce de ventas por día con el clima |

## 🔗 Relacionado

- [[Esquema de datos]] — el contrato de columnas: la única versión válida de los nombres
- [[Chuleta de errores]] — qué dice el motor cuando una columna no coincide
- [[01 · PostgreSQL, DDL y carga]] — donde `ventas.csv` se convierte en una tabla
- [[01 · Pandas, leer y perfilar]] — donde descubres que todo entró como texto
