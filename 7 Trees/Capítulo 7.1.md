# Capítulo 7.1: Árboles (Trees)

**Etiquetas:** #pyret #arboles #estructuras-de-datos #recursividad-multiple #design-recipe #errores
**Relacionado:** [[Capítulo 5.3: Datos Recursivos (Recursive Data)]], [[Procesamiento de Tablas]]

Las tablas son excelentes para manejar datos planos (como un catálogo), pero cuando lidiamos con **relaciones jerárquicas** —como un árbol genealógico— procesar datos en tablas requiere búsquedas repetitivas y muy costosas computacionalmente. Los **Árboles** son la evolución natural de los datos recursivos: en lugar de ir en línea recta como las listas, se "ramifican", permitiéndonos navegar por las relaciones de forma directa y elegante.

---

## 1. El Límite de las Tablas y el Manejo de Errores
Imagina tener una tabla de ancestros. Si queremos encontrar los abuelos de alguien, tendríamos que buscar a la persona, extraer a sus padres, y luego volver a filtrar toda la tabla *dos veces más* para buscar a los padres de esos padres. Hacer esto para "todos los ancestros" requeriría una recursión extremadamente ineficiente.

> **Nota técnica útil (`raise`):** Al trabajar con tablas, a veces buscamos a alguien que no existe. En lugar de devolver un valor engañoso (como una cadena vacía `""`), Pyret nos permite detener el programa con un error intencional usando `raise("Mensaje")`. Para probar esto en nuestro bloque `where:`, usamos la palabra clave `raises` en lugar de `is`:
> `parents-of(ancestors, "Kathi") raises "No such person"`

Para resolver el problema de la jerarquía sin usar tablas ineficientes, necesitamos diseñar nuestro propio tipo de dato.

---

## 2. Definiendo un Árbol (Ancestor Trees)
Si observamos un árbol genealógico tradicional, ¿qué compone a una persona? Su información básica, su madre y su padre. Esto nos lleva a crear el tipo de dato `AncTree` (Ancestor Tree).

```pyret
data AncTree:
  | noInfo
  | person(
      name :: String,
      birthyear :: Number,
      eye :: String,
      mother :: AncTree,
      father :: AncTree
      )
end
```

**La clave de los árboles (Recursividad múltiple):** 
A diferencia de las listas que solo tenían un campo recursivo (`rest`), ¡nuestro `person` tiene **dos** campos recursivos! (`mother` y `father`). 
* El caso base que detiene la recursividad es `noInfo` (cuando ya no tenemos registros de los ancestros de alguien). A esto también se le llama un "nodo hoja" (leaf).

---

## 3. La Plantilla Estructural para Árboles (The Template)
Al igual que con las listas, la forma del dato dicta la forma del código. Dado que tenemos dos campos recursivos, nuestra plantilla incluirá **dos llamadas recursivas**.

**Plantilla para `AncTree`:**
```pyret
#|
fun anctree-fun(at :: AncTree) -> ???:
  cases (AncTree) at:
    | noInfo => ... # Caso base (la hoja del árbol)
    | person(n, y, e, m, f) =>
      ... n ...
      ... y ...
      ... e ...
      ... anctree-fun(m) ... # LLAMADA RECURSIVA para la rama de la madre
      ... anctree-fun(f) ... # LLAMADA RECURSIVA para la rama del padre
  end
end
|#
```

---

## 4. Procesamiento en Acción (Ejemplo: `in-tree`)
Vamos a crear una función que busque si un nombre específico existe en algún lugar del árbol genealógico. Tomamos la plantilla y la adaptamos:

```pyret
fun in-tree(at :: AncTree, name :: String) -> Boolean:
  doc: "Determina si el nombre buscado está en el árbol"
  cases (AncTree) at:
    | noInfo => false # Si no hay información, el nombre no está aquí.
    | person(n, y, e, m, f) => 
        (name == n) or in-tree(m, name) or in-tree(f, name)
  end
where:
  in-tree(anna-tree, "Anna") is true
  in-tree(anna-tree, "Ellen") is true
  in-tree(noInfo, "Ellen") is false
end
```
*Traza mental:* La función se pregunta: "¿Soy yo la persona que buscas? (`name == n`)". Si la respuesta es no, usa el operador `or` para decir: "Entonces, busca en todo el árbol de mi madre (`in-tree(m, name)`). Si tampoco está ahí, busca en todo el árbol de mi padre (`in-tree(f, name)`)". 

---

## 5. La Receta de Diseño para Árboles (The Design Recipe)
Procesar árboles usa exactamente la misma receta de diseño que aprendimos con las listas, pero adaptada a esta nueva estructura:

1. **Definir el tipo de dato:** Asegúrate de incluir la variante recursiva y un caso base/hoja (como `noInfo`).
2. **Crear ejemplos de datos:** Define variables constantes con árboles pequeños y grandes para usarlos en tus pruebas (ej. `anna-tree`).
3. **Contrato y Encabezado:** Escribe el `fun`, los parámetros y el tipo de retorno.
4. **Ejemplos (`where:`):** Piensa qué debería devolver la función para el caso base y para casos más complejos.
5. **Copiar la Plantilla:** Pega la estructura con el `cases` y las llamadas recursivas múltiples.
6. **Implementación:** Rellena los huecos combinando los datos actuales con los resultados de las ramas (usando operaciones como `+`, `and`, `or`, etc.).
7. **Probar:** Ejecuta el programa para verificar que tus ejemplos del paso 4 pasen exitosamente.