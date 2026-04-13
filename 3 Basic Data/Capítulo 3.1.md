# Capítulo 3.1: Primeros Pasos (Tipos, Expresiones e Imágenes)

**Etiquetas:** #pyret #conceptos-basicos #tipos-de-datos #imagenes #sintaxis
**Relacionado:** [[Composición de Funciones]], [[Contratos (Type Signatures)]]

## 1. Conceptos Básicos y Terminología
En el entorno de Pyret (CPO) interactuamos escribiendo código que la computadora evalúa.
* **Expresión:** Un cálculo escrito en notación formal que Pyret puede entender (ej. `3 + 4`, `5 * 1`). Toda expresión evalúa a un valor.
* **Valor:** Una expresión que ya no se puede calcular más; es el resultado final (ej. `8`, `"hola"`).
* **Programa:** Una secuencia de expresiones que queremos ejecutar.
* **Sintaxis:** Las reglas gramaticales del código. En Pyret:
  * Los operadores aritméticos **requieren espacios** a su alrededor: `3 + 4` (correcto), `3+4` (incorrecto).
  * Se requieren paréntesis para evitar ambigüedad en el orden de las operaciones: `(3 + 4) * 5`.

## 2. [[Tipos de Datos]] Fundamentales
Hasta ahora hemos visto tres tipos principales:
1. **Number:** Números (ej. `3`, `-20`, `1/3`, `3.95`).
2. **String:** Texto. Se escribe siempre entre comillas dobles `" "`. Es *case-sensitive* (distingue mayúsculas de minúsculas). Ej. `"red"`, `"CSCI0111"`.
3. **Image:** Elementos gráficos generados mediante funciones.

## 3. Manipulando Imágenes y [[Composición de Funciones]]
Pyret tiene funciones nativas para dibujar imágenes. La sintaxis común es `nombre-funcion(parametros)`.
* **Figuras:** `circle(30, "solid", "red")`, `rectangle(20, 10, "outline", "blue")`, `triangle(50, "solid", "purple")`.
* **Composición:** El resultado de una función puede ser la entrada de otra. Esto permite combinar imágenes complejas:
  * `rotate(45, imagen)`
  * `overlay(imagen_arriba, imagen_abajo)`
  * `above(imagen_arriba, imagen_abajo)`
  * `beside(imagen_izquierda, imagen_derecha)`

## 4. [[Contratos (Type Signatures)]] y Errores
Un **contrato** define qué tipos de datos espera recibir una función y qué tipo devolverá. Nos ayuda a leer la documentación y evitar errores de tipo.
* Notación: `::` significa "tiene el tipo de" y `->` significa "devuelve".
* Ejemplo: `circle :: (radius :: Number, mode :: String, color :: String) -> Image`

Si pasamos un tipo de dato incorrecto (ej. multiplicar una imagen por un número) o la cantidad incorrecta de argumentos, Pyret arrojará un error indicando que se violó el contrato de la función.