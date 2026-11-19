---
tipo: tema
modulo: "Anexos"
capa: familiaridad
herramienta: subagentes
resultado: "bitácora de 3 afirmaciones del subagente, verificadas por ejecución, con el veredicto de cada una"
tiempo-lectura: 16 min
prerrequisitos: ["Agentes, skills"]
tags: [agentes, subagentes, ia]
---

# Agentes: subagentes, delegar y verificar

> [!tip] En una frase, antes de empezar
> **Un subagente es un agente al que le pasas una tarea acotada y te devuelve el resultado, sin que veas el camino: te entrega la respuesta, no lo que intentó ni lo que falló.**

> [!info] Antes de empezar — 16 minutos
> Léelo completo y marca `#duda` donde no te cierre. **No está acá:** la solución del ejercicio; eso se construye al hacerlo.

## 🤔 Para qué sirve

Tienes una tarea grande y se la pasas al agente con el repo entero. Tarda, elige qué archivos mirar sin decírtelo, y vuelve con un resumen prolijo: nombres de funciones, columnas, conclusiones. Suena perfecto.

Si le crees, el error viaja hasta el proyecto. Si no le crees, tienes que revisar todo igual y terminas haciéndolo tú. El problema no es la desconfianza: es que no hay un procedimiento para comprobar en un minuto si lo que dice es cierto.

**Eso es lo que resuelve la delegación bien hecha:** le pasas lo que necesita —el archivo, la pregunta, el formato de vuelta— y después verificas tres afirmaciones concretas ejecutando algo. Un `ls`, un `grep`, un `df.columns`. Nada más.

> [!note] Qué no es
> Es el mismo tipo de agente, con un encargo acotado y sin tu conversación. Y no es un oráculo: escribe con la misma seguridad cuando acierta que cuando inventa. Por eso se verifica, no se cree.

## 🎯 Qué vas a lograr

| Etapa | Al terminar, vas a poder… |
|---|---|
| **Primero** | Delegarle una tarea acotada a un subagente, pasándole solo el contexto que necesita |
| **Después** | Tomar tres afirmaciones de su respuesta, verificarlas ejecutando código y anotar cuántas eran falsas |

Con este tema cierra el bloque de IA. Lo que te llevas no es una herramienta: es un procedimiento que funciona con la que venga después.

## 🧠 Conceptos

1. **Delegar con contexto acotado** — le pasas el archivo y la pregunta, no el repo entero. Lo que no le pasas también es una decisión, y conviene saber cuál tomaste.
2. **Lo que devuelve un subagente suena bien siempre** — un resumen en prosa, seguro, ordenado. El mismo tono que usaría si estuviera en lo correcto, y eso es exactamente lo que lo hace riesgoso.
3. **La regla de las tres afirmaciones** — eliges tres cosas verificables de la respuesta y las compruebas ejecutando algo. No leyendo: ejecutando.

> Es el mismo criterio que un `JOIN` bien scopeado: le das la llave, no la tabla entera. Y el riesgo es el de un `merge` sin `validate` ([[04 · Pandas, groupby y merge]]): vuelve algo que parece bien.

## 📖 Teoría

### 1. Qué le pasas y qué no

| Sí | No |
|---|---|
| El archivo o el fragmento que importa | El repo entero |
| El objetivo en una frase | "Mejora el proyecto" |
| Qué formato quieres de vuelta | Tu historial de intentos |
| Datos públicos del ejemplo | Credenciales, nombres, teléfonos |

**Por qué importa:** si le pasas todo, no sabes sobre qué trabajó. Y si le pasas de más, tampoco sabes qué quedó expuesto del otro lado.

### 2. Por qué la respuesta parece confiable

Un subagente devuelve un texto limpio, con nombres de archivo, funciones y conclusiones. No trae las dudas que tuvo en el camino, ni los pasos que fallaron. **Ves el resultado, no el proceso** — y el resultado está escrito con la misma seguridad cuando acierta que cuando inventa.

```text
"El pipeline se arma en src/etl.py, en la función construir_ventas(),
que lee la columna total_bs y la convierte a float."
```

Suena perfecto. Puede tener tres errores: el archivo, la función y la columna. Los tres se comprueban con tres comandos, y ninguno tarda más de un segundo.

### 3. Verificar: la regla de las tres afirmaciones

De la respuesta, extrae tres afirmaciones **verificables** (un archivo existe, una columna existe, una función se llama así) y comprueba cada una:

- archivo → `ls src/etl.py`
- función → `grep -n "def construir_ventas" src/etl.py`
- columna → `df.columns`, con el bloque de abajo

```python
import pandas as pd
df = pd.read_parquet("ventas.parquet")
print([c for c in df.columns if "total" in c.lower()])   # ¿existe total_bs?
```

Anota: **verdadera / falsa / no verificable**. Si dos de tres son falsas, la respuesta entera se descarta, aunque las otras partes suenen bien.

## ⚠️ El error más común

> [!warning] `KeyError: "['total_bs'] not in index"`
> ```text
> >>> df[ ["fecha", "total_bs"] ].head()
> KeyError: "['total_bs'] not in index"
> ```
> **Qué significa:** la columna que el subagente nombró no existe. La inventó, o la vio en un CSV viejo, o la dedujo del nombre del negocio. **Cómo se lee:** pandas te dice exactamente qué nombre no encontró. Antes de usarlo, `df.columns` — y compara letra por letra, incluidos acentos y guiones bajos. **Cómo se arregla:** no renombres la columna para que la respuesta quede bien. Verifica el nombre real y vuelve a pedir la tarea con el esquema correcto en el contexto.

## 🏋️ Ejercicios

### Base — obligatorio, 25 min

1. Delega una tarea acotada: "¿qué archivo y qué función generan el CSV de ventas? Dame ruta, función y línea". Pásale el árbol de archivos y el README, **nada más**.
2. Elige tres afirmaciones de la respuesta y verifica cada una por ejecución (`ls`, `grep`, correr la función). Anota el comando y el resultado.
3. Escribe la bitácora: las tres afirmaciones, el comando que usaste, el veredicto y **cuántas eran falsas**.

### Reto — opcional

1. Pide lo mismo dos veces: una con el repo entero, otra con el contexto acotado. Compara cuántas afirmaciones inventó cada uno.
2. Convierte la verificación en un script que corra solo (que compruebe que las columnas citadas existen y falle si no). Así la próxima delegación se verifica en un comando.

El reto no es obligatorio para aprobar. Suma si explicas la decisión que tomaste.

## ✅ Criterios de logro

- [ ] Tu bitácora tiene 3 afirmaciones, 3 comandos y 3 veredictos
- [ ] Puedes decir cuántas eran falsas y cuál te habría costado más caro
- [ ] La tarea delegada tenía límites explícitos (qué archivos, qué formato)
- [ ] Puedes explicar en una frase por qué el tono seguro no es evidencia

> [!tip] Cierre del bloque de IA
> Lo que queda del curso no es el nombre de la herramienta —cambia cada pocos meses— sino el criterio: **delegar con contexto acotado y verificar lo que vuelve.** Ese procedimiento funciona con el agente que exista el año que viene.

## 🔗 Para profundizar

- [Subagents](https://code.claude.com/docs/en/sub-agents) — qué contexto recibe un subagente y qué devuelve
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — cuándo un subagente es la herramienta correcta y cuándo no
- [pytest: asserts](https://docs.pytest.org/en/stable/how-to/assert.html) — convertir una afirmación en una línea que falla sola

## 🧭 Problemas frecuentes

- **"Me devolvió un resumen sin nada verificable"** → pídele afirmaciones concretas: rutas, nombres, líneas. Si no las puede dar, no hay nada que verificar.
- **"Cito archivos que no existen"** → `ls` y `grep` primero, siempre. El subagente no tiene por qué haber leído el archivo que nombra.
- **"Verifica bien pero me costó más que hacerlo yo"** → el reto del curso no es delegar todo, es saber **cuándo** delegar. Anota el costo y decide con el número.
- **"Le pasé el repo entero y tardó muchísimo"** → contexto acotado, otra vez. Menos es más rápido y más verificable.

## ➡️ Siguiente tema

No hay nota nueva: empieza el proyecto de integración, y el pipeline completo —extraer, cargar, avisar— tiene que correr de punta a punta en menos de tres minutos.

---

## 📝 Mi resultado

> [!note] Tu espacio
> Pega acá la bitácora de las tres afirmaciones: qué dijo el subagente, qué comando usaste y cuántas eran falsas. En el siguiente tema arrancamos por lo que quedó acá.
