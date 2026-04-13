# Capítulo 3.4: Condicionales y Booleanos

**Etiquetas:** #pyret #booleanos #condicionales #refactorización #funciones
**Relacionado:** [[Funciones en Pyret]], [[Tipos de Datos]], [[Directorio (Entorno de Variables)]]

## 1. El Tipo de Dato `Boolean`
Un valor Booleano (`Boolean`) representa el resultado de una pregunta de verdadero o falso. Solo existen dos valores en este tipo:
* `true` (verdadero)
* `false` (falso)

### Operadores de Comparación
Cualquier expresión que compare valores devolverá un `Boolean`.
* Genéricos: `==` (igualdad), `<>` (desigualdad), `<`, `>`, `<=`, `>=`.
* **Cuidado con los Strings:** La comparación de cadenas (`String`) distingue entre mayúsculas y minúsculas (*case-sensitive*). `"a" == "A"` es `false`. El orden alfabético se basa en el estándar ASCII.

**Operadores Específicos de Tipo (Recomendado):**
Es una buena práctica usar operadores específicos para evitar errores silenciosos en Pyret si comparamos tipos incompatibles (ej. comparar un String con un Number):
* `num-equal(1, 1)`
* `string-equal("a", "b")`
* `string-contains("will.i.am", "will")` -> `true`

### Operadores Lógicos (Combinando Booleanos)
Se utilizan para evaluar múltiples condiciones a la vez:
* `and`: Devuelve `true` solo si **ambas** condiciones son `true`.
* `or`: Devuelve `true` si **al menos una** condición es `true`.
* `not(condicion)`: Invierte el valor (de `true` a `false` y viceversa).

---

## 2. Condicionales (`if / else if / else`)
La estructura condicional permite a un programa tomar decisiones basadas en preguntas (booleanos). Pyret evalúa las condiciones de **arriba hacia abajo** y ejecuta solo el bloque de la primera condición que devuelva `true`.

```pyret
if <pregunta_1>:
  <resultado_si_pregunta_1_es_true>
else if <pregunta_2>:
  <resultado_si_pregunta_2_es_true>
else:
  <resultado_si_todo_lo_anterior_es_false>
end
```
*Nota: Se pueden anidar condicionales (poner un `if` dentro de un `else`), pero a menudo es más limpio usar `else if` o combinar condiciones con `and` / `or`.*

---

## 3. Composición de [[Funciones]] y el [[Directorio (Entorno de Variables)]]
Podemos usar el resultado de una función como entrada (argumento) para otra función. Esto se puede hacer de dos formas:

1. **Composición directa:** `add-shipping(pen-cost(10, "bravo"))`
2. **Variable intermedia:** 
   ```pyret
   pens = pen-cost(10, "bravo")
   add-shipping(pens)
   ```

### ⚠️ Scope (Alcance local vs global)
* **Directorio Global:** Variables definidas fuera de funciones (como `pens`).
* **Directorio Local:** Variables definidas *dentro* de una función (como `base` o `message-cost`). Son privadas. Ninguna otra función puede "espiar" o usar variables locales de otra función. Si lo intentas, Pyret dará error de variable no definida.

---

## 4. Buenas Prácticas y Refactorización (Clean Code)

A lo largo del capítulo se enseña cómo evolucionar una función para que sea más profesional y fácil de mantener:

1. **Nunca usar `== true` o `== false`:**
   * ❌ Malo: `if is-senior == true:`
   * ✅ Bueno: `if is-senior:`
   * ❌ Malo: `if is-senior == false:`
   * ✅ Bueno: `if not(is-senior):`

2. **Evitar redundancia condicional:**
   Si una función solo debe devolver `true` o `false` basándose en una condición, **retorna la condición directamente**.
   ```pyret
   # MAL
   if age > 18: true else: false end
   
   # BIEN
   age > 18
   ```

3. **No repetir lógica (Principio DRY - Don't Repeat Yourself):**
   Si usas el mismo cálculo en múltiples partes de tu condicional, extráelo a una variable local antes del `if`. (Ver ejemplo en `buy-tickets6`).

---

## 5. Código de Referencia Sintetizado

```py
# 1. Función con múltiples condiciones (else if) y operadores lógicos (and)
fun add-shipping(order-amt :: Number) -> Number:
  doc: "Calcula los costos de envío basados en el precio del pedido"
  if order-amt <= 10:
    order-amt + 4
  else if (order-amt > 10) and (order-amt <= 30):
    order-amt + 8
  else:
    order-amt + 12
  end
end

# 2. Retornar expresiones booleanas directamente (sin usar if true else false)
fun show-ad(age :: Number, haircolor :: String) -> Boolean:
  doc: "Muestra ad si tiene entre 9 y 18 años, y pelo rosa o morado"
  ((age >= 9) and (age <= 18)) and ((haircolor == "purple") or (haircolor == "pink"))
end

# 3. Código final refactorizado usando variables locales y or (sin == true)
fun buy-tickets(count :: Number, is-senior :: Boolean) -> Number:
  doc: "Calcula el precio de los boletos. Descuento si es senior o compra > 5"
  
  base = count * 10 # Variable local para no repetir el cálculo base
  
  if is-senior or (count > 5):
    base * 0.85
  else:
    base
  end
end
```
