# Capítulo 3.2: Nombrando Valores (Variables y el Directorio)

**Etiquetas:** #pyret #variables #directorio #expresiones-vs-declaraciones
**Relacionado:** [[Tipos de Datos]], [[Directorio (Entorno de Variables)]]

## 1. Panel de Definiciones vs. Interacciones
* **Interacciones (derecha):** Funciona como una calculadora. Escribes una expresión y obtienes el resultado inmediato.
* **Definiciones (izquierda):** Donde escribes el código, defines variables y funciones para guardarlas y reutilizarlas. Requiere darle al botón **Run** para que el código cargue en la memoria.

## 2. Declaraciones vs. Expresiones
* **Declaración (Statement):** Da una instrucción a Pyret, pero **no devuelve un valor** en pantalla. (Ej. `width = 30`).
* **Expresión:** Realiza un cálculo y **sí devuelve un valor**. (Ej. `5 + 8` o simplemente escribir la variable `width`).

## 3. Nombres (Identificadores) vs. Strings
¡Error común de principiantes!
* Un **String** (`"puppy"`) es un dato literal de texto. Lleva comillas.
* Un **Nombre/Identificador** (`puppy`) es una referencia a un valor guardado en memoria. No lleva comillas y no puede contener espacios (en Pyret se usan guiones, ej. `red-circ`).
* Si usas un nombre que no ha sido definido previamente, Pyret arrojará un error de *"unbound identifier"* (identificador no vinculado).

## 4. El [[Directorio (Entorno de Variables)]]
Cuando presionas **Run**, Pyret lee el panel de definiciones de arriba hacia abajo y guarda los nombres y sus valores en una tabla interna llamada **Directorio**.
* Pyret **evalúa** la expresión a la derecha del `=` y guarda el **resultado final**, no la fórmula.
  * Ej: Si `width = 30` y `height = width * 3`, el directorio guardará `height -> 90`.
* **Sustitución:** Cuando usas un nombre en una expresión (ej. `beside(blue-rect, ...)`), Pyret busca el nombre en el directorio y lo sustituye por su valor asociado.
* **Inmutabilidad básica:** Si intentas redefinir un nombre que ya existe (ej. `width = 50` después de haberlo definido), Pyret arrojará un error indicando que el nombre ya está en uso en el directorio actual.

## 5. Clean Code: Uso estratégico de nombres
Nombrar resultados intermedios hace que las composiciones complejas sean más fáciles de leer y evita tener que recalcular / reescribir cosas repetidas.
```pyret
# Difícil de leer
overlay(rotate(45, triangle(30, "solid", "purple")), rectangle(60, 60, "solid", "green"))

# Código Limpio (usando nombres en el directorio local/global)
purple-tri = triangle(30, "solid", "purple")
green-sqr = rectangle(60, 60, "solid", "green")

overlay(rotate(45, purple-tri), green-sqr)
```
