# Capítulo 4.2: Procesamiento de Tablas (Data Analysis)

**Etiquetas:** #pyret #analisis-de-datos #limpieza-de-datos #visualizacion #buenas-practicas
**Relacionado:** [[Introducción a los Datos Tabulares]], [[Tipos de Datos]], [[Funciones en Pyret]]

El análisis de datos en el mundo real rara vez ocurre con datos perfectos. Los conjuntos de datos (*datasets*) suelen tener información faltante, errores de tipeo o formatos incorrectos que deben ser procesados antes de poder analizarlos.

---

## 1. Carga de Datos (`load-table`) y Sanitizadores
Pyret permite cargar tablas desde fuentes externas. Dependiendo del entorno (CPO o VSCode), se cargan desde Google Sheets o archivos CSV.

### El problema de los datos faltantes: El tipo `Option`
En las hojas de cálculo, las celdas pueden estar vacías. Pyret maneja la ausencia de datos con un tipo especial llamado `Option`:
* `none`: Significa que **no hay dato** (está vacío).
* `some(valor)`: Significa que hay un dato y lo envuelve (ej. `some("estudiante")`).
* *Nota:* A veces una celda parece vacía pero tiene espacios (`"   "`). Pyret lo leerá como texto válido si no se limpia.

### Sanitizadores (Sanitizers)
Para evitar que una tabla se cargue con tipos mixtos o datos `none` inmanejables, usamos **sanitizadores**. Estos dictan cómo leer cada columna y qué hacer si falta un dato:
* `string-sanitizer`: Convierte todo a texto y reemplaza celdas vacías por `""`.
* `num-sanitizer`: Convierte a número. **No** pone un `0` por defecto si está vacío (porque un `0` podría arruinar un promedio; si no hay número, da error o requiere un sanitizador personalizado).
* **Regla de oro:** Usa `string-sanitizer` para números que actúan como identificadores (ej. **Códigos Postales** o números de teléfono), ya que no tiene sentido hacer cálculos matemáticos con ellos.

**Sintaxis de carga:**
```pyret
include data-source # Necesario para los sanitizadores

event-data =
  load-table: name, email, tickcount, discount, zip
    source: load-spreadsheet(ssid).sheet-by-name("Data", true)
    sanitize name using string-sanitizer
    sanitize tickcount using num-sanitizer
    sanitize zip using string-sanitizer # El código postal es un String!
  end
```

### Diferencias entre Entornos y Tipos Mixtos
* **Google Sheets (CPO):** Pyret adivina el tipo de dato de la columna basándose en la *primera fila*. Si luego hay un tipo diferente (ej. un texto `"tres"` en una columna de números), arrojará un error.
* **Archivos CSV (VSCode):** Pyret lee inicialmente *todas* las celdas como texto (`String`), por lo que no da error al cargar, pero dará problemas al hacer cálculos matemáticos.
* **Buenas Prácticas:** Si una columna tiene múltiples tipos de datos mezclados por error humano, **la mejor solución es corregirlo en el archivo fuente** (haciendo una copia del original para no perder los datos crudos) antes de cargarlo en Pyret, en lugar de intentar arreglarlo con código.

---

## 2. Limpieza y Normalización (Data Cleaning)
Normalizar significa dar un formato estándar a todos los datos de una columna (ej. poner todo en mayúsculas o unificar los valores en blanco).

Se recomienda usar `transform-column` junto con funciones personalizadas para limpiar:
```pyret
fun limpiar-descuentos(str :: String) -> String:
  doc: "Unifica celdas vacías como 'none' y pasa el resto a mayúsculas"
  if (str == "") or (string-to-lower(str) == "none"):
    "none"
  else:
    string-to-upper(str)
  end
end

# Aplicando la normalización
discount-fixed = transform-column(event-data, "discount", limpiar-descuentos)
```
*Tip: Piensa en cómo se recolectó el dato (¿lista desplegable o texto libre?) para anticipar los errores (ej. errores de tipeo).*

### Resumir y Detectar Errores
* **Summarizar datos (`count`):** Una vez que una columna está normalizada, una operación muy común es contar cuántas veces aparece cada valor. La función `count(tabla, "nombre-columna")` devuelve una tabla resumen con las frecuencias.
* **Detectar errores mediante código:** Aunque no siempre podamos arreglar los errores con código, podemos escribir funciones validadoras (ej. `fun es-email-valido(str)`) y pasarlas por un `filter-with` para extraer y aislar rápidamente todas las filas que tienen datos corruptos o inválidos y revisarlas manualmente.

---

## 3. Preparación de Datos (Data Preparation)
A veces los datos están limpios, pero no en el formato que necesita nuestra pregunta de análisis.

### A. Categorización (*Binning*)
Reduce un rango amplio de valores (ej. cantidad de tickets) en una lista pequeña de categorías ("pequeño", "mediano", "grande"). Se usa `build-column` para añadir la nueva columna categórica.
```pyret
fun etiquetar-tamano(r :: Row) -> String:
  if r["tickcount"] >= 10: "grande"
  else if r["tickcount"] >= 5: "mediano"
  else: "pequeño"
  end
end
```

### B. Separación de Columnas (*Splitting*)
Separar un dato en múltiples partes (ej. separar Nombre Completo en Nombre y Apellido).
```pyret
# string-split divide un texto según el caracter indicado (ej. un espacio " ")
primer-nombre = string-split("Josie Zhao", " ").get(0)
apellido = string-split("Josie Zhao", " ").get(1)
```

### ⚠️ Computación Responsable: Representación de Nombres
Al separar columnas como el nombre, hay que tener mucho cuidado con las **suposiciones culturales**. Asumir que un nombre siempre se divide por un espacio en "Nombre" y "Apellido" es falso. Existen culturas con múltiples apellidos, apellidos que van antes del nombre, o nombres que son una sola palabra. Un mal diseño de tabla puede excluir involuntariamente a ciertas poblaciones.

---

## 4. Planes de Tareas (Task Plans)
**Nunca empieces a escribir código complejo de inmediato.** 
Si tienes un problema de datos, crea un "Plan de Tareas":
1. Toma un ejemplo de 4 a 6 filas dibujado en un papel.
2. Identifica la tabla de entrada y cómo quieres que se vea la de salida.
3. Haz un diagrama mental o escrito de los pasos intermedios (ej. 1° Filtrar -> 2° Añadir Columna -> 3° Ordenar).
4. Escribe el código paso a paso según tu diagrama.

### Encadenamiento de Operaciones (Chaining)
Para evitar crear demasiadas variables con nombres intermedios en tu código (ej. `tabla-limpia1`, `tabla-limpia2`), puedes **anidar** las transformaciones. Nombra solo la tabla inicial y el resultado final:
```pyret
cleaned-event-data =
  transform-column(
    transform-column(event-data, "discount", limpiar-descuentos),
    "delivery", normalizar-envio)
```



---

## 5. Visualización y Gráficos (Plots)
Las visualizaciones no solo sirven para el reporte final, **sirven para detectar errores** (ej. si ves una barra en un gráfico para "emall" en lugar de "email", descubriste un error tipográfico).

**Tipos de Variables:**
* **Cuantitativas:** Valores numéricos con los que tiene sentido hacer cálculos (ej. edad, precio).
* **Categóricas:** Valores fijos que agrupan características (ej. códigos postales, género, tallas).

**Gráficos comunes en Pyret:**
* **Scatterplots (Dispersión):** Relación entre *dos* variables cuantitativas.
* **Frequency Bar charts (Barras de Frecuencia):** Muestra cuántas veces aparece cada valor en una variable *categórica*. (Ej. `freq-bar-chart(tabla, "columna")`).
* **Histograms (Histogramas):** Muestra la distribución de una variable *cuantitativa* dividida en intervalos (bins).
* **Pie charts (Gráficos circulares):** Proporciones de una variable *categórica*.

---

## 6. Flujo de Trabajo en Ciencia de Datos (Resumen)
Para realizar un buen análisis, sigue este orden:
1. **Entender los datos:** ¿Qué tipo de valor va en cada columna y qué errores son posibles?
2. **Revisar y Limpiar:** Usa inspección manual, programas (filtros) y gráficos para encontrar errores. Normaliza los datos.
3. **Nombrar estratégicamente:** Guarda en variables separadas la tabla cruda (`raw-data`), la tabla limpia (`cleaned-data`) y la tabla preparada para gráficos (`analysis-data`).
4. **Preparar (Transformar):** Añade o separa columnas (`binning`, `splitting`).
5. **Analizar y Visualizar:** Aplica la estadística y gráficos correspondientes a tu pregunta, documentando qué código generó cada gráfico.


