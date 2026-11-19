---
tipo: tema
modulo: "06 Pandas"
capa: dominio inicial
herramienta: pandas
resultado: "carga exitosa de un CSV con encoding y fechas ambiguas"
tiempo-lectura: 15 min
prerrequisitos: ["Pandas, groupby y merge"]
tags: [tema, pandas, encoding, duplicados]
---

# Pandas: limpieza difícil, encoding, fechas y duplicados

![[pandas.svg|64]]

> [!tip] En una frase, antes de empezar
> **Un archivo de texto no guarda letras: guarda números, y el `encoding` es la tabla que dice qué número es cada letra.** Si la tabla con la que se guardó no es la misma con la que lo lees, la ñ se rompe o la carga se detiene.

> [!info] Lee esto antes de empezar — 15 minutos
> No hace falta entender todo. Lee completo una vez y escribe `#duda` donde algo no te cierre; esas dudas son el arranque.
>
> **Lo que NO está acá:** la solución del caso. Se resuelve en la computadora, no leyendo.

## 🤔 Para qué sirve

El contador te manda un CSV "sacado de un Excel viejo". Lo abres y los productos dicen `PiÃ±a`. O peor: `pd.read_csv` se cae con un `UnicodeDecodeError` y el archivo tiene 50.000 filas que no puedes ni mirar. En la misma carpeta, la columna de fechas dice `03/04/2024`: nadie sabe si es 3 de marzo o 4 de abril, y el reporte mensual se mueve de un mes a otro sin que nada falle.

Después vienen los duplicados: dos filas idénticas en fecha, producto y monto. ¿Es la misma venta cargada dos veces o dos clientes que compraron lo mismo el mismo día? Equivocarse cuesta dinero en las dos direcciones: borrar de más subestima los ingresos, dejar de más los infla.

**Eso es lo que resuelve este tema:** cargar sin perder caracteres y decidir con evidencia en vez de adivinar. La regla es una sola: la decisión se escribe, con el dato que la sostiene.

> [!note] Qué no es
> Borrar la ñ o usar `errors="ignore"` no arregla el encoding: descarta bytes en silencio y te deja un archivo que parece bien hasta que alguien busca un producto. Y un `encoding` no es lo mismo que un formato de fecha: son dos decisiones distintas, y cada una se documenta aparte.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Cargar el archivo con ñ sin perder filas y verificar que los nombres llegaron bien |
| **Después** | Justificar por escrito si `03/04` es marzo o abril, con la evidencia que usaste |

El archivo de este tema viene de un Excel viejo: la ñ rompe la lectura, las fechas no dicen si son de marzo o abril, y hay filas que parecen duplicadas sin serlo. La regla es una: no adivines, deja evidencia.

## 🧠 Conceptos

1. **Encoding: los bytes no son letras** — `utf-8` y `latin-1` codifican la ñ distinto; el archivo correcto se lee sin perder ni un carácter.
2. **Fechas ambiguas: la evidencia decide** — `01/02/2024` puede ser 2 de enero o 1 de febrero; se elige con datos y se escribe el supuesto.
3. **Duplicados por llave compuesta: candidatos, no culpables** — misma fecha, mismo producto y mismo monto no siempre son la misma venta.

## 📖 Teoría

### 1. Encoding: por qué la ñ rompe todo

```python
import pandas as pd

ventas = pd.read_csv("ventas_latin1.csv", encoding="latin-1")
print(ventas["producto"].head())
print(ventas["producto"].str.contains("ñ").sum())
```

En `utf-8`, la ñ ocupa dos bytes (`0xC3 0xB1`); en `latin-1`, uno solo (`0xF1`). Si el archivo está guardado en `latin-1` y lo lees como `utf-8`, Python llega al byte `0xF1`, no puede interpretarlo y se detiene: no adivina. Con `encoding="latin-1"` lee todos los bytes, pero si el archivo era `utf-8` vas a ver "PiÃ±a" (mojibake). Si empieza con BOM, prueba `utf-8-sig`.

**Por qué importa:** un encoding equivocado o detiene la carga o, peor, carga basura sin error. Después esa basura rompe categorías, joins y conteos. Verifica una palabra conocida con ñ después de cargar.

### 2. Fechas ambiguas: `03/04` no dice nada por sí solo

```python
dia = pd.to_numeric(ventas["fecha"].str.slice(0, 2), errors="coerce")
mes = pd.to_numeric(ventas["fecha"].str.slice(3, 5), errors="coerce")
print(dia.max(), mes.max())
```

Si el primer campo llega a 15, es el día: `%d/%m/%Y`. Si el segundo llega a 15, el día es el segundo: `%m/%d/%Y`. Si ninguno pasa de 12, los dos formatos parsean todo y el código no puede decidir. Ahí entran la fuente (Venezuela usa dd/mm), el nombre del archivo, la fecha del correo, una factura o el calendario de feriados. Compara también los `NaT` de cada formato: el correcto suele parsear más filas.

**Por qué importa:** un mes y un día intercambiados mueven ventas de un mes a otro. El reporte mensual queda mal y nada falla. La decisión se escribe: qué evidencia usaste y qué pasaría si estuvieras equivocado.

### 3. Duplicados por llave compuesta: candidatos, no culpables

```python
llave = ["fecha", "producto", "total"]
candidatos = ventas[ventas.duplicated(subset=llave, keep=False)]
print(len(candidatos))
```

`duplicated(subset=...)` marca filas que repiten esa combinación; no prueba que sean la misma venta. Dos clientes pueden comprar el mismo producto por el mismo monto el mismo día. Un duplicado real necesita un identificador (`id_venta`, ticket + línea) o evidencia: misma hora, mismo pago, mismas columnas. Sin id, inspecciona los candidatos, usa una llave más estricta o conserva ambos y documenta la decisión.

**Por qué importa:** borrar ventas reales subestima los ingresos; dejar duplicados los infla. Las dos cosas son errores, y solo la evidencia decide cuál estás cometiendo.

## ⚠️ El error más común

> [!warning] `UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 42: invalid continuation byte`
> ```
> >>> pd.read_csv("ventas_latin1.csv")
> UnicodeDecodeError: 'utf-8' codec can't decode byte 0xf1 in position 42: invalid continuation byte
> ```
> **Qué significa:** pandas intentó leer el archivo como UTF-8 y encontró el byte `0xF1`, que en UTF-8 no puede aparecer solo. Ese byte es la ñ guardada en `latin-1` (la Ñ mayúscula es otro byte). **Cómo se lee:** `position 42` es el byte 42 del archivo, no el carácter 42. Mira la fila que está alrededor de esa posición: ahí está la ñ. Si el archivo viene de Excel en Windows o LATAM, lo más probable es `latin-1` o `cp1252`. **Cómo se arregla:** lee con `encoding="latin-1"` (o `"cp1252"` si hay € y comillas tipográficas), y verifica que la ñ se vea y que el conteo de filas coincida. Si el archivo es UTF-8 con BOM, usa `utf-8-sig`. Nunca "arregles" el problema borrando la ñ ni con `errors="ignore"`: eso descarta bytes en silencio.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

Los haces **en tu terminal o VS Code**, no acá. Cuando termines, pega la salida en "Mi resultado".

1. Carga el archivo con `encoding="latin-1"` y compara el conteo de filas con las líneas del archivo menos el encabezado. Cuenta los productos que contienen ñ. Después prueba sin encoding y lee el error completo. (8 min)
2. Decide el formato de fecha con evidencia: imprime el máximo del primer y del segundo campo, cuenta los `NaT` de `%d/%m/%Y` y de `%m/%d/%Y`, y escribe tres líneas con la evidencia y la decisión. (9 min)
3. Cuenta los candidatos a duplicado por `fecha`, `producto` y `total`; inspecciona 5 pares y decide cuáles son duplicados y cuáles son dos ventas. Guarda los candidatos en `candidatos_duplicados.csv`; no borres nada todavía. (8 min)

### Reto — opcional

1. Escribe `decisiones.md` con tres secciones (encoding, formato de fecha, duplicados). En cada una: evidencia, decisión y qué dato te haría cambiarla.
2. Calcula los ingresos mensuales con `%d/%m/%Y` y con `%m/%d/%Y`, y cuantifica la diferencia. Explica por qué uno de los dos resultados es insostenible.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] El archivo carga con el encoding correcto y la ñ se ve en los productos
- [ ] El conteo de filas coincide con el archivo: no se perdió ninguna en la carga
- [ ] La decisión de fecha está escrita con evidencia, no con un "porque sí"
- [ ] Los candidatos a duplicado están contados y clasificados; no borraste sin registro
- [ ] Puedes explicar en una frase por qué usaste pandas y no una hoja de cálculo

## 🔗 Para profundizar

- [Python: Unicode HOWTO](https://docs.python.org/3/howto/unicode.html) — qué es un encoding y por qué falla un byte
- [pandas: `to_datetime`](https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html) — `format` explícito y `errors`
- [pandas: `drop_duplicates`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.drop_duplicates.html) — `subset` y `keep`
- Anexo del vault: [[05 · Estadística para leer el número]] — lo que viene en el siguiente tema

## 🧭 Problemas frecuentes

- **`UnicodeDecodeError`** → prueba `encoding="latin-1"` o `"cp1252"`; verifica que la ñ se vea. No uses `errors="ignore"`.
- **La ñ se ve como `Ã±`** → leíste un archivo UTF-8 como latin-1. Cambia a `encoding="utf-8"` y vuelve a cargar.
- **Las fechas quedan en `NaT`** → el `format` no coincide con el archivo. Imprime 5 cadenas originales y cuenta los separadores antes de elegir.
- **`drop_duplicates()` borró ventas reales** → restaura desde el archivo original y vuelve a clasificar los candidatos con una llave más estricta.

## ➡️ Siguiente tema

Vas a leer el número en vez de calcularlo, empezando por entender por qué el ticket promedio miente: [[05 · Estadística para leer el número]].

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá lo que te salió (una captura, un conteo, el texto del error que no pudiste resolver). En el siguiente tema se retoma lo que quedó acá.
