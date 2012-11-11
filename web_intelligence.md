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

Del mismo modo, el contenido de las páginas web sirve como "búsqueda" para mostrar la publicidad. Pero entran a jugar otros factores como las palabras claves de un documento y se introduce el concepto de **Inverse Document Frequency - IDF** de una palabra. *Es inverso pues no se buscan documentos desde palabras claves (que sería la búsqueda), sino palabras claves a partir de un documento*. Esto es, si una palabra es muy común en la web, no es importante. Así se tiene que la palabra "*Turing*" es mejor palabra clave que "*Persona*". Pero, palabras más raras, más comunes en el documento, son aún mejores. Entonces se define la **frecuencia de un término - TF** y se puede obtener el **TF-IDF**.

**IDF** de una palabra `w` = ![IDF](http://f.cl.ly/items/0g0T3L183F2N1f1n1y3b/CodeCogsEqn.png)

Frecuencia de `w` en un documento `d` = ![TF](http://f.cl.ly/items/3Q0f00322H0u0Y0X0Y0r/CodeCogsEqn1.png)

El **TF-IDF** = ![TF-IDF](http://f.cl.ly/items/0l2e0L1C332H3E3E2j0y/CodeCogsEqn2.png)

El **TF-IDF** se inventó como una heurística, pero en 2003 se demostró que la **información mutua** entre **todas** las páginas y **todas** las palabras es proporcional a:

![Mutial Information](http://f.cl.ly/items/0O301e31131U3x400V2i/CodeCogsEqn.png)

* “*An information-theoric perspective of TF-IDF measures*”, Kiko Aizawa, Journal of Information Processing and Management, Volume 39 (1), 2003

### Machine Learning

Machine learning es "aprender" basado en (mucha) información pasada. Es hacer predicciones usando probabilidad condicionada (por ejemplo, el [teorema de Bayes](http://es.wikipedia.org/wiki/Teorema_de_Bayes)).

Es muy importante resaltar que si dos variables `R` y `B` son **independientes**, se tiene que `P(R | B) = P(R)`. Es decir, el hecho de *saber* que `B` ocurrió, no influye en la probabilidad de que ocurra `R`.

#### Clasificador Bayesiano Ingenuo

Emepecemos por definir un clasificador, esto es simplemente un mecanismo de clasificar algún contenido. Por ejemplo, se puede asignar un "sentimiento" positivo o negativo a un comentario (Me encanta la leche - positivo  vs  No me gusta la leche - negativo).

Un Clasificador Bayesiano es aquel que se basa en el Teorema de Bayes. Es *ingenuo* pues se **asume** que todas las variables predictorias son independientes. Recordemos el Teorema de Bayes:

`P(B,R) = P(B|R) P(R) = P(R|B) P(B)`

Ahora, supongamos que un usuario busca "flores `R`ojas y `B`aratas", queremos saber si ese usuario es un 'navegador' o un 'comprador'. En otras palabras si `C = y`.

* `P(C|R,B) * P(R,B)` -> Cuál es la prob. de que sea un `C`omprador si se da `R` y `B`?
* `P(R,B|C) * P(C)` -> Bayes
* `P(R|B,C) * P(B|C) * P(C)` -> Bayes en el primer término
* `P(R|C) * P(B|C) * P(C)` -> Independencia de términos

Ahora, si se dan valores a `R` y `B` (`r` y `b`), se tiene:

![Bayes Naïve Classifier](http://f.cl.ly/items/0P2x3H361x2o281E1v0Z/bayes.png)

Recordando que *machine learning* es basando en datos antiguos, se tienen los valores de cada término de la expresión.

Estos clasificadores funcionan igual para N variables (debido a la independencia entre ellas), entonces se puede generalizar a:

![Bayes Generalizado](http://f.cl.ly/items/052M2D2A3s3j040t0w1e/bayesgral.png)

Como esto da una probabilidad, se elige un límite. Si la probabilidad es al menos el límite se tiene que el usuario es un comprador, si da por debajo se tiene que el usuario es un navegador.

Si miramos *machine learning* desde el punto de vista de teoría de la información, se están transmitiendo características (de una búsqueda, como flores rojas baratas) para predecir un comportamiento (comprar o solo navegar). La idea es mejorar la información mutua entre la búsqueda y el resultado, pero para esto, se necesita definir *información mutua*. Entonces, la **información mutua** entre un término (`F`) y una comportamiento (`B`) es:

![Mutual Info](http://f.cl.ly/items/2e1E3p1h0Z3U2R0Q0M39/CodeCogsEqn.gif)

Claramente, si los términos son independientes, se tiene que `p(f,b) = p(f)*p(b)` y `I(F,B) = 0`. Se presenta un ejemplo en el cual se tienen comentarios sobre el curso, varios positivos y negativos (ya catalogados, recordemos que es decidir en base a resultados pasados). Se muestra como la palabra "odiar" tiene información mutua con el sentimiento (negativo), mientras que la palabra "curso" no.


## Cargar Datos

En la sección de cargar de datos, se habla de computación paralela. Se empieza definiendo la **aceleración (S)** que nos dice realmente cuánto se mejoró al paralelizar el procesamiento. Si procesar los datos con 1 solo proceso toma `Ti` tiempo y con `p` procesos toma `Tp`, entonces `S = Ti / Tp`.

Se define también la **eficiencia (E)** de la paralelización como: `E = Ti/(p*Tp)`.

### Map Reduce

Map Reduce es una *forma* de paralelizar el procesamiento que consiste en dos *entidades*. Un *mapper* y un *reducer*. Los *mappers* reciben parejas `K1, V1` de un diccionario (llave -> valor) y lo transforman en parejas `K2, V2`. Los *reducers* reciben **todas** las parajas (`K2, V2` )de un `K2` específico y procesan estos datos. Normalmente se combinan los datos (contar ocurrencias o sumar números). La plataforma donde se corre el sistema está encargada de enviar todas las parejas de `K2` al mismo *reducer*.

Es importante resaltar que *map-reduce* es una forma de procesamiento **en lote**. Esto quiere decir que no hay interacción (ni feedback) hasta que se tiene el resultado final.

Tomemos el ejemplo de contar las palabras en docuementos. Queremos saber cuántas veces existe una palabra en un conjunto de documentos. Tenemos entonces que cada *mapper* recibe una pareja de `(idDocumento, 'texto')
` y producen un arreglo de parejas `(palabra, cuentaPalabraEnDocumento)`.

![Mappers](http://f.cl.ly/items/1Z3F2s1z0S111l2D1S09/mapp.gif)

Ahora, cada *reducer* redice un arreglo de parejas, donde el primer elemento de cada pareja es el mismo (la palabra). Claramente esto viene de muchos documentos diferentes, pero es **necesario** que la *llave* de la pareja sea la misma. Tenemos entonces el siguiente reducer:

![Reducers](http://f.cl.ly/items/0g2W131k34230K1G3r2e/red.gif)

El proceso se puede mejorar un poco más si los *mappers* procesan ciertos datos y pasan resultados parciales a los reducers. Pero, para que esto sea posible, la función final debe ser asociativa y conmutativa. Esto se debe a que se hace todo en diferente orden y por diferentes *mappers*, entonces el orden no puede importar.

La solución al exámen de Map-Reduce se puede encontrar en: [https://gist.github.com/8d43d211816208efe7d2](https://gist.github.com/8d43d211816208efe7d2). El archivo `word_count.py` contiene la solución al problema.

#### ¿Cómo funciona Map Reduce?

Se tiene un nodo principal (master) que vigila el proceso. Se tienen los archivos de entrada divididos en la red (para evitar un cuello de botella en la lectura). El proceso master inicia los *mappers*, que leen los archivos y los procesan. Es importante resaltar que, por la forma como se plantean los problemas, si un mapper falla se puede iniciar otro y volver a empezar sin que el sistema tenga problemas.

El problema está si falla el nodo master. Obviamente este nodo está monitoreado y en una máquina de alto rendimiento y, dados `n` (grande) procesos, la probabilidad de que al menos uno falle es muy alta, pero de que falle uno específico es muy baja. Basados en esto el master "siempre estará vivo".

Los *mappers* escriben los resultados (parciales) a disco y el master inicia varios nodos *reducers*. Un *mapper* que haya finalizado se puede utilizar como un *reducer* posteriormente. El master es el encargado de enrutar los resultados parciales para que cada *reducer* reciba **todas** las parejas de la misma llave y así poder juntar los resultados.

![Map Reduce](http://f.cl.ly/items/1Z0c1h1w450o1l2o0V2C/mapreduce.png)

Map Reduce es muy una aproximación muy poderosa en la cual **siempre** se escribe (o se obtiene) nueva información. Es importante entender que el resultado de un proceso de Map-Reduce puede ser la entrada de otro proceso, encadenando así diferentes resultados para generar mucha más información **nueva**.












