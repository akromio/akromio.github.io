---
title: Plugins
permalink: /es/plugins
layout: default
---

Un ***plugin*** es un componente que aporta operaciones extras para su utilización en los catálogos de trabajo.
Es otro medio con el que extender la funcionalidad que viene de fábrica con **Akromio**.
Estos *plugins* se desarrollan con **JavaScript** o con cualquier lenguaje de programación que sea capaz de *transpilar* a este lenguaje y a **Node.js**.

Actualmente, los *plugins* pueden ser con estado y sin estado.
Un ***plugin* con estado** (*stateful state*) es aquel que contiene un valor que puede ser reutilizado por sus operaciones y las ejecuciones de estas operaciones.
En cambio, un ***plugin* sin estado** (*stateless plugin*) es aquel que no comparte un valor entre las operaciones ni sus ejecuciones.

## Propiedad *plugins* de los catálogos de trabajo

Todo catálogo de trabajo dispone de una propiedad ***plugins*** con la que cargar *plugins* extras que necesitamos.
En el siguiente ejemplo, solicitamos la carga del plugin ***@akromio/pi-redis*** para ejecutar operaciones en una instancia de **Redis**:

```yaml
plugins:
  - plugin: redis
    ini:
      host: localhost
      port: 6379
```

La propiedad ***plugins*** es una lista donde cada uno de sus elementos representa un *plugin*, el cual puede contener las siguientes propiedades:

Propiedad | Descripción
-- | --
***plugin*** | Identificador con el que haremos referencia al *plugin* en el catálogo para usar sus operaciones.
***impl*** | Identificador del paquete de **Node.js** que implementa la lógica del *plugin*.
***ini*** | Argumentos a pasar al iniciador del *plugin* si es con estado.

La propiedad ***impl*** hace referencia al paquete de **Node.js** que implementa el *plugin*.
Estos paquetes *siempre* deben ser paquetes de ámbito, recordemos, *@ámbito/paquete*.
En caso de omisión, se usará el valor de ***plugin***.
El nombre del paquete debe comenzar por *pi-*, o sea, debe seguir el siguiente formato: **@ámbito/pi-nombre**.
Para relajar un poco las cosas, si el *plugin* es oficial del proyecto como, por ejemplo, el de **Redis**, lo que hace **Akromio** es lo siguiente:

1. Si el nombre de paquete no contiene ámbito y no comienza por *pi-*, le añade el prefijo *@akromio/pi-*.

2. Si el nombre de paquete no contiene ámbito y comienza por *pi-*, lo prefija sólo con *@akromio/*.

De ahí que la anterior declaración, simplemente *redis*, sea realmente *@akromio/pi-redis*.

Por otra parte, tenemos la propiedad ***plugin*** con la que indicamos el nombre con el que haremos referencia al *plugin* en el catálogo y con el que accederemos a sus operaciones.
Recuerde que también indica, en caso de omisión de ***impl***, el nombre del paquete que implementa su lógica.
Así pues, podremos tener cosas como la siguiente:

```yaml
- macro: create-broker
  title: Create the broker of the botnet
  ini:
    - sudo: docker pull $(botnet.broker.image)
  steps:
    - sudo: docker run --name $(botnet.broker.container) $(network) --rm -d -p 6379:6379 $(botnet.broker.image)
    - sleep 100ms
    - redis.ping
```

Observe el uso de la operación *ping* del plugin *redis*.

Gracias al nombre, podemos utilizar un *plugin* con estado con varias configuraciones.
Al igual que creamos varios objetos instancias de una misma clase.
Lo siguiente es válido:

```yaml
plugins:
  - plugin: redis1
    impl: redis
    ini:
      host: localhost
      port: 63179

  - plugin: redis2
    impl: redis
    ini:
      host: localhost
      port: 63279
```

Finalmente, la propiedad ***ini*** indica los argumentos a pasar al *plugin* en su instanciación.
Si el *plugin* es sin estado o con estado pero no requiere argumentos, se puede omitir.

## *Preset*

Un ***preset*** es una colección de *plugins* relacionados con alguna cosa.
Actualmente, **Gattuso** utiliza automáticamente, sin necesidad de incorporarlo explícitamente, el *preset* **@akromio/preset-gattuso**;
y **Cavani**, **@akromio/preset-cavani**.

Todos los *presets* se deben implementar mediante paquetes con ámbito cuyo formato de nombre debe ser **@ámbito/preset-nombre**.

## Instalación de *plugins*

Tanto **gattuso** como **cavani** proporcionan un comando de instalación de *plugins*.
La idea es indicarle un catálogo y la aplicación recorrerá su propiedad *plugins* buscando si, en el directorio en el que se encuentra, tiene acceso a los *plugins* indicados.
Aquellos que no pueda acceder, los instalará, o sea, ejecutará el comando **npm i** correspondiente.
Así, por ejemplo, si decidimos utilizar el *plugin* **@akromio/pi-redis**, no tendremos más que indicarlo en el catálogo y, a continuación, ejecutar el siguiente comando:

```bash
$ gattuso i -g fs://$PWD/registry -c udp-flood
Installing @akromio/pi-redis...
```

Puede utilizar también **cavani**.
Use aquel que vaya a utilizar con el catálogo.
Esto es importante porque cada aplicación viene con su propio *preset* y estos no son iguales.
El *preset* de **Gattuso** tiene *plugins* que no tiene el de **Cavani** y viceversa.
