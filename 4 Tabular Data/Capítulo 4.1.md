# Capítulo 4.1: Introducción a los Datos Tabulares

**Etiquetas:** #pyret #tablas #datos-estructurados #lambda #procesamiento-de-datos
**Relacionado:** [[Tipos de Datos]], [[Funciones en Pyret]], [[Testing y Ejemplos]]

*Nota: Para usar estas funciones en Pyret, el entorno debe estar configurado con el contexto `dcic2024` (seleccionado en el menú desplegable "Choose Context" en CPO).*

## 1. Características y Creación de Tablas
Los datos tabulares (*Tidy Data*) son un tipo de [[Datos Estructurados]]. Sus características principales son:
* **Filas (Rows):** Representan ítems individuales (ej. un usuario, una canción, un mes).
* **Columnas (Columns):** Representan los atributos compartidos (ej. nombre, edad). Cada columna tiene un tipo de dato consistente.
* **Orden:** El orden de las columnas importa. Dos tablas con los mismos datos pero con distinto orden de columnas **no** son iguales.

**Sintaxis para crear una tabla literal:**
```pyret
personas = table: nombre, edad
  row: "Alicia", 30
  row: "Meihui", 40
  row: "Jamal", 25
end
```

---

## 2. Extracción de Datos (Filas y Celdas)
En Pyret, no extraemos columnas enteras de golpe, sino que navegamos por las filas (porque cada fila es una observación lógica). El índice de las filas comienza en **0**.

* **Extraer una Fila (Row):** Usamos el método `.row-n(índice)`. Esto devuelve un dato de tipo `Row`.
  `fila_marzo = shuttle.row-n(2)`
* **Extraer una Celda (Cell):** Se usan corchetes `["nombre-columna"]` sobre una fila.
  `pasajeros_marzo = fila_marzo["riders"]` o directamente `shuttle.row-n(2)["riders"]`
* **Error común (¡Cuidado con las comillas!):** Si olvidas las comillas al extraer una celda y escribes `shuttle.row-n(2)[riders]`, Pyret pensará que `riders` es una variable guardada en el directorio y arrojará un error de **"unbound identifier"** (identificador no vinculado). Los nombres de las columnas no se guardan como variables individuales.

---

## 3. Procesamiento de Tablas (Operaciones Base)
Pyret ofrece funciones integradas para analizar y modificar tablas. 
**⚠️ Regla de Oro:** Las operaciones de tablas *siempre* devuelven una tabla nueva; **nunca** modifican la tabla original (Inmutabilidad).

### A. Filtrar Filas (`filter-with`)
Devuelve una nueva tabla solo con las filas que cumplan una condición.
* **Argumentos:** (Tabla, Función que recibe un `Row` y devuelve un `Boolean`).
```pyret
fun menos-de-1000(r :: Row) -> Boolean:
  r["riders"] < 1000
end

meses_bajos = filter-with(shuttle, menos-de-1000)
```

### B. Ordenar Filas (`order-by`)
Devuelve una nueva tabla ordenada por una columna específica.
* **Argumentos:** (Tabla, "Nombre-Columna", Booleano ascendente). `true` = menor a mayor, `false` = mayor a menor.
```pyret
ordenado = order-by(shuttle, "riders", true)
mes_con_menos_pasajeros = ordenado.row-n(0)["month"]
```

### C. Añadir una Columna (`build-column`)
Crea una nueva columna calculada a partir de los datos de las otras columnas de esa misma fila.
* **Argumentos:** (Tabla, "Nombre-Nueva-Columna", Función que recibe un `Row` y devuelve el valor nuevo).
```pyret
fun calcular-sueldo(r :: Row) -> Number:
  r["horas"] * r["pago-hora"]
end

con_totales = build-column(empleados, "pago-total", calcular-sueldo)
```

### D. Transformar/Actualizar una Columna (`transform-column`)
Modifica los valores de una columna existente.
* **Argumentos:** (Tabla, "Nombre-Columna", Función que recibe el **Valor de la celda actual** y devuelve el valor actualizado). *Nota: ¡No recibe un Row, recibe un valor (ej. Number o String)!*
```pyret
fun aplicar-aumento(salario :: Number) -> Number:
  if salario < 20: salario * 1.1 else: salario end
end

empleados_felices = transform-column(empleados, "pago-hora", aplicar-aumento)
```

**💡 Buena práctica de diseño: ¿Por qué las funciones auxiliares reciben un `Row` y no el índice numérico?**
Al escribir funciones para evaluar filas (como la que usamos en `filter-with`), es mejor pasar el `Row` completo en lugar de pasar su número de posición. Esto hace que la función sea **flexible y reutilizable** para cualquier tabla que comparta ese nombre de columna. Si le pasáramos el índice numérico, estaríamos obligados a escribir el nombre de una tabla específica dentro de la función (ej. `shuttle.row-n(indice)`), atando la función a una sola tabla.

---

## 4. Funciones Lambda (Funciones Anónimas)
A veces, crear una función auxiliar entera (con `fun`) solo para usarla una vez en `filter-with` o `build-column` hace el código muy largo. Para eso usamos **`lam`** (lambda), que crea una función temporal "al vuelo".

**Sintaxis:** `lam(parametro): expresion end`

```pyret
# En lugar de crear la función 'menos-de-1000', lo hacemos en una línea:
meses_bajos = filter-with(shuttle, lam(r): r["riders"] < 1000 end)

# Añadir 10 pasajeros extra a cada mes usando lambda:
mas_pasajeros = transform-column(shuttle, "riders", lam(pasajeros): pasajeros + 10 end)
```

---

## 5. Mejores prácticas de [[Testing y Ejemplos]] con Tablas
Hacer pruebas en el bloque `where:` para funciones que devuelven tablas puede ser tedioso. Estrategias:
1. **Crear tablas de prueba pequeñas:** No uses la tabla gigante original. Crea una versión miniatura que tenga solo las columnas necesarias y los casos específicos (ej. un valor por debajo del límite y otro por encima).
2. **Escribir la tabla literal directamente en el bloque `where:`**:
   ```pyret
   where:
     dar-aumentos(tabla-test) is table: pago-hora
       row: 15 * 1.1
       row: 20
     end
   ```
3. **Construir usando funciones de filas:**
   * `tabla.empty()` crea una tabla vacía con los mismos encabezados.
   * `add-row(tabla, fila)` añade una fila extraída al final de una tabla.

4. **Testing fuera de funciones (`check:` e `is-not`):**
   Si quieres hacer aserciones (tests) de tablas en el nivel principal (fuera del bloque `where:` de una función), se utiliza la anotación `check:`. Para comprobar que dos tablas *no* son iguales (por ejemplo, si tienen distinto orden de columnas), se usa `is-not`:
   ```pyret
   check:
     tabla1 is-not tabla2
   end