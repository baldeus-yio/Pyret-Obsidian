# Capítulo 3.3: De Expresiones Repetidas a Funciones

**Etiquetas:** #pyret #funciones #DRY #testing #documentacion
**Relacionado:** [[Funciones en Pyret]], [[Principio DRY]], [[Testing y Ejemplos]]

## 1. Motivación: El [[Principio DRY]] (Don't Repeat Yourself)
Si te encuentras escribiendo la misma expresión matemática o código estructural varias veces cambiando solo un par de datos (ej. colores de banderas o pesos de diferentes astronautas), es hora de crear una **Función**.
* Las funciones permiten configurar cálculos iguales con valores concretos (argumentos) en diferentes momentos.

## 2. Anatomía de una Función (`fun`)
Una función profesional en Pyret consta de 5 partes fundamentales: nombre, parámetros (con tipos), tipo de retorno, documentación (docstring) y ejemplos (testing).

```pyret
fun pen-cost(num-pens :: Number, message :: String) -> Number:
  doc: ```Calcula el costo total: 25 centavos por bolígrafo 
       más 2 centavos por letra del mensaje```
       
  # Cuerpo de la función (Body)
  num-pens * (0.25 + (string-length(message) * 0.02))
  
where:
  # Bloque de pruebas/ejemplos
  pen-cost(3, "wow") is 3 * (0.25 + (string-length("wow") * 0.02))
  pen-cost(0, "") is 0
end
```
* **Anotaciones de Tipo (`::` y `->`):** Son opcionales pero recomendadas. Ayudan a que los mensajes de error de Pyret apunten al uso de la función y no a fallos internos confusos, y sirven como guía para otros programadores.
* **Docstring (`doc:`):** Una cadena literal que explica qué hace la función. Se usan tres acentos graves ` ``` ` si la descripción ocupa más de una línea.

## 3. Evaluando Funciones: Llamada y Sustitución
1. **Definición:** Cuando le das a *Run*, Pyret guarda el nombre de la función y su código en el [[Directorio (Entorno de Variables)]], **pero no evalúa el cuerpo todavía**.
2. **Llamada (Call):** Al usar la función (ej. `moon-weight(150)`), Pyret reemplaza la llamada por el cuerpo de la función, sustituyendo los parámetros (`earth-weight`) por los argumentos proporcionados (`150`). Luego evalúa la expresión resultante.

## 4. [[Testing y Ejemplos]] (Bloque `where:`)
* Los ejemplos actúan como **Tests Automáticos**. Cada vez que corres el programa, Pyret verifica que la salida de la función coincida con la salida escrita después de la palabra `is`.
* **Regla de oro del Testing:** NUNCA escribas el cuerpo de la función, la ejecutes y luego pegues el resultado que te dio Pyret en el bloque `where:`. Los ejemplos deben calcularse mentalmente o mostrar el desglose matemático explícito. Sirven para corroborar que tu lógica mental coincide con el comportamiento real del código.
* Asegúrate de testear "casos límite" o especiales (como pedir `0` bolígrafos o un `String` vacío `""`).
