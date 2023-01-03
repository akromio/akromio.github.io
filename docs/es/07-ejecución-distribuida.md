---
title: Ejecución distribuida
permalink: /es/ejecucion-distribuida
---

**Akromio** se ha desarrollado tanto para su ejecución local como distribuida.
Mientras que **Gattuso** se puede usar para ejecutar trabajos localmente, **Carboni** y **Cavani** nos permitirán hacerlo de manera distribuida.
**Carboni** es el encargado de generar la carga y hacérsela llegar a los agentes **Cavani**, los cuales se pueden instalar en distintos equipos.
Cuando se utiliza un entorno distribuido, necesitaremos un elemento intermediario, en el que **Carboni** deje las peticiones generadas, y del que los agentes **Cavani** recojan su trabajo a realizar.
Como intermediario debemos usar una instancia de **Redis**, se utiliza **Redis Streams**.

El principal uso de estos entornos es la realización de pruebas de rendimiento o carga, así como ataques distribuidos de denegación de servicio (**DDoS**).

## Generación del entorno

Si estamos realizando una prueba de rendimiento o carga o un ataque de denegación de servicio, por lo general, utilizaremos **Docker**.
Por un lado, tendremos:

- Un contenedor para **Carboni**, imagen **akromio/carboni**.

- Un contenedor para **Redis**, imagen **redis**.

- Tantos contenedores como sean necesarios para representar los *bots* de la *botnet*.
  Cada uno de ellos con un **Cavani**, imagen **akromio/cavani**.

Este entorno se puede generar fácilmente con **Gattuso** desde nuestro propio equipo.

## Carboni

**Carboni** es la aplicación de la *suite* **Akromio** que tiene asignada una función de generación de carga.
Actualmente, sólo sabe generar carga para ataques a servicios, ya sea para pruebas de rendimiento o denegación de servicio.
En un futuro, esperemos que no muy lejano, también podrá realizar otras tareas como, por ejemplo, solicitar la realización de copias de seguridad u otros trabajos en determinados agentes.

**Carboni** se encuentra disponible en:

- **NPM**, paquete **@akromio/carboni**.

- **GitHub Actions**, acción **akromio/setup-carboni**.

- **Docker**, imagen **akromio/carboni**.

### Catálogos de carga o fases

Recuerde que **Carboni** tiene una función muy definida: generar solicitudes de ejecución de trabajo a los agentes **Cavani**.
Al igual que **Gattuso** y **Cavani**, los cuales tienen catálogos específicos, de tipo trabajo (*job*), **Carboni** también tiene catálogos específicos, de tipo fase (*stage*).

Un **catálogo de fases** (*stage catalog*) no es más que una colección de fases de generación de solicitudes de ejecución.
Al igual que los de trabajo, se pueden almacenar en registros, facilitando así su reutilización.
Por convenio y buenas prácticas, se ubican en el directorio **stages/catalogs**, en caso de un entorno local, esto quiere decir en **.akromio/stages/catalogs**.

Su formato es muy sencillo:

```yaml
spec: v1.0  # versión de la especificación en la que se escribe
desc: breve descripción del catálogo

dataset:
  # al igual que los catálogos de trabajos

stages:
  # las fases que puede ejecutar carboni
```

### Fases

Una **fase** (*stage*) representa un período de tiempo durante el cual se generan peticiones de ejecución de trabajo.
Actualmente, se soportan los siguientes tipos de fases: las de pausa o descanso, las constantes y las incrementales.

He aquí un ejemplo ilustrativo:

```yaml
spec: v1.0

dataset:
  - input: pauseDuration
    desc: Pause stage duration.
    defaultValue: 10s
    
  - input: warmupDuration
    desc: Warm-up stage duration.
    defaultValue: 1m
  
  - input: loadDuration
    desc: Load stage duration.
    defaultValue: 15m

defaultStageName: load
stages:
  - sleep: pause
    desc: A pause.
    duration: $(pauseDuration)
    
  - const: warmup
    desc: Warm-up stage for filling the cache.
    duration: $(warmupDuration)
    interval:
      duration: 1s
      requests: 5
    jobs:
      - registry: git:///performance
        catalog: webservice
        job: web
        weight: 100

  - inc: load
    desc: Constant load stage for performance analysis.
    duration: $(loadDuration)
    interval:
      duration: 1s
      requests: 200
      inc: 2.5
    jobs:
      - registry: git:///performance
        catalog: webservice
        job: web
        weight: 50
      
      - registry: git:///performance
        catalog: webservice
        job: android
        weight: 30
      
      - registry: git:///performance
        catalog: webservice
        job: ios
        weight: 20
```

#### Fase de pausa o descanso

Una **fase de pausa** (*sleep stage*) es aquella durante la cual no se genera nada.
Su sintaxis es como sigue:

```yaml
- sleep: nombreFase
  desc: breve descripción si lo deseamos
  duration: duración # por ejemplo: 10s, 1m, 2h, etc.
```

#### Fase de generación constante

Una **fase de generación constante** (*constant stage*) es aquella que genera peticiones a un ritmo constante durante un intervalo de tiempo.
En este tipo de fase, tenemos que tener claras algunas cosas:

- La fase debe tener, al igual que la de pausa, una **duración** (*duration*).

- La fase debe tener un **intervalo** (*interval*) que indica cada cuánto tiempo se generará la carga.
  De manera predeterminada, es cada segundo.

- La fase debe indicar cuántas **peticiones** (*requests*) de ejecución debe generar en cada intervalo.

- La fase debe indicar qué trabajos deben ejecutarse.
  En cada intervalo, teniendo en cuenta el número de peticiones que se deben generar, se utilizarán estos trabajos para finalmente generar las peticiones.

Veamos el formato de este tipo de fases antes de continuar:

```yaml
- const: nombreFase
  desc: breve descripción de la fase
  duration: duración    # duración de la fase; ejemplos: 10m, 1h
  interval:
    duration: duración  # duración del intervalo; ejemplos: 1s, 2s, 1m
    requests: número    # número de solicitudes de ejecución a generar por intervalo
  jobs:
    - registry: nombre  # registro donde se encuentra el catálogo de trabajos
      catalog: nombre   # catálogo a utilizar
      job: nombre       # trabajo a ejecutar
      args: dato        # argumentos a pasar al trabajo a ejecutar
      weight: peso      # porcentaje de peticiones que deben ser de este trabajo
    
    - ...
```

La suma de todos los pesos (*weights*) de los trabajos debe ser **100**.
El **peso** (*weight*) indica el porcentaje de las peticiones generadas que deben asociarse al trabajo.

#### Fase de generación incremental

Una **fase de generación incremental** (*incremental stage*) es aquella que genera peticiones a un ritmo constante durante un intervalo de tiempo, incrementando en cada intervalo el número de peticiones en un determinado número dado.
Su sintaxis es similar a una fase constante, teniendo en cuenta lo siguiente:

- Se usa ***inc*** en vez de ***const***.

- El intervalo debe contener una propiedad ***inc*** que indique el número de peticiones a incrementar en cada intervalo.

En el siguiente ejemplo, se comienza con **200** peticiones, incrementándose **2.5** peticiones en cada intervalo:

```yaml
inc: load
desc: Constant load stage for performance analysis.
duration: $(loadDuration)
interval:
  duration: 1s
  requests: 200
  inc: 2.5
jobs:
  # ...
```

#### *Botnet*

Una ***botnet*** es una colección de agentes (*bots*) que se encargarán de ejecutar las peticiones generadas por **Carboni**.
**Carboni** genera las peticiones y se las asigna a los agentes de esta *botnet* según una configuración dada.
Esta configuración debe indicarse en un argumento.
En el caso de desplegarse en un contenedor, usaremos variables de entorno **KRM_ARG_**.

Cada fase puede utilizar una *botnet* que puede ser la misma o distinta a las de las demás.
En el momento de ejecutar **Carboni**, indicaremos para cada fase cuál es su *botnet*.
En caso de no indicarse, se usará el argumento *botnet*.

Para cada botnet, hay que indicar:

- El tipo de botnet: **redis**.

- Los miembros de la *botnet*, o sea, los identificadores de los agentes asociados a la *botnet*.

Vamos a presentar un ejemplo y, a continuación, lo analizamos.
Supongamos el siguiente catálogo:

```yaml
spec: v1.0

stages:
  - sleep: pause
    desc: A pause.
    duration: 10s
    
  - const: warmup
    desc: Warm-up stage for filling the cache.
    duration: 10s
    interval:
      duration: 1s
      requests: 5
    jobs:
      - registry: git:///performance
        catalog: webservice
        job: web
        weight: 100

  - inc: load
    desc: Constante load stage for performance analysis.
    duration: 10s
    interval:
      duration: 1s
      requests: 1
      inc: 0.5
    jobs:
      - registry: git:///performance
        catalog: webservice
        job: web
        weight: 50
      
      - registry: git:///performance
        catalog: webservice
        job: android
        weight: 30
      
      - registry: git:///performance
        catalog: webservice
        job: ios
        weight: 20
```

La fase de pausa dura 10 segundos.
La fase de calentamiento, 10 segundos también, generándose cada segundo 5 peticiones.
Finalmente, la fase de carga dura otros 10 segundos, comenzando en 1 petición e incrementando en 0.5 cada segundo.

Vamos cómo ejecutarlo desde nuestra máquina local:

```bash
# fijamos el argumento botnet como variable de entorno
$ cat args.yaml
botnet:
  impl: redis
  host: localhost
  port: 6379
  bots:
    - bot: cavani1
    - bot: cavani2
    - bot: cavani3
$ carboni encode args.yaml -x
export KRM_ARG_botnet=json+base64://eyJpbXBsIjoicmVkaXMiLCJob3N0IjoibG9jYWxob3N0IiwicG9ydCI6NjM3OSwiYm90cyI6W3siYm90IjoiY2F2YW5pMSJ9LHsiYm90IjoiY2F2YW5pMiJ9LHsiYm90IjoiY2F2YW5pMyJ9XX0=
$ export KRM_ARG_botnet=json+base64://eyJpbXBsIjoicmVkaXMiLCJob3N0IjoibG9jYWxob3N0IiwicG9ydCI6NjM3OSwiYm90cyI6W3siYm90IjoiY2F2YW5pMSJ9LHsiYm90IjoiY2F2YW5pMiJ9LHsiYm90IjoiY2F2YW5pMyJ9XX0=

# ejecutamos las fases que deseamos del catálogo
$ carboni r -L pause warmup load
Stage: pause (duration: 10s)
Stage: warmup (duration: 10s)
[2023-01-02T10:26:04.602Z] cavani1 ts:1672655164601 assignTs:1672655164602 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:04.603Z] cavani2 ts:1672655164601 assignTs:1672655164602 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:04.603Z] cavani3 ts:1672655164601 assignTs:1672655164602 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:04.603Z] cavani1 ts:1672655164601 assignTs:1672655164602 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:04.603Z] cavani2 ts:1672655164601 assignTs:1672655164602 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:05.605Z] cavani3 ts:1672655165601 assignTs:1672655165603 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:05.606Z] cavani1 ts:1672655165602 assignTs:1672655165604 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:05.607Z] cavani2 ts:1672655165602 assignTs:1672655165604 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:05.607Z] cavani3 ts:1672655165602 assignTs:1672655165604 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:05.608Z] cavani1 ts:1672655165602 assignTs:1672655165605 registry:git:///performance catalog:webservice job:web
...
[2023-01-02T10:26:12.611Z] cavani2 ts:1672655172609 assignTs:1672655172610 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:12.611Z] cavani3 ts:1672655172609 assignTs:1672655172610 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:12.611Z] cavani1 ts:1672655172609 assignTs:1672655172610 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:12.611Z] cavani2 ts:1672655172610 assignTs:1672655172610 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:12.612Z] cavani3 ts:1672655172610 assignTs:1672655172610 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:13.614Z] cavani1 ts:1672655173611 assignTs:1672655173612 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:13.615Z] cavani2 ts:1672655173611 assignTs:1672655173612 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:13.615Z] cavani3 ts:1672655173611 assignTs:1672655173612 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:13.615Z] cavani1 ts:1672655173611 assignTs:1672655173612 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:13.615Z] cavani2 ts:1672655173611 assignTs:1672655173613 registry:git:///performance catalog:webservice job:web
Stage: load (duration: 10s)
[2023-01-02T10:26:14.621Z] cavani1 ts:1672655174620 assignTs:1672655174620 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:15.621Z] cavani2 ts:1672655175621 assignTs:1672655175621 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:16.622Z] cavani3 ts:1672655176622 assignTs:1672655176622 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:16.622Z] cavani1 ts:1672655176622 assignTs:1672655176622 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:17.623Z] cavani2 ts:1672655177623 assignTs:1672655177623 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:17.623Z] cavani3 ts:1672655177623 assignTs:1672655177623 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:18.624Z] cavani1 ts:1672655178624 assignTs:1672655178624 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:18.624Z] cavani2 ts:1672655178624 assignTs:1672655178624 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:18.624Z] cavani3 ts:1672655178624 assignTs:1672655178624 registry:git:///performance catalog:webservice job:ios
[2023-01-02T10:26:19.625Z] cavani1 ts:1672655179624 assignTs:1672655179624 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:19.625Z] cavani2 ts:1672655179624 assignTs:1672655179625 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:19.625Z] cavani3 ts:1672655179624 assignTs:1672655179625 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:20.627Z] cavani1 ts:1672655180625 assignTs:1672655180626 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:20.627Z] cavani2 ts:1672655180626 assignTs:1672655180626 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:20.628Z] cavani3 ts:1672655180626 assignTs:1672655180626 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:20.628Z] cavani1 ts:1672655180626 assignTs:1672655180627 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:21.629Z] cavani2 ts:1672655181627 assignTs:1672655181628 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:21.630Z] cavani3 ts:1672655181627 assignTs:1672655181629 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:21.630Z] cavani1 ts:1672655181627 assignTs:1672655181629 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:21.630Z] cavani2 ts:1672655181627 assignTs:1672655181629 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:22.630Z] cavani3 ts:1672655182629 assignTs:1672655182630 registry:git:///performance catalog:webservice job:ios
[2023-01-02T10:26:22.630Z] cavani1 ts:1672655182629 assignTs:1672655182630 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:22.630Z] cavani2 ts:1672655182629 assignTs:1672655182630 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:22.630Z] cavani3 ts:1672655182629 assignTs:1672655182630 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:22.631Z] cavani1 ts:1672655182629 assignTs:1672655182630 registry:git:///performance catalog:webservice job:android
[2023-01-02T10:26:23.632Z] cavani2 ts:1672655183631 assignTs:1672655183631 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:23.632Z] cavani3 ts:1672655183631 assignTs:1672655183632 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:23.632Z] cavani1 ts:1672655183631 assignTs:1672655183632 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:23.632Z] cavani2 ts:1672655183631 assignTs:1672655183632 registry:git:///performance catalog:webservice job:web
[2023-01-02T10:26:23.632Z] cavani3 ts:1672655183631 assignTs:1672655183632 registry:git:///performance catalog:webservice job:ios
```

Observe que durante la fase de pausa, no se generan solicitudes de ejecución.
Durante la fase de calentamiento, se generan sólo solicitudes que, en este caso, simulan un usuario web.
Mientras que en la fase de carga, se generan solicitudes tanto de web, android como ios.
Cada 100 peticiones realizadas, se cumplirán los pesos o porcentajes indicados.
Y tras cada lote de 100 peticiones, el orden puede cambiar.

La repartición es en forma de anillo entre los agentes indicados en la *botnet*.
Cuando se ha recorrido todos los *bots* de la *botnet*, se vuelve a comenzar.

La configuración de la *botnet* depende del distribuidor de las peticiones.
Groso modo, **Carboni** tiene los siguientes componentes:

- El **iniciador** (*starter*) el cual pone encima del proceso las peticiones a *rellenar*.
  Se ejecuta cada intervalo y pone tantas peticiones vacías como se indica en *requests*.

- El **asignador** (*assigner*) coge las peticiones puestas en el flujo por el iniciador y con la información de la botnet y de los trabajos, les va asignando el trabajo a realizar.
  El asignador es el encargado de hacerlo y lo hace cumpliendo con los pesos indicados en cada trabajo.
  Pero lo hará aleatorimente.
  Un lote de 100 peticiones no tiene que ser exactamente el mismo que el siguiente.
  Sí se cumplirán los pesos, pero no tiene por qué cumplirse el mismo orden.

- Finalmente, el **distribuidor** (*distributor*) es el que recibe las peticiones de ejecución y se las hace llegar a los agentes.

En el ejemplo anterior, se ha utilizado el distribuidor por flujos de **Redis**.
Con la opción ***-L***, pedimos a **Carboni** que sólo muestre las peticiones que va generando, sin hacérselas llegar a nadie.
Tiene como objeto ayudar a que podamos ver cómo trabajará y lo que generará.
Pero de una ejecución a otra, el orden de las peticiones puede cambiar.
Actualmente, se hace aleatoriamente.

Si se desea, para cada fase podemos indicar el identificador de su *botnet*, el cual debe ser uno de los argumentos.
Si no se indica uno explícitamente, se usará el argumento *botnet*.
El identificador de la botnet a usar se separa del identificadror de la fase por dos puntos (`:`).
Ejemplo:

```bash
$ export KRM_ARG_botnet1=...
$ export KRM_ARG_botnet2=...
$ carboni r pause warmup:botnet1 load:botnet2
```

#### Distribuidor de *Redis*

El **distribuidor de *Redis*** (*Redis distributor*) tiene como objeto dejar las peticiones en flujos (*streams*) de **Redis**.
Por lo tanto, tanto **Carboni** como los agentes **Cavani** deben saber cómo llegar a la instancia de **Redis**.
**Carboni** para escribir en ella y los **Cavani**s para leer de ella.
El identificador de cada *bot* es el nombre del flujo en el que debe escribir **Carboni**.
A cada **Cavani**, le indicaremos cuál es el flujo del que debe extraer sus peticiones de ejecución.
En el ejemplo anterior, los flujos serían *cavani1*, *cavani2* y *cavani3*.

La configuración de **Redis** se indica en la *botnet* como sigue:

```json
{
  "impl": "redis",
  "host": "localhost",
  "port": 6379,
  "bots": [
    {"bot": "cavani1"},
    {"bot": "cavani2"},
    {"bot": "cavani3"}
  ]
}
```

Puede definir el argumento de la *botnet* en un archivo **YAML** y usar **carboni encode** para obtener su representación en **json+encode**.
Tal y como hemos hecho en los ejemplos anteriores.

Una nota antes de continuar.
Aunque hemos usado variables de entorno **KRM_ARG_** para pasar los argumentos relacionados con las *botnets*, podríamos hacerlo también con la opción ***-a***.
Lo hemos hecho así porque cuando desplegamos el entorno en **Docker**, al crear los contenedores lo mejor será crear variables de entorno con los argumentos.
Pero lo siguiente también sería posible:

```bash
$ carboni r pause warmup load -L -a args.yaml
```

### Identificador de *Carboni*

La opción ***--name*** (o ***-n***) permite fijar un nombre personalizado para la instancia de **Carboni** en ejecución.
Este nombre se utiliza para fijar el de la conexión de **Redis** y, así, poder localizar más fácilmente las conexiones abiertas por **Carboni** con el comando **CLIENT LIST**.

Por ejemplo, si lanzamos la siguiente ejecución:

```bash
$ carboni r pause warmup load -a args.yaml -n test-carboni
```

Podremos ver algo similar a lo siguiente en nuestro **Redis**:

```bash
$ redis-cli CLIENT LIST
id=52 addr=172.17.0.1:59754 laddr=172.17.0.2:6379 fd=8 name=test-carboni age=2 idle=0 flags=N db=0 sub=0 psub=0 ssub=0 multi=-1 qbuf=0 qbuf-free=20474 argv-mem=0 multi-mem=0 rbs=1024 rbp=22 obl=0 oll=0 omem=0 tot-mem=22272 events=r cmd=xadd user=default redir=-1 resp=2
id=54 addr=172.17.0.1:39374 laddr=172.17.0.2:6379 fd=9 name= age=0 idle=0 flags=N db=0 sub=0 psub=0 ssub=0 multi=-1 qbuf=26 qbuf-free=20448 argv-mem=10 multi-mem=0 rbs=16384 rbp=16384 obl=0 oll=0 omem=0 tot-mem=37658 events=r cmd=client|list user=default redir=-1 resp=2
```

## Cavani

Bajo desarrollo.
