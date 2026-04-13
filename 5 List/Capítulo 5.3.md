¡Excelente! Este capítulo marca un punto de inflexión muy importante. Pasamos de "adivinar" el código usando ejemplos (como en el capítulo anterior) a **derivar la estructura del código directamente desde la forma de los datos**. Además, introduce la famosa "Receta de Diseño" (Design Recipe).

Aquí tienes la nota lista para tu Obsidian:

***

# Capítulo 5.3: Datos Recursivos (Recursive Data)

**Etiquetas:** #pyret #recursividad #estructuras-de-datos #design-recipe #plantillas
**Relacionado:** [[Procesamiento de Listas]], [[Tipos de Datos]], [[Testing y Ejemplos]]

Las listas integradas en Pyret son en realidad un tipo de **dato condicional**. Entender cómo se construyen desde cero nos permite crear nuestras propias estructuras de datos de tamaño arbitrario o infinito.

---

## 1. Definiendo Datos Recursivos
Para entender cómo funcionan las listas por debajo, podemos crear nuestra propia versión: una lista estrictamente de números (`NumList`).

```pyret
data NumList:
  | nl-empty
  | nl-link(first :: Number, rest :: NumList)
end
```

**La clave de la recursividad:** Observa el campo `rest`. Su tipo es `NumList`, ¡el mismo tipo de dato que estamos definiendo! 
* Los datos definidos en términos de sí mismos se llaman **Datos Recursivos**.
* Esto funciona porque siempre hay un caso base que no es recursivo (`nl-empty`), lo que permite que la construcción de la lista termine en algún momento.

---

## 2. La Plantilla Estructural (The Template)
El mayor descubrimiento de este capítulo es que **la forma de los datos dicta la forma del código**. 
Para cualquier dato recursivo, podemos derivar una "plantilla" universal. Si escribes una función para procesar ese dato, *siempre* empezarás copiando y pegando esta plantilla.

**Reglas para crear la plantilla:**
1. Usa `cases` para separar las variantes del dato.
2. En cada caso, extrae todos sus campos (ej. `first` y `rest`).
3. **Paso mágico:** Si un campo es recursivo (es del mismo tipo, como `rest`), haz una **llamada recursiva a la función misma** pasándole ese campo.

**Plantilla para `NumList`:**
```pyret
#|
fun num-list-fun(nl :: NumList) -> ???:
  cases (NumList) nl:
    | nl-empty => ... # Caso base
    | nl-link(first, rest) =>
      ... first ...
      ... num-list-fun(rest) ... # LLAMADA RECURSIVA sobre el campo recursivo
  end
end
|#
```

---

## 3. Procesamiento en Acción (Ejemplo)
Si queremos escribir una función `contains-3` (que devuelva `true` si la lista contiene el número 3), simplemente tomamos la plantilla y rellenamos los huecos `...`:

```pyret
fun contains-3(nl :: NumList) -> Boolean:
  cases (NumList) nl:
    | nl-empty => false # Si está vacía, no tiene el 3
    | nl-link(first, rest) =>
      if first == 3:
        true # Lo encontramos en el primer elemento
      else:
        contains-3(rest) # No está aquí, buscamos en el resto (usando la llamada recursiva)
      end
  end
end
```
*Traza mental:* La función revisa el primer elemento. Si no es lo que busca, "pasa la pelota" a la misma función, pero entregándole el resto de la lista, hasta llegar a `nl-empty`.

---

## 4. La Receta de Diseño (The Design Recipe)
Uniendo todo lo que hemos aprendido, los programadores utilizan una metodología formal de 6 pasos para diseñar funciones sobre datos recursivos de manera infalible (adaptado de *How to Design Programs*):

1. **Contrato y Encabezado:** Escribe el nombre de la función, los tipos de entrada y el tipo de salida. (Sirve como guía para saber qué devuelve la llamada recursiva).
2. **Ejemplos Ilustrativos (`where:`):** Escribe ejemplos usando datos concretos. Escribe ejemplos secuenciales (donde la entrada de uno sea el `rest` del siguiente) para ver el patrón.
3. **La Plantilla:** Copia la plantilla correspondiente a la definición del tipo de dato que estás procesando (como la de `NumList` arriba).
4. **Implementación:** Adapta la plantilla. Usa los ejemplos del Paso 2 para descubrir cómo rellenar los huecos y combinar el `first` con el resultado de la llamada recursiva.
5. **Ejecutar Ejemplos:** Corre el programa para comprobar que los ejemplos básicos del Paso 2 pasan correctamente.
6. **Tests Exhaustivos:** Ahora que la función básica sirve, añade pruebas más complejas y detalladas en tu bloque `where:` o `check:` (casos límite, errores, etc.) para ganar confianza total en el código.

***

¡Ya lo tienes! Si dominas esta "Receta de Diseño" y el uso de las "Plantillas", cualquier estructura de datos por compleja que sea (árboles, grafos, etc., que se ven más adelante en ciencias de la computación) se procesará exactamente con la misma lógica.

Envíame la siguiente parte cuando estés listo.