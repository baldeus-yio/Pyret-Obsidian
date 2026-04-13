¡Excelente! Este capítulo une todo lo que hemos visto hasta ahora. Nos enseña cómo pasar de tener un solo dato a tener **colecciones** de ellos, y cómo modelar escenarios complejos del mundo real combinando listas, sets, y datos estructurados/condicionales.

Aquí tienes la nota estructurada para tu bóveda de Obsidian:

***

# Capítulo 6.2: Colecciones de Datos Estructurados

**Etiquetas:** #pyret #colecciones #listas #sets #modelado-de-datos #design
**Relacionado:** [[Introducción a los Datos Estructurados]], [[Procesamiento de Listas]], [[Datos Recursivos]]

En el mundo real, rara vez trabajamos con un solo dato estructurado. Normalmente tenemos **Colecciones** de ellos (una lista de canciones, un set de usuarios, una bandeja de entrada de correos).

* **Datos Estructurados:** Tienen un número *fijo* de partes de tipos *diferentes* (ej. título, artista, año).
* **Datos Colectivos:** Tienen un número *variable* de elementos del *mismo* tipo (ej. una lista donde todo son canciones).

---

## 1. Listas como Datos Colectivos
Las listas pueden contener cualquier tipo de valor, incluyendo nuestros propios tipos de datos estructurados (ej. `List<ITunesSong>`).
El procesamiento es idéntico a lo que ya sabemos, simplemente combinamos el `cases` de la lista con la notación de punto (`.`) para acceder a los campos del dato.

```pyret
# Encuentra la canción más antigua en una lista de canciones
fun oldest-song(sl :: List<ITunesSong>) -> ITunesSong:
  cases (List) sl:
    | empty => raise("no definido para listas vacías")
    | link(f, r) =>
      cases (List) r:
        | empty => f
        | else =>
          osr = oldest-song(r)
          if osr.year < f.year: # Comparamos los campos 'year'
            osr
          else:
            f
          end
      end
  end
end
```

**💡 Buena práctica de diseño:** Si necesitas saber la *edad* de la canción más antigua, **no** modifiques la función `oldest-song` para que devuelva un número. Déjala intacta (devolviendo el objeto `ITunesSong`) y crea una nueva función que la llame y extraiga la edad. Esto mantiene tu código modular y reutilizable.

---

## 2. Sets (Conjuntos) como Datos Colectivos
A diferencia de las listas, los Sets tienen dos reglas de oro:
1. **No tienen orden.**
2. **Ignoran los duplicados.**

Se usan para cosas como historial de páginas visitadas o listas de votantes donde el orden de llegada o si clickearon dos veces no importa.

```pyret
import sets as S
song-set = [S.set: lver, so, wnkkhs]
```

### Extrayendo elementos de un Set (`.pick`)
Como los sets no tienen orden, **no existen los conceptos de `first` y `rest`**.
Para procesar un set en Pyret usamos el método **`.pick()`**, que devuelve un tipo de dato condicional llamado `Pick`. 

Pyret extrae el elemento de forma **aleatoria** intencionalmente, para evitar que el programador dependa accidentalmente de un orden oculto.

```pyret
fun mi-set-size(s :: S.Set) -> Number:
  cases (Pick) s.pick():
    | pick-none => 0 # Equivalente a 'empty'
    | pick-some(e, r) => # 'e' es el elemento aleatorio, 'r' es el set restante
      1 + mi-set-size(r)
  end
end
```
*Nota sobre Testing:* Como `.pick()` es aleatorio, no puedes hacer un `check` que espere un elemento específico. Tus tests deben verificar propiedades (ej. "el tamaño disminuye" o "el elemento pertenece al set original").

---

## 3. Combinando todo: Modelando Aplicaciones
Las aplicaciones reales combinan los tres tipos de datos:
* **Estructurados** (campos fijos).
* **Condicionales** (variantes con `|`).
* **Colectivos** (Listas o Sets).

**Ejemplo: Modelando un Quiz de preguntas y respuestas.**
Imagina un Quiz donde hay preguntas normales y preguntas que incluyen una "pista" (*hint*).
```pyret
# 1. Definimos un Dato Condicional que a su vez es Estructurado
data Question:
  | basic-ques(text :: String, expect :: Any)
  | hint-ques(text :: String, expect :: Any, hint :: String)
end

# 2. Un Quiz completo es simplemente un Dato Colectivo de esas preguntas:
# Un quiz es un List<Question>
```
*Nota:* Usamos `Any` para `expect` porque la respuesta a una pregunta podría ser un número, un string o un booleano.

---

## 4. ⚠️ Computación Responsable: Considerar el proceso humano a desplazar
Al usar software para automatizar tareas (como un sistema que genera y corrige Quizzes para reemplazar a los profesores en los MOOCs - Cursos Online Masivos), debemos tener cuidado. 

Reducir la educación simplemente a "elegir preguntas y dar notas" ignora factores humanos vitales del proceso de enseñanza: empatía, entendimiento del contexto del alumno, y motivación. 
**Lección:** La tecnología y los algoritmos funcionan mejor cuando se diseñan para **complementar** el trabajo humano, no cuando intentan reemplazar un proceso social complejo sin entender todas sus dimensiones.

***

¡Listo! Tu nota captura a la perfección la sintaxis de los Sets, la arquitectura combinada y la importante reflexión final del autor. 

Dime cuando tengas la siguiente parte lista.