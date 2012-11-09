# Remote Laboratory Systems Interoperability Standard: Interface Definitions. GOLC

Fuente: [Document URL - GOLC Website](http://www.online-lab.org/index.php?option=com_joobb&view=topic&topic=12&Itemid=47)

El documento de interoperabilidad entre sistemas de laboratorios en línea (reales o virtuales) **no** establece una *forma* de implementación de servicios. La idea es encontrar servicios (interfaces o mensajes) comunes y estandarizarlos, dejando la implementación a criterio de cada plataforma de laboratorios.

## Terminología

Antes de entrar en detalles sobre el documento, es necesario tener claros algunos términos:

**Interfaz:** Un *canal* de comunicación común entre dos sistemas de laboratorios (un *consumidor* y un *proveedor*) por el cual se pueden comunicar.

**Consumidor:** Una organización que ha negociado (tiene acceso) con un *proveedor* a nombre de sus propios usuarios (los del consumidor).

**Grupo Consumidor:** Un grupo de usuarios que hace uso de un acuerdo entre un *consumidor* y un *proveedor*. Es importante resaltar que este grupo tiene unos permisos específicos en un contexto específico.

**Usuario:** Una persona *identificable* en un laboratorio en línea. Es importante resaltar que los *proveedores* autentican al nivel de consumidor (no de usuarios individuales). Es responsabilidad de los consumidores autenticar al usuario y certificar los grupos a los que pertenece. El proveedor solo necesitará la información del usuario individual para estadísticas y análisis de uso del laboratorio.

**Proveedor:** Una organización que comparte (o hace disponible) sus laboratorios en línea (algunos o todos) para los *consumidores*.

**Rig y RigSet (Rig, Tipo de Rig y Pools):** Un *rig* es una instancia de un *aparato* (equipo de laboratorio) sobre la cual se puede ejecutar un experimento.

Un *RigSet* puede ser un conjunto de *Rigs* y contener más RigSets, creándo un grafo dirigido donde las hojas son, probablemente, *RigSets* con un solo *Rig*, esto se necesita pues no todos los laboratorios se manejan igual. Un estudiante puede elegir un *rig* específico solo por la calibración (y consistencia en la calibración), mientras que en otros experimentos se podría utilizar cualquier *rig*.

## Principios del estándar  

En la sección de principios se definen 6 principios sobre los cuales se basará el estándar.

### Principio 1

El **primer principio** define que el estándar definirá un *baseline*, que es la interfaz mínima que debe ofrecer un sistema de laboratorios para implementar el estándar. Se define también que se pueden crear interfaces más amplias para funcionalidad específica de un laboratorio, pero, como no todos tendrán la misma interfaz, debe existir un servicio que permite preguntar por la interfaz del laboratorio.

Creo que este principio es muy válido, pero solo mientras se mantenga la parte de "preguntar por la interfaz". No tipo WSDL, pero los consumidores que se conecten a los laboratorios, deberían ser capaces de obtener la información sobre las funciones que ofrece el laboratorio.

Es más, los consumidores deberían poder **interactuar** con los laboratorios, solo conociendo lo básico de la respuesta del servidor. Esto, conocido como [HATEOAS](http://en.wikipedia.org/wiki/HATEOAS) (*Hypermedia as the Engine of Application State*). Esto implica que los clientes interactúan con la aplicación (los laboratorios en este caso) **completamente** por medio de ***Hypermedia* brindada por las aplicaciones**.

Para mí, HATEOAS, es una idea impresionante. Es **muy** escalable (así funciona WWW) pueden interactuar con los laboratorios conociendo solo **una** URL dentro del laboratorio. A partir de ahí, todas las interacciones se hacen por medio de _links_ encontrados en la respuesta del servidor.  Para verlo de una manera más clara, si yo entro a una página de internet, como la de EAFIT, solo debo saberme la [dirección principal](http://www.eafit.edu.co). Después, toda la interacción que yo tengo con la página (como mirar un Pénsum de una carrera) lo hago basado en las respuestas de la primera petición. Así, no importa si las URLs internas de EAFIT cambian, mi consumidor, siempre podrá ir al Pénsum deseado (siempre y cuando las respuestas tengan el camino).

### Principios 2 & 3

El **segundo y tercer principio** indican que el estándar deberá brindar interoperabilidad entre sistemas que hayan adoptado versiones diferentes del estándar (segundo). Y que el estándar debe ser compatible con versiones futuras (*upwardly compatible*) cuándo sea posible.

Yo no estoy completamente de acuerdo con esto (especialmente con el segundo). Creo que es muy importante brindar **cierta** compatibilidad entre versiones, pero no total. A largo plazo mantener versiones muy antiguas es muy costoso.

Un buen mecanismo para esto, es mantener un sistema de versionamiento que permita reconocer cuáles versiones son compatibles. Un ejemplo es **[Semantic Version](http://semver.org/)**. La forma más fácil de explicar SemVer es que cada versión está compuesta por 3 números: **Major.Minor.Patch**.

* El primer número (*major*) indica la versión 'principal' del estándar. **Cualquier aumento en este número, implica que se han introducido cambios que no son compatibles con la versión anterior**.

* El segundo número (*minor*) indica la 'sub-versión' del estándar. El incremento en este número indica que **se han agregado funciones pero todas son compatibles con la versión actual del estándar**.

* El tercer número (*patch*) indica que no se ha agregado nada nuevo, pero se **han corregido bugs** encontrados en la versión actual.

Utilizando SemVer, un consumidor 3.2.1 puede comunicarse con un proveedor 3.5.3 sin ningún problema. El "*minor version*" permite evolucionar el estándar sin afectar los clientes. Si se agregan cambios incompatibles, se cambia el "*major version*" y preferiblemente, se da soporte para las **últimas dos versiones major** y no para todas las que ha habido.

### Principio 4

El **cuarto** principio determina que el estándar debe brindar diferentes opciones a los proveedores de acuerdo al *nivel de servicio* que desean brindar (tanto entre varios sistemas como al interior del propio proveedor). Ponen algunos ejemplos como:

* Varios niveles de garantía de disponibilidad del laboratorio.
* La disponibilidad de algunos datos recogidos al usar el laboratorio, como que hardware y software se utilizó en la sesión.  Es importante resaltar que estos datos son **complementarios** y no son los resultados de los experimentos.

Personalmente me parece fantástico este punto. Cada proveedor del servicio debe tener autonomía para decidir qué presta y con qué calidad lo presta. Forzar un nivel de servicio muy alto sería una barrera muy grande para que nuevos miembros ofrecieran sus laboratorios.

### Principio 5

El **quinto** principio asegura que el estándar proveerá **autenticación y autorización** para consumidores, grupos y usuarios, basados en estándares actuales. El estándar que se utilizará para esto debe soportar autenticación tanto a nivel de consumidor (entre universidades) como por grupos de consumidor (a nivel de usuarios). Esto es indispensable para poder asegurar el principio 4 (del nivel de prestación del servicio).

Creo que actualmente el estándar que se debe utilizar es ***OAuth 2.0***. Hay mucho trabajo y dinero en mejorarlo y tiene el apoyo de las empresas más importantes de internet.

## Perfiles e Interacciones

En esta sección el estándar define las interacciones básicas que un Proveedor debe soportar. Se define sólo el perfil básico pero se mencionan "futuros perfiles".

Se presentan diferentes escenarios y los mensajes requeridos para las interacciones.

### Perfil Básico

Este perfil define la funcionalidad mínima requerida para que un consumidor pueda utilizar un laboratorio (interactivo o *batch*) de un proveedor.

#### Conocimiento del Sistema (*System Enqiry*)

Se definen diferentes mensajes para diferentes interacciones, por ejemplo, se da una sección de **conocimiento del sistema** (*Systen Enquiry*), donde un consumidor puede pregunrarle al proveedor por los perfiles que soporta y qué versión de un perfil específico soporta. También se puede preguntar por el nivel de servicio que se puede esperar de ese proveedor.

En esta sección se dan varios ejemplos de uso de estos mensajes. Por ejemplo, se pregunta por un nivel de servicio específico. Dentro del ejemplo se definen atributos por los que uno puede preguntar (como *ServiceAvailability* o *InformationRetention*). Personalmente pienso que hay un problema con esto (no con el ejemplo, sino con definir las cosas dentro del ejemplo). Si se van a definir los atributos por los que puedo preguntar, deben ser una sección espefícica del estándar.

Pero, adicionalmente a esto, me parece que esta definición **no** debe ser así. Debería existir un *endpoint* donde se pregunte por los atributos por los que puedo preguntar (volviendo a la idea de *HATEOAS*). Con el nuevo endpoint, yo podría tener una respuesta así:

    attributes: [
        {
          name: "Service Availability",
          endPointQuery: "QueryServiceLevel(Attrib='ServiceAvailability', [RigSet=])"
        },
        {
          name: "Information Retention",
          endPointQuery: "QueryServiceLevel(Attrib='InformationRetention', [RigSet=])"
        }
    ]

Esto sería aún mejor si cada atributo tuviera su propio *endpoint*, entonces la respuesta podría ser algo como:

    attributes: [
        {
          name: "Service Availability",
          url: "https://example.com/service-availability"
        },
        {
          name: "Information Retention",
          url: "https://example.com/information-retention"
        }
    ]

Donde los parámetros se pueden mandar en el request de HTTP.  Nuevamente, las ventajas que esto genera es que si el proveedor cambia de tal manera que, por ejemplo, un servidor atiende las peticiones sobre el servicio de seguridad y otro servidor sobre la consistencia de los datos, los clientes **no** tienen que cambiar, pues siempre han leído el "lugar a preguntar" de la respuesta del servidor.

Esta observación se mantiene en todos los ejemplos presentados en el estándar, por esto, solo la pondré una vez.


















