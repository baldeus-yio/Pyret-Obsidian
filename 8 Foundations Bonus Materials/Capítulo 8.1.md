# Capítulo 8.1: Funciones como Datos (Functions as Data)

**Etiquetas:** #pyret #funciones-orden-superior #lambdas #streams #evaluacion-perezosa #lazy-evaluation
**Relacionado:** [[Capítulo 5.3: Datos Recursivos (Recursive Data)]], [[Procesamiento de Listas]]

Hasta ahora hemos pensado en las funciones simplemente como bloques de código que parametrizan expresiones. Sin embargo, en Pyret **las funciones son valores** (como un número o un string). Pueden pasarse como argumentos y devolverse como resultados. Esto nos permite dos "superpoderes": crear funciones que generan otras funciones (útil en matemáticas) y **suspender la evaluación** de un código para crear estructuras de datos de tamaño infinito.

---

## 1. Funciones que devuelven Funciones (Ejemplo de Cálculo)
En cálculo, la operación derivada $\frac{d}{dx}$ toma una función (ej. $x^2$) y produce *otra* función (ej. $2x$). Podemos modelar esta noción platónica directamente en código.

En lugar de calcular la derivada en un punto específico inmediatamente, escribimos una función `d-dx` que toma una función `f` y **devuelve una función anónima** (lambda) que espera recibir un valor `x`:

```pyret
fun d-dx(f):
  lam(x :: Number) -> Number:
    epsilon = 0.00001
    (f(x + epsilon) - f(x)) / epsilon
  end
end

# Uso:
fun sq(x): x * x end
d-dx-sq = d-dx(sq) # Ahora d-dx-sq es una función equivalente a 2x
```

*Traza mental:* Si intentáramos evaluar la fórmula matemática sin el `lam(x):`, Pyret nos daría un error diciendo que `x` no existe. Envolver la lógica en una función anónima nos permite decir: *"Aquí tienes la receta de la derivada, guárdala. Ya me darás la `x` más tarde"*.

---

## 2. Sintaxis Abreviada para Funciones Anónimas
Para evitar que el código se llene de las palabras `lam` y `end`, Pyret ofrece una sintaxis corta para funciones anónimas usando llaves `{}`:
`{(argumentos): cuerpo}`

*   Versión larga: `lam(x): x * x end`
*   Versión corta: `{(x): x * x}`

**Nota importante:** Si la función no recibe ningún argumento, ¡los paréntesis siguen siendo obligatorios! Escribimos `{(): ...}`. Esta sintaxis es ideal para funciones de una sola línea ("one-liners").

---

## 3. Streams: Listas Infinitas (Evaluación Perezosa)
Una lista tradicional (`List`) debe ser finita porque evalúa todos sus elementos inmediatamente. Si intentamos crear una lista de todos los números naturales, el programa entrará en un bucle infinito y colapsará.

Aquí entra el segundo superpoder de las funciones: **retrasar la ejecución**. Si ponemos el "resto" de una lista dentro de una función sin argumentos (llamada técnicamente **thunk**), ese código no se ejecutará hasta que lo llamemos explícitamente. A esta estructura se le llama **Stream** (o lista perezosa / *lazy list*).

```pyret
data Stream<T>:
  # t es una función que no recibe nada y devuelve el resto del Stream
  | lz-link(h :: T, t :: ( -> Stream<T>)) 
end
```

**Creando Streams:**
Para definir un stream infinito de "unos", usamos la palabra clave `rec` (que le dice a Pyret que la variable se usará a sí misma en su definición) y nuestra sintaxis abreviada para el thunk:

```pyret
rec ones = lz-link(1, {(): ones})
```

También podemos usar funciones recursivas normales (que no necesitan `rec`) para generar streams dinámicos, como los números naturales:

```pyret
fun nats-from(n :: Number):
  lz-link(n, {(): nats-from(n + 1)}) # El +1 solo se hace si pedimos el siguiente!
end

nats = nats-from(0)
```

---

## 4. Procesando Streams (`take` y `map`)
Dado que un Stream nunca termina, sus funciones de procesamiento tienen una peculiaridad: **¡solo tienen un caso en la plantilla!** No hay caso base (como `empty`), porque el stream siempre tiene un `lz-link`.

Para extraer información, necesitamos dos funciones clave. Nota cómo para obtener el resto, **tenemos que invocar al thunk** escribiendo `s.t()`:

```pyret
fun lz-first(s :: Stream): s.h end
fun lz-rest(s :: Stream): s.t() end # <- ¡Los paréntesis "fuerzan" la evaluación!
```

Para poder probar (`check:`) nuestros streams, necesitamos extraer una porción finita y convertirla en una lista normal usando una función `take`:

```pyret
fun take<T>(n :: Number, s :: Stream<T>) -> List<T>:
  if n == 0:
    empty
  else:
    # Combinamos el elemento actual y sacamos (n-1) elementos del resto del stream
    link(lz-first(s), take(n - 1, lz-rest(s)))
  end
end
```

---

## 5. Combinando Fuerzas (El panorama general)
El poder real aparece al combinar ambos conceptos. En lugar de usar un `epsilon` fijo para nuestra función de derivada `d-dx`, podemos generar un **Stream de valores de epsilon cada vez más pequeños** (ej. $1/10, 1/100, 1/1000...$). 

Luego, aplicamos (usando `map` para streams) nuestra función derivada sobre ese stream infinito. El resultado es un nuevo stream que nos da aproximaciones de la derivada cada vez más y más precisas, evaluadas "bajo demanda" (solo generamos más precisión cuando la necesitamos). ¡Esto delega el manejo del infinito y la memoria al propio lenguaje!