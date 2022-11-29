---
title: Registros
permalink: /es/registros
---

# Registros

Un **registro** (*registry*) es un contenedor de catálogos.
Estos registros utilizan **conectores** (*connectors*) los cuales contienen la lógica que permite acceder a los registros de un determinado almacenamiento.
Son como adaptadores que permiten que los catálogos se encuentren almacenados en distintos almacenes de datos como, por ejemplo, el sistema de ficheros.

Hasta el momento, **Akromio** soporta varios conectores:

- El **conector de sistema de archivos** (*filesystem connector*), que almacena los catálogos en el sistema de archivos de nuestro equipo.

- El **conector de Git** (*Git connector*) que lo hace en un repositorio de **GitHub**.

## Búsqueda de catálogos

**Akromio** puede trabajar con varios registros al mismo tiempo.
En este caso, debemos indicar explícitamente el registro a consultar o bien indicar en qué orden debe consultar los registros con los que puede trabajar.

Por un lado, tenemos la variable de entorno **$KRM_REGISTRIES**, que indica la lista de los registros en el orden en el que deseamos sean consultados cuando no se indique ninguno en particular.
Podemos consultar su valor fácilmente con el comando **gattuso e KRM_REGISTRIES**:

```bash
$ gattuso e KRM_REGISTRIES

 Variable        Value       Desc.
----------------------------------------------------------------------------------------
 KRM_REGISTRIES  local,user  Available registries to use in order, separated by commas.
```

En el ejemplo anterior, se buscará primero en el registro local del proyect y, a continuación, en el del usuario.

También es posible utilizar la opción ***--registries*** o ***-g***.
En caso de omitirse, su valor predeterminado es el de la variable de entorno **$KRM_REGISTRIES**.
El siguiente ejemplo muestra cómo solicitar la ejecución del trabajo predeterminado del catálogo predeterminado en el registro *local* o *user*, según el que lo contenga en ese orden:

```bash
gattuso -g local,user r
```

## Listado de catálogos de un registro

Para conocer los catálogos disponibles en un registro, utilice el comando **gattuso g** como, por ejemplo:

```bash
gattuso g nombreRegistro
```

## Registros de sistema de archivos

Un **registro de sistema de archivos** (*filesystem registry*) permite almacenar catálogos en el sistema de archivos local, o sea, en el equipo.
Para este tipo de registros, hay que indicar la **ruta base** (*base path*) del directorio que se considerará el registro con los catálogos.

De manera predeterminada, **Akromio** crea el **registro local** (*local registry*), el cual representa al directorio actual.
Y por ello, no hay que crearlo.
De la mismo manera, si no sobrescribimos la variable de entorno ni indicamos la opción ***--registries***, este registro es el primero en consultarse.
Por otra parte, tenemos el **registro de usuario** (*user registry*), el cual se encuentra en el directorio *home* del usuario.

Si necesitamos crear nuestro propio registro, debemos usar una sintaxis similar a la siguiente:

```
fs://rutaBase
```

Como, por ejemplo:

```
fs:///mi/registro/de/catálogos
```

Cuando es necesario añadir nuevos registros o deseamos fijar nuestros registros de búsqueda, tendremos que usar **$KRM_REGISTRIES** o ***--registries***.
Ejemplo:

```bash
gattuso -g local,root=fs:///mi/directorio/local r
```

Los registros tienen un nombre con el que identificarlos.
Si no lo fijamos, será el del protocolo usado, en este caso, *fs*, pero sólo podremos tener uno con este nombre.
Lo mejor es indicar otro como en el ejemplo que hemos indicado *root*.
Esto permitirá su uso más fácilmente.

Así, por ejemplo, para indicar que deseamos ejecutar un trabajo de un catálogo de un registro concreto, podremos tener algo como:

```bash
gattuso -g local,root=fs:///mi/directorio/local -c root://catálogo r trabajo
```

En este caso, el catálogo a ejecutar es *catálogo* y se tendrá  que buscar en el registro *root*.
Si no indicamos el registro explícitamente, buscará primero en *local* y después en *root* tal y como hemos indicado con la opción ***-g***.

## Registros de *Git*

También es posible un **registro de *Git*** (*Git registry*) con el que utilizar un repositorio de **GitHub** como almacén de catálogos.
Puede consultar el registro oficial ***builtin-registry***, [https://github.com/akromio/builtin-registry](https://github.com/akromio/builtin-registry), como ejemplo.

Los registros de **Git** no se encuentran en la lista predeterminada de búsqueda de registros, pero puede añadir tantos como necesite.
A continuación, se muestra cómo invocar el trabajo predeterminado del catálogo *empty* del registro de Git *builtin-registry* para consultar las variables del conjunto de datos global:

```bash
# usando variables de entorno para su configuración
gattuso -g git -c empty dataset

# configurando el registro explícitamente
gattuso -g git://akromio/built-in-registry/master/.akromio -c empty dataset
```

Los registros de Git se configuran mediante los siguientes formatos:

```bash
git
git://repositorio
git://usuario/repositorio
git://usuario/repositorio/rama
git://usuario/repositorio/rama/prefijo
```

Cuando no se indican todos los campos, estos se toman de las variables de entorno siguientes:

```bash
$ gattuso e KRM_REGISTRY_GIT_*

 Variable                 Value                      Desc.
--------------------------------------------------------------------------------------------
 KRM_REGISTRY_GIT_HOST    raw.githubusercontent.com  Host where the Git repository is.                                
 KRM_REGISTRY_GIT_USER    akromio                    User name where the Git repository is.                           
 KRM_REGISTRY_GIT_REPO    builtin-registry           Repository name to use as registry.                              
 KRM_REGISTRY_GIT_BRANCH  master                     Branch name to use.
 KRM_REGISTRY_GIT_PREFIX  .akromio                   Path prefix to use.     
```

Así pues, si solo indicamos *git*, será lo mismo que *git://akromio/builtin-registry/master/.akromio*.

Si lo deseamos, podemos dejar cualquier sección en blanco para que su valor se tome de la variable de entorno correspondiente.
Así, por ejemplo, *git:///nodejs-registry*, será lo mismo que *git://akromio/nodejs-registry/master/.akromio*.

### Listado de catálogos de un registro de *Git*

Este tipo de registro no soporta esta operación.