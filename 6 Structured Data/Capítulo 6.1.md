¡Perfecto! Este capítulo sienta las bases para modelar el mundo real en código a través de "Datos Estructurados" (objetos con múltiples atributos) y "Datos Condicionales" (datos que pueden ser de diferentes tipos bajo un mismo paraguas). 

Aquí tienes la nota estructurada para tu Obsidian:

***

# Capítulo 6.1: Introducción a los Datos Estructurados

**Etiquetas:** #pyret #estructuras-de-datos #tipos-de-datos #cases #modelado
**Relacionado:** [[Datos Recursivos]], [[Tipos de Datos]]

Hasta ahora hemos usado tipos de datos simples (Number, String, Boolean) y listas. Sin embargo, en el mundo real, los datos suelen ser más complejos y tienen múltiples atributos. Para esto utilizamos **Datos Estructurados** y **Datos Condicionales**.

---

## 1. Datos Estructurados (Structured Data)
Un dato estructurado es aquel que agrupa múltiples piezas de información (llamadas **campos** o *fields*) bajo una sola entidad fija. 
*Ejemplo:* Una canción tiene un título (String), un artista (String) y un año (Number).

### Definiendo el Tipo de Dato
Para enseñar a Pyret un nuevo tipo de dato estructurado usamos la palabra clave `data`.
*Por convención, los nombres de los nuevos tipos siempre empiezan con mayúscula.*

```pyret
data ITunesSong: 
  song(name :: String, singer :: String, year :: Number) 
end
```
* **`ITunesSong`** es el nombre del nuevo Tipo.
* **`song`** es el **constructor** (la función que usamos para crear el dato).

### Creando y Anotando Instancias
Para crear datos de este tipo, usamos el constructor. Podemos guardar estos datos en variables, anotando explícitamente su tipo:
```pyret
lver :: ITunesSong = song("La Vie en Rose", "Édith Piaf", 1945)
so :: ITunesSong = song("Stressed Out", "twenty one pilots", 2015)
```

### Extrayendo Campos (Accessors)
Para acceder a un campo individual de un dato estructurado ya creado, usamos la **notación de punto (`.`)**:
```pyret
fun song-age(s :: ITunesSong) -> Number:
  2016 - s.year # Extraemos el campo 'year' usando el punto
end
```

---

## 2. Datos Condicionales (Conditional Data)
Un dato condicional representa un conjunto de opciones ("esto **O** esto **O** esto") agrupadas bajo un mismo Tipo.
*Ejemplo:* Un color de semáforo puede ser Rojo, Amarillo o Verde. Un animal del zoo puede ser una Boa o un Armadillo.

### Definiendo Variantes (Variants)
Usamos el símbolo `|` (stick) para listar cada una de las opciones o variantes.
```pyret
# Variante sin campos (como un Enum)
data TLColor:
  | Red
  | Yellow
  | Green
end

# Variante con campos estructurados
data Animal:
  | boa(name :: String, length :: Number)
  | armadillo(name :: String, liveness :: Boolean)
end
```

### Creando Instancias
Usamos los constructores específicos de cada variante, pero anotamos el tipo "paraguas" (`Animal`):
```pyret
b1 :: Animal = boa("Ayisha", 10)
a1 :: Animal = armadillo("Glypto", true)
```

---

## 3. Programando con Datos Condicionales (`cases`)
Como no sabemos qué variante exacta nos va a llegar (ej. si recibimos un `Animal`, no sabemos a priori si es `boa` o `armadillo`), usamos la expresión **`cases`** para dividir el flujo del programa.

### A. Para Variantes sin Campos
Simplemente ejecutamos código según la variante que "matchee":
```pyret
fun advice(c :: TLColor) -> String:
  cases (TLColor) c:
    | Red => "wait!"
    | Yellow => "get ready..."
    | Green => "go!"
  end
end
```

### B. Para Variantes con Campos Estructurados
Esta es la magia real de `cases`: **extrae automáticamente los campos de la variante** y los guarda en variables temporales que puedes usar directamente en la respuesta (¡sin necesidad de usar la notación de punto!).

```pyret
fun animal-name(a :: Animal) -> String:
  cases (Animal) a:
    # Se extraen los campos de la boa en las variables 'n' y 'l'
    | boa(n, l) => n 
    # Se extraen los campos del armadillo en las variables 'n' y 'l'
    | armadillo(n, l) => n 
  end
end
```
*Nota de diseño:* Los nombres de las variables en el `cases` (`n`, `l`) no tienen que ser idénticos a los definidos en el `data` (`name`, `length`). Suelen ponerse más cortos por conveniencia. Pyret los asigna por orden.

***

¡Listo! Tu bóveda de Obsidian ya está lista para seguir expandiéndose. Con estos conceptos de definición estructurada, el salto a modelar árboles de decisión (como se suele ver después) será muchísimo más sencillo.

¿Seguimos con el próximo capítulo?