# Capítulo 5.1: De Tablas a Listas (From Tables to Lists)

**Etiquetas:** #pyret #listas #estadistica #lambda #manipulacion-de-datos
**Relacionado:** [[Procesamiento de Tablas]], [[Tipos de Datos]], [[Funciones en Pyret]]

Hasta ahora hemos procesado tablas **fila por fila**. Sin embargo, para responder preguntas estadísticas (como promedios, máximos, mínimos o conteos únicos), necesitamos operar sobre **columnas enteras**. Para lograr esto, debemos extraer las columnas de la tabla, lo que nos introduce a un nuevo tipo de dato: **Las Listas (`List`)**.

---

## 1. Extracción de una Columna
Hay dos formas de aislar una columna, y devuelven tipos de datos muy diferentes:
1. **Como Tabla:** `select-columns(tabla, [list: "nombre-col"])` devuelve una nueva tabla que contiene solo esa columna. Sigue sujeta a las reglas de las tablas.
2. **Como Lista:** `tabla.get-column("nombre-col")` extrae los valores de las celdas y los agrupa en una **Lista**.

```pyret
ventas = cleaned-data.get-column("tickcount")
# Resultado: [list: 2, 1, 5, 0, 3, 10, 3]
```

---

## 2. Características de las Listas (Datos Anónimos)
Una lista es una secuencia ordenada de elementos. Tienen dos características clave:
* **Orden y Tipo:** Los elementos tienen un orden (primero, segundo, etc.) y se espera que todos sean del mismo tipo (ej. todos números o todos strings).
* ⚠️ **Datos Anónimos:** A diferencia de una tabla que tiene un "encabezado de columna" que describe qué son los datos, **las listas no tienen nombre**. Un `3` en una lista es solo un `3` (el programa debe saber si son edades, dólares o tickets). Esto las hace flexibles y genéricas, pero requiere que el programador tenga cuidado de no malinterpretar los datos.

**Sintaxis para crear listas literales:**
```pyret
numeros = [list: 1, 2, 3]
palabras = [list: "hola", "mundo"]
lista_vacia = [list: ]
```
*Tip de diseño: Como convención, los nombres de las variables que guardan listas deben ser **plurales** (ej. `codes`, `emails`), a diferencia de los encabezados de tabla que suelen ser singulares.*

---

## 3. Operaciones Numéricas y Estadísticas
Para calcular estadísticas sobre listas de números, Pyret incluye librerías integradas (`math` y `statistics`) que debemos importar:

```pyret
import math as M
import statistics as S

tickcounts = cleaned-data.get-column("tickcount")

M.max(tickcounts)     # Devuelve el número más grande
M.sum(tickcounts)     # Suma todos los números
S.mean(tickcounts)    # Calcula el promedio (media)
S.median(tickcounts)  # Calcula la mediana
```

---

## 4. Operaciones Generales de Listas (Módulo `lists`)
Para operar y transformar listas en general, importamos el módulo de listas: `import lists as L`.

### Funciones Principales (Contratos y Tipos):
*(Nota: `List<A>` significa una lista de cualquier tipo `A`, mientras que `List<B>` representa una lista de un tipo potencialmente distinto `B`).*

* **`L.distinct :: List<A> -> List<A>`**
  Devuelve una lista solo con los valores únicos (elimina duplicados).
  `L.distinct([list: "a", "a", "b"]) # Resultado: [list: "a", "b"]`

* **`L.filter :: (A -> Boolean), List<A> -> List<A>`**
  Devuelve una lista con los elementos que cumplen una condición (devuelven `true`).
  `L.filter(lam(x): x > 5 end, [list: 2, 8, 10]) # Resultado: [list: 8, 10]`

* **`L.map :: (A -> B), List<A> -> List<B>`**
  Aplica una función a cada elemento para **transformarlo**, devolviendo una nueva lista con los resultados.
  `L.map(string-to-upper, [list: "hola"]) # Resultado: [list: "HOLA"]`

* **`L.length :: List<A> -> Number`**
  Devuelve la cantidad de elementos en la lista.

* **`L.member :: List<A>, Any -> Boolean`**
  Comprueba si un elemento específico existe dentro de la lista.

### Extracción por posición y división de Strings
Para obtener un elemento específico de una lista, usamos **`.get(índice)`** (el índice empieza en 0).
Esto es muy útil al combinarlo con la función `string-split`, que divide un texto y devuelve una lista:
```pyret
# Separar un email por el arroba y obtener el dominio (índice 1):
string-split("bonnie@pyret.org", "@").get(1) # Devuelve "pyret.org"
```

---

## 5. Funciones Lambda (`lam`) en Listas
Al igual que con las tablas, podemos usar funciones anónimas (`lam`) para no tener que crear funciones auxiliares completas con `fun` al usar `L.filter` o `L.map`.

```pyret
# Filtrar una lista usando Lambda
codigos_reales = L.filter(lam(c): not(c == "none") end, codes)
```

---

## 6. Flujo de Trabajo: Combinando Tablas y Listas
El análisis de datos real rara vez usa solo tablas o solo listas. El flujo normal combina ambos enfoques, por ejemplo:
1. Filtrar las **filas** de la tabla original según ciertas condiciones (usando `filter-with`).
2. Extraer la **columna** relevante como lista (usando `.get-column()`).
3. Calcular una **estadística** o aplicar un mapeo sobre esa lista (usando `M.max`, `L.length`, `S.mean`, etc.).

**Ejemplo:** ¿Cuántas personas con email ".org" compraron tickets?
```pyret
import lists as L

# Paso 1 y 2 combinados:
correos = cleaned-data.get-column("email")

# Paso 3: Filtrar la lista y contar
correos_org = L.filter(lam(e): string-contains(e, ".org") end, correos)
total_org = L.length(correos_org)
```
*Diseño: Existen múltiples formas de resolver el mismo problema (ej. filtrar primero la tabla entera vs filtrar la lista de correos). Elegir una depende de si necesitarás esos resultados intermedios para otras operaciones posteriores.*

