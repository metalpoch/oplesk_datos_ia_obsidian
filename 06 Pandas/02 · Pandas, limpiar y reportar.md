---
tipo: tema
modulo: "06 Pandas"
capa: dominio inicial
herramienta: pandas
resultado: "ventas_limpias.csv + reporte de cambios"
tiempo-lectura: 15 min
prerrequisitos: ["Pandas, leer y perfilar"]
tags: [tema, pandas, limpieza]
---

# Pandas: limpiar y reportar sin perder la cuenta

![[pandas.svg|64]]

> [!tip] En una frase, antes de empezar
> **Limpiar datos es transformarlos dejando rastro: cada fila que cambias o descartas queda anotada con su motivo y su conteo.** No es borrar lo que molesta: es poder explicar qué hiciste.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y escribe `#duda` donde algo no te cierre; esas dudas son el arranque.
>
> **Lo que NO está acá:** la solución del reporte. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

Limpias el archivo, entregas el informe y alguien pregunta: "¿y las 2.188 filas que faltan?". No lo sabes. Revisas, y en algún paso entre el CSV y el resultado se perdieron filas que no puedes nombrar. El número está, el total cambió, y no hay forma de saber en qué línea pasó.

Peor cuando la limpieza se hace a mano en Excel: se borran las filas raras sin dejar copia, y tres meses después alguien busca la venta de un cliente que no aparece. Nadie sabe si estaba mal o si se fue en un paso que nadie documentó.

**Eso es lo que resuelve este tema:** limpiar como una transformación con reporte. Entraron 50.000, salieron 47.812, y cada una de las 2.188 que no pasaron tiene un motivo escrito y un conteo que lo respalda.

> [!note] Qué no es
> Arreglar el archivo a mano deja un resultado que no se puede repetir ni auditar. Acá el archivo original no se toca: el script lee, transforma, escribe uno nuevo y guarda aparte las filas que descartó.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Dejar la columna de fechas como `datetime` y las categorías reducidas a 6 valores canónicos |
| **Después** | Producir `ventas_limpias.csv` y un reporte que diga cuántas filas entraron, cuántas salieron y por qué |

Limpiar no es borrar: es transformar dejando rastro. En este tema cada fila que sale del dataset tiene que tener un motivo escrito y un conteo que lo respalde.

## 🧠 Conceptos

1. **Normalizar texto** — `strip`, `lower` y un mapa a categorías canónicas: "Bebidas", "bebidas" y "BEBIDAS" son la misma cosa para el negocio.
2. **Convertir fechas es una decisión, no una línea** — `to_datetime` con `errors="coerce"` convierte lo que no entiende en `NaT`; hay que contar cuántos y mirar cuáles.
3. **Reportar el delta** — filas de entrada, filas de salida y el motivo de cada descarte. Nunca borres sin contar.

## 📖 Teoría
### 1. Categorías: de tres strings a un valor canónico

```python
import pandas as pd
categorias = ventas["categoria"].str.strip().str.lower()
print(categorias.value_counts())
mapa = {"bebidas": "Bebidas", "bebida": "Bebidas"}
ventas["categoria"] = categorias.replace(mapa)
```

`.str` trata la columna como texto; `.strip()` quita espacios al inicio y al final; `.lower()` unifica mayúsculas. `replace(mapa)` cambia solo las claves que conoce. Lo que quede en minúsculas después de `value_counts()` es tu lista de pendientes: no lo borres, decídelo con el negocio. Las 6 categorías canónicas se cierran en la práctica con esa lista a la vista.

**Por qué importa:** si "Bebidas" y "bebidas" cuentan por separado, cada `groupby` y cada gráfico parte la misma categoría en dos. El total de esa categoría parece más bajo y nadie sabe por qué.

### 2. Fechas: convertir, o marcar y contar

```python
fechas = pd.to_datetime(ventas["fecha"], format="mixed", dayfirst=True, errors="coerce")
print(fechas.isna().sum())
ventas["fecha"] = fechas
```

El archivo mezcla ISO (`2024-03-16`) con dd/mm/aaaa (`15/03/2024`). `format="mixed"` parsea cada elemento; `dayfirst=True` resuelve `03/04` como 3 de abril; `errors="coerce"` convierte lo que no puede parsear en `NaT` en vez de lanzar el error. Coerce es un cinturón de seguridad, no un arreglo: esconde la fila mala. Antes de asignar, cuenta los `NaT` y mira las cadenas originales de esas filas.

**Por qué importa:** una fila con `NaT` desaparece de un filtro o de un `groupby` por fecha sin hacer ruido. Si no la contaste, tu serie diaria perdió un día y no lo sabes.

### 3. El delta: entraron, salieron y por qué

```python
reporte = {
    "entraron": len(crudo),
    "salieron": len(limpio),
    "fecha_invalida": int(fechas.isna().sum()),
}
print(reporte)
```

`crudo` es el DataFrame antes; `limpio`, el de después. La diferencia entre los dos tiene que ser explicable por los motivos que anotaste: fechas inválidas, duplicados, filas fuera de rango. Si los números no cuadran, perdiste filas en un paso que no documentaste. Guarda las descartadas en `descartadas.csv` para que el conteo se pueda auditar.

**Por qué importa:** el reporte es lo que permite defender la limpieza frente a otra persona. "Quedaron 47.812" no es un reporte; por ejemplo, "entraron 50.000, salieron 47.812, 1.940 por fecha inválida y 248 duplicadas" sí.

## ⚠️ El error más común

> [!warning] `ValueError: time data "15/03/2024" doesn't match format specified`
> ```
> >>> pd.to_datetime(ventas["fecha"])
> ValueError: time data "15/03/2024" doesn't match format "%Y-%m-%d", at position 1.
> ```
> **Qué significa:** pandas eligió un formato a partir de las primeras fechas y se topó con una que no encaja. No adivina fila por fila: se detiene. En pandas 2.x el mensaje nombra el formato que intentó; en versiones viejas dice `format specified`. **Cómo se lee:** `at position 1` es el segundo elemento (se cuenta desde 0). Mira esa cadena y las de al lado: el 80 % de las veces es la mezcla entre ISO y dd/mm/aaaa. **Cómo se arregla:** decide qué formatos hay y pásalos explícitos: `format="mixed", dayfirst=True`. Si además hay basura real, `errors="coerce"` y cuenta los `NaT`. No uses coerce para tapar un formato que no entendiste.
>
> **El silencio peligroso:** `errors="coerce"` no arregla el formato, solo convierte a `NaT` lo que no encaja. Sin `format="mixed"`, pandas puede inferir dd/mm y convertir todas las fechas ISO en `NaT`. Cuenta los `NaT` antes y después: si el número sube, coerce se comió filas.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Normaliza las 6 categorías: imprime `value_counts()` antes y después, escribe el mapa y anota qué variantes uniste y cuáles quedaron pendientes. No borres ninguna fila todavía. (10 min)
2. Convierte los dos formatos de fecha con `format="mixed", dayfirst=True, errors="coerce"`. Cuenta los `NaT` e imprime 5 cadenas originales que quedaron así; explica si eran fechas malas o un formato no soportado. (8 min)
3. Arma el reporte del delta: filas que entraron, filas que salieron, fechas inválidas y duplicados. Guarda `descartadas.csv` y `ventas_limpias.csv`, y haz que los números cuadren. (7 min)

### Reto — opcional

1. Corre la limpieza dos veces seguidas y demuestra que el resultado es idéntico: mismas filas y mismas categorías. Si cambia, tu mapa no es idempotente.
2. Agrega un `assert` que falle si queda una categoría fuera de las 6 canónicas, e imprime las culpables antes de fallar.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] `ventas_limpias.csv` abre y `fecha` es de tipo `datetime64[ns]`
- [ ] Quedan exactamente las 6 categorías canónicas que definiste
- [ ] El reporte dice filas de entrada, filas de salida y el conteo de cada motivo
- [ ] Puedes explicar por qué `errors="coerce"` es peligroso si no cuentas los `NaT`
- [ ] Puedes explicar en una frase por qué usaste pandas y no una hoja de cálculo

## 🔗 Para profundizar

- [pandas: `to_datetime`](https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html) — `format`, `dayfirst` y `errors`
- [pandas: working with text data](https://pandas.pydata.org/docs/user_guide/text.html) — `.str.strip()`, `.str.lower()` y reemplazos
- Anexo del vault: [[04 · Pandas, groupby y merge]] — lo que viene en el siguiente tema

## 🧭 Problemas frecuentes

- **`UserWarning: Parsing dates in %d/%m/%Y format when dayfirst=False`** → pasa `dayfirst=True` o un `format` explícito; el warning te está diciendo que asumió mal.
- **Aparecen `NaT` que no esperabas** → imprime las cadenas originales de esas filas. Casi siempre es el otro formato de fecha, no un dato vacío.
- **`AttributeError: Can only use .str accessor with string values`** → la columna tiene números o tipos mezclados. Revisa `dtype` y conviértela a texto antes de usar `.str`.
- **Las categorías siguen partidas** → normaliza espacios y mayúsculas antes de mapear; compara el `value_counts()` de antes con el de después.

## ➡️ Siguiente tema

Vas a agrupar esas ventas por fecha y cruzarlas con el clima, con una fila por fecha y un `validate=` que lo demuestre: [[04 · Pandas, groupby y merge]].

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (una captura, un conteo, el texto del error que no pudiste resolver). En el siguiente tema se retoma lo que quedó acá.

---
![[meme-analisis-datos.jpg]]