# Capítulo 8.2: Colas a partir de Listas (Queues from Lists)

**Etiquetas:** #pyret #estructuras-de-datos #colas #FIFO #tuplas #tipos-envoltorio #abstraccion
**Relacionado:** [[Procesamiento de Listas]], [[Tipos de Datos]]

Una lista estándar funciona con una lógica **LIFO** (*Last-In-First-Out* / Último en entrar, primero en salir), similar a una pila de platos. Sin embargo, en la vida real (como en la fila de un supermercado), a menudo necesitamos una estructura **FIFO** (*First-In-First-Out*), llamada **Cola** (*Queue*). En este capítulo aprenderemos cómo usar una estructura de datos nativa (Listas) para "codificar" o simular una estructura diferente (Colas).

---

## 1. El Tipo de Dato Envoltorio (Wrapper Datatype)
En lugar de crear una cola desde cero, podemos crear un nuevo tipo de dato que no hace más que **envolver** a una lista. Esto nos da seguridad de tipos y nos permite definir nuestras propias reglas.

```pyret
data Queue<T>:
  | queue(l :: List<T>) # Envuelve una lista de tipo T
end

fun mk-mtq<T>() -> Queue<T>: queue(empty) end
fun is-mtq<T>(q :: Queue<T>) -> Boolean: is-empty(q.l) end
```

**Insertar (`enqueue`):** 
Podemos decidir arbitrariamente que los elementos nuevos se añadan al principio de la lista interna. 
```pyret
fun enqueue(q, e):
  queue(link(e, q.l)) # El elemento más nuevo está al principio
end
```
*Traza mental:* Si insertamos al principio (`link`), el elemento "más viejo" (el que debería salir primero) queda empujado hasta el **final** de la lista. Esto significa que para sacarlo, tendremos que ir hasta el final.

---

## 2. El Desafío de Extraer (`dequeue`)
Para sacar un elemento de la cola necesitamos hacer dos cosas simultáneamente: obtener el valor del elemento más viejo y obtener el "resto" de la cola.
Dado que una función normalmente solo devuelve un valor, tenemos varias formas de resolver esto:

### Solución 1: Usar un tipo de retorno combinado (`Pick`)
Pyret tiene una librería integrada llamada `pick` (usada normalmente para Conjuntos/Sets) que sirve exactamente para devolver un elemento y el resto de la estructura, manejando casos vacíos de forma segura.

```pyret
include pick

fun dequeue<T>(q) -> Pick<T, Queue<T>>:
  rev = q.l.reverse() # Invertimos la lista para tener el elemento más viejo al frente
  cases (List) rev:
    | empty => pick-none # Si la cola estaba vacía
    | link(f, r) =>
      # Devolvemos el valor (f) y la nueva cola (r, que volvemos a invertir)
      pick-some(f, queue(r.reverse()))
  end
end
```

### Solución 2: Usar Tuplas (Tuples)
A veces, crear tipos de datos nuevos (como un `Dequeued` o usar `Pick`) puede sentirse exagerado para valores que solo van a existir un milisegundo antes de ser separados. Para esto, Pyret ofrece las **Tuplas**.

*   **Sintaxis de creación:** `{valor1; valor2; valor3}` (se separan por punto y coma `;`).
*   **Sintaxis de extracción (destructuring):** `{a; b} = {1; 2}` (asigna 1 a `a` y 2 a `b`).

```pyret
fun dequeue-tuple<T>(q :: Queue<T>) -> {T; Queue<T>}:
  rev = q.l.reverse()
  cases (List) rev:
    | empty => raise("No se puede extraer de una cola vacía")
    | link(f, r) =>
      {f; queue(r.reverse())} # Retornamos la tupla con el valor y la cola
  end
end

# Ejemplo de uso:
{elemento; resto-cola} = dequeue-tuple(mi-cola)
```

> **Advertencia de diseño:** Las tuplas son geniales para usos rápidos y locales, pero reducen la legibilidad y aumentan la probabilidad de errores si viajan muy lejos en tu código, porque no tienen nombres de campos descriptivos (a diferencia de los constructores de datos). ¡Úsalas con precaución!

---

## 3. Métodos en Estructuras de Datos (Avanzado)
Pyret permite que los tipos de datos tengan **métodos** integrados directamente en su definición usando la palabra clave `with: method`. Esto permite un estilo más orientado a objetos.

```pyret
data Queue<T>:
  | queue(l :: List<T>) with:
    method pick(self):
      rev = self.l.reverse()
      cases (List) rev:
        | empty => pick-none
        | link(f, r) => pick-some(f, queue(r.reverse()))
      end
    end
end
```
**El poder del Polimorfismo:** Al implementar un método `pick` estándar, nuestra `Queue` ahora comparte una "interfaz" con otras estructuras (como los Sets). Esto nos permite escribir funciones genéricas que pueden procesar indistintamente una Cola o un Set, siempre y cuando ambos respondan al método `.pick()`.