# Capítulo 8.3: Ejemplos, Testing y Verificación de Programas (Examples, Testing, and Program Checking)

**Etiquetas:** #pyret #testing #ejemplos #oraculos #check-vs-where #buenas-practicas
**Relacionado:** [[Testing y Ejemplos]], [[Capítulo 8.1: Funciones como Datos (Functions as Data)]]

Hasta ahora hemos usado ejemplos para entender qué debe hacer una función. Sin embargo, escribir ejemplos de forma precisa para que la computadora los evalúe tiene dos grandes ventajas: automatiza la verificación de nuestro código y **nos obliga a entender los detalles exactos del problema**. Conforme nuestros programas crecen, debemos evolucionar de usar "simples ejemplos" a crear "tests exhaustivos".

---

## 1. Ejemplos (`where:`) vs. Tests (`check:`)
Pyret es único porque separa conceptualmente los ejemplos ilustrativos de las pruebas rigurosas.

*   **Bloque `where:` (Adentro de la función):** Sirve como **documentación para humanos**. Contiene 2 o 3 casos claros y representativos que muestran cómo se usa la función y qué devuelve en casos normales.
*   **Bloque `check:` (Afuera de la función):** Sirve como **campo de pruebas para la máquina**. Aquí van los casos extraños, límites, errores (`raises`), mayúsculas/minúsculas inesperadas, etc. En el desarrollo profesional, los bloques `check:` suelen vivir en archivos completamente separados del código principal.

```pyret
fun count-uses(of-string :: String, in-list :: List<String>) -> Number:
  # ... código ...
where: # Ejemplos ilustrativos
  count-uses("pepper", [list: "pepper", "onion"]) is 1
end

check: # Pruebas exhaustivas (Casos límite)
  count-uses("ppper", [list: "pepper"]) is 0 # Fallo de ortografía
  count-uses("ONION", [list: "pepper", "onion"]) is 1 # Diferentes mayúsculas
  # ... decenas de pruebas más ...
end
```

> **Nota profesional:** La colección de tests *crece* durante el desarrollo. Cada vez que descubres un "bug" en tu código, añades un test en el bloque `check:` para asegurarte de que ese mismo error nunca vuelva a ocurrir.

---

## 2. Comparaciones Refinadas (El problema de los decimales)
A veces, la comparación exacta con la palabra clave `is` no es suficiente. Si calculamos distancias o raíces cuadradas, el resultado puede ser `1.41421356...`, una aproximación que Pyret no puede comparar de forma exacta con `1.41`.

Para resolver esto, Pyret nos permite usar una función de comparación personalizada inyectándola con el símbolo `%`:

```pyret
# 1. Definimos qué significa que un número esté "cerca" de otro
fun around(actual :: Number, expected :: Number) -> Boolean:
  num-abs(actual - expected) < 0.01
end

# 2. Usamos la sintaxis is%(funcion) en nuestros tests
check:
  num-sqrt(2) is%(around) 1.41
  distance-to-origin(point(1, 1)) is%(around) 1.41
end
```
*Traza mental:* Cuando Pyret ve `A is%(around) B`, internamente ejecuta `around(A, B)`. Si la función devuelve `true`, el test pasa exitosamente.

---

## 3. ¿Por qué fallan los tests?
Cuando la computadora reporta que un test ha fallado, solo nos está diciendo que **hay una inconsistencia**. La máquina no juzga quién tiene la culpa. Hay 3 razones principales para un fallo:

1.  **Los datos del test están mal:** Escribiste mal la expectativa (ej. `sqrt(4) is 1.75`).
2.  **El código de la función está mal:** Escribiste mal la lógica interna.
3.  **La trampa (Relaciones vs. Funciones):** El test y la función están bien, pero hay *múltiples respuestas válidas*. Por ejemplo, la raíz cuadrada de 4 puede ser `2` o `-2`. Si tu código devuelve `-2` pero tu test espera `2`, el test fallará a pesar de que la respuesta es matemáticamente correcta.

---

## 4. Oráculos de Pruebas (`satisfies`)
Para solucionar el "tercer fallo" (múltiples respuestas válidas), no debemos probar un valor exacto, sino **comprobar que el resultado cumple con una propiedad** (a esto se le llama un "Oráculo").

En lugar de usar `is`, usamos la palabra clave `satisfies`, la cual espera recibir a la derecha una función (predicado) que tome el resultado y devuelva `true` o `false`.

```pyret
# El Oráculo: una función que genera una función anónima (lam)
fun check-sqrt(original-number):
  lam(computed-root):
    # La propiedad: ¿La raíz al cuadrado nos da el número original?
    original-number == (computed-root * computed-root)
  end
end

check:
  # "La raíz de 4 debe SATISFACER la propiedad de check-sqrt(4)"
  sqrt(4) satisfies check-sqrt(4)
end
```

**¿Por qué usar `satisfies` en lugar de simplemente hacer un `is true`?**
Si escribiéramos `(sqrt(4) * sqrt(4) == 4) is true`, y fallara, Pyret solo nos diría *"Esperaba true pero obtuve false"*. 
Al usar `satisfies`, si el test falla, Pyret nos dirá exactamente **qué valor devolvió la función `sqrt(4)`** que no logró satisfacer nuestra condición, lo cual es invaluable para depurar (debuggear) el programa.