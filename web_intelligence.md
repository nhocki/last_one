# Web Intelligence & Big Data
### Nicolás Hock Isaza - 200727001010


El curso se basó en cómo usando muchos datos (big data) se pueden encontrar y utilizar reglas para mejorar las experiencias de los usuarios en la web.

El ejemplo clásico es un buscador. Todo el mundo piensa en Google cuándo se habla de Big Data y de buscar en la web. Pero esta no es la única aplicación que tiene. Por ejemplo, las recomendaciones en Amazon son producto del mismo análisis y procesamiento de millones de pedidos y peticiones, que se procesan teniendo en cuenta la historia (reciente) del usuario en la página y se muestran productos que el usuario debería (querer) comprar.

El curso se dió en un formato de:

* Observar (Search)
* Escuchar (Machine Learning)
* Aprender (Information Extraction)
* Conectar (Reasoning)
* Predecir (Data Mining)
* Corregir (Optimization)

En cada sección se introducían nuevos conceptos y se iban conectando para al final tener una "cadena" de herramientas que posibilitan tener una aplicación web "inteligente".


## Observar y Buscar

En esta sección del curso se trataron diferentes formas de crear índices. El índice es igual al de un diccionario. Dada una palabra que estoy buscando (en el diccionario), me dice qué páginas web tenían esa palabra. El reto es poder crear un índice **muy** grande, donde la búsqueda sea rápida y hacerlo se demore un tiempo "aceptable".

Se sabe que buscar **una** palabra en un índice de _m_ palabras es `O(log m)`, pero si se tiene una búsqueda de varias palabras (_q_ palabras diferentes) es necesario armar o reunir el resultado. Si se tienen _r_ resultados parciales (al menos una de las _q_ palabras aparecen en un documento, que en total suman _r_ documentos), para construir el resultado final se necesita `O(rq)`, pues hay que ir por cada palabra en cada resultado y preguntar si está.

Una forma básica de crear un índice es la siguiente:

    class Index
      def create(D):  // D = conjunto de documentos
        for d in D:    
          for w in d:
            i = index.lookup(w)
              if i < 0:
                j = index.add(w)
                index.append(j, d.id)
              else:
                index.append(i, d.id)


Si se tienen `n` documentos, `m` palabras en el alfabeto y `w` palabras por documento en promedio:

* Cada palabra de cada documento se debe leer una vez, el orden es **al menos** `O(nw)`

Adicionalmente, mientras *cada* palabra se lee:

* Necesitamos mirar si una lista asociada a esta palabra ya existe en el índice `O(log m)`
* Se inserta la URL (id) del documento en la lista (que se crea de ser necesario). Se puede asumir que esto tiene un costo constante.

Así, la complejidad para crear este índice es: `O(nw log m)`

### Similitud vs Importancia

Cuándo se tiene una búsqueda (sobre todo de varias palabras), es posible obtener muchos resultados (suponiendo que hay entre 30 y 40 billones de páginas en Internet - Indexadas por Google). Entonces los resultados se deben presentar por similitud a la búsqueda (número de palabras que había en el documento) o por importancia de una página web (como el NYT o CNN)?

Para esto se utilizan diversos algoritmos, uno de ellos es el **Google Page Rank** que mide la importancia de una página en internet.

**Page Rank** se basa en la forma como funciona la web. Muchísimos documentos enlazados por vínculos, por el cual se pasa de un lugar a otro. Google Page Rank toma un usuario de internet aleatorio, que visita una página aleatoria y sigue enlaces aleatorios en esa página. Entonces, cuál es la probabilidad que este usuario aleatorio visite una página?

Es claro que si una página tiene **muchos** enlaces, va a tener una probabilidad mayor, lo que la hace más importante y tiene un Page Rank grande. Pero, es importante aclarar que no solo los enlaces que entran a una página son suficientes para calcular el Page Rank, pues imaginemos que haya un ciclo en la web (que hay muchos!) y el usuario se queda allí para siempre.

Google maneja esto de diferentes maneras, una de ellas es hacer un "salto" con el usuario, donde no visita un enlace de la página sino otra página aleatoria (y vuelve a empezar la navegación aleatoria).

Un problema muy grande (e interesante) que hay con Google, es que mientras más efectivo sea (la gente *siente* que encuentra lo que quería, sin esfuerzo), va perdiendo precisión. La razón es que las personas empezarán a **no** usar enlaces a páginas pues "es más fácil buscar en Google", lo que hace que las páginas nuevas (y el contenido nuevo) no sea vinculado y el Page Rank pierde precisión!

Es también importante resaltar que las personas somos muy malas para recordar hechos específicos y cada vez dependemos más y más de Google, entonces al **buscar y encontrar** (recuperar la información), le estamos dando información a Google sobre la relevancia de un documento en nuestra búsqueda. Esto es darle retroalimentación a Google, que mejora el Page Rank.


### Otros tipos de búsquedas

La búsqueda de palabras en texos es, posiblemente la más sencilla. La dificultad se obtiene por la cantidad de información, no por la forma de buscar. Pero, qué pasa si lo que buscamos no son palabras sino objetos? Por ejemplo, cómo se buscaría una imagen en una colección de imágenes? Un índice **no** es la mejor forma de hacer esto. Existe una aproximación llamada **Locality Sensitive Hashing**.

#### Locality Sensitive Hashing

La idea básica de LSH es que los objetos que "se parecen" tendrán el mismo resultado en una función de Hash `h`. Es decir:

* Si `x == y` o `x similar y`, `h(x) == h(y)` con gran probabilidad.
* Si `x != y` o `x muy-diferente y`, `h(x) != h(y)` con gran probabilidad.

La construcción de la función `h` es compleja y estaba por fuera del alcance del curso, pero se recomienda leer [Ulman and Rajamaran - Capítulo 3](http://infolab.stanford.edu/~ullman/mmds/book.pdf).

Normalmente, LSH se verá de la siguiente forma:

![LSH Common Form](http://f.cl.ly/items/0E412E2A441a1I1Q2V0c/CodeCogsEqn.gif)

Donde `k` es las veces que se repite cada función, `p` la probabilidad de que la función evaluada sea 1 (un acierto) y `b` la cantidad de funciones que se toman. Esto implica que una variación pequeña en `p`, afectará de manera muy grande el resultado.


## Escuchar

En esta sección se trabajó el tema de _información_. Se tocaron varios aspectos de teoría de la información, como "¿qué es nueva información?" y se brinda la definición de Claude Shannon en 1948: La información está relacionada a la *sorpresa*. Es decir, un mensaje que informa de algo que tiene una baja probabilidad `p` tiene más información que algo que **siempre** ocurre.

Por *escuchar* se entiende, por ejemplo, el poder diferenciar usuarios que van a visitar una página web a usuarios que van a comprar. Se "escucha" sus búsquedas y se toman decisiones respecto a estas. Ojo, es diferente a "encontrar" información. Si hay una gran probabilidad de comprar (se buscó por ejemplo 'flores rojas baratas') se puede mostrar publicidad. Pero, si la búsqueda es 'flores rojas', es posible que **no** se esté pensando en comprar.

Al tomar en cuenta esto, se introduce el concepto de "información mutua" (se definirá más tarde), al aumentar la información mutua entre un usuario (su búsqueda) y la publicidad, se aumenta la rentabilidad.

Del mismo modo, el contenido de las páginas web sirve como "búsqueda" para mostrar la publicidad. Pero entran a jugar otros factores como las palabras claves de un documento y se introduce el concepto de **Inverse Document Frequency - IDF** de una palabra. Esto es, si una palabra es muy común en la web, no es importante. Así se tiene que la palabra "*Turing*" es mejor palabra clave que "*Persona*". Pero, palabras más raras, más comunes en el documento, son aún mejores. Entonces se define la **frecuencia de un término - TF** y se puede obtener el **TF-IDF**.

**IDF** de una palabra `w` = ![IDF](http://f.cl.ly/items/0g0T3L183F2N1f1n1y3b/CodeCogsEqn.png)

Frecuencia de `w` en un documento `d` = ![TF](http://f.cl.ly/items/3Q0f00322H0u0Y0X0Y0r/CodeCogsEqn1.png)

El **TF-IDF** = ![TF-IDF](http://f.cl.ly/items/0l2e0L1C332H3E3E2j0y/CodeCogsEqn2.png)

