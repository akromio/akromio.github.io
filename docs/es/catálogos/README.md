---
title: Catálogos
permalink: /es/catalogos
---

Un **catálogo** (*catalog*) representa un índice de trabajos, donde un **trabajo** (*job*) una operación para realizar alguna cosa particular como, por ejemplo, un proceso automatizado.
Los catálogos se declaran en formato **YAML**.
He aquí un ejemplo ilustrativo para abrir boca:

```yaml
spec: v1.0
desc: Catalog for working with Visual Studio Code.

dataset:
  - const: src
    desc: Template dir where the source templates are.
    value: $(__dir)/_vscode
    tags: [hidden]
  
  - const: dst
    desc: Local directory where to save the files.
    value: $(workDir)/.vscode
  
  - const: settingsQ
    desc: Questions for the creation of the settings.json file
    tags: [hidden, questions]
    value:
      - input: lang
        title: cSpell.language (for example, es, en, fr, it...)
        defaultValue: en
      
      - confirm: prettier
        title: Would you like to configure Prettier for TypeScript?
        defaultValue: false
      
      - confirm: formatMarkdownOnSave
        title: Would you like to format Markdown files on save?
        defaultValue: false

jobs:
  - macro: settings
    title: Create .vscode/settings
    ini:
      - quiet: $answers = inquire $(settingsQ) $(answers)
    steps:
      - fs.createDir $(dst)
      - quiet: $item = cr.getItem $(src)/settings.json.hbs
      - quiet: $content = hbs.render $(item.value) $(answers)
      - file.write $(content) $(dst)/settings.json
```

El ejemplo anterior se ha extraído del catálogo predefinido **vscode** disponible en el registro predefinido **builtin-registry** disponible en **Git**.
Su ejecución es como sigue:

```bash
$ gattuso -g git -c vscode r settings
> Create .vscode/settings
? cSpell.language (for example, es, en, fr, it...) en
? Would you like to configure Prettier for TypeScript? No
? Would you like to format Markdown files on save? No
  - fs: create dir '/tmp/akro/.vscode' if not exists ok (3 ms)
  - file: write content to '/tmp/akro/.vscode/settings.json' ok (2 ms)
```

La opción ***-g*** indica el registro que contiene el catálogo, el cual se indica mediante la opción ***-c***.
El comando ***r*** (o ***run***) indica que deseamos ejecutar el trabajo indicado, en nuestro caso, *settings*.

En este caso, el trabajo requiere datos del usuario.
Como no se los hemos indicado en la línea de comandos, nos preguntará.
Si deseamos pasar valores para las preguntas, podemos utilizar la opción ***-A***:

```bash
# listamos las preguntas
$ gattuso -g git -c vscode q

Location: git:///jobs/catalogs/vscode.yaml

Questions: settingsQ
 Name                  Type     Question                                              Default 
----------------------------------------------------------------------------------------------
 formatMarkdownOnSave  confirm  Would you like to format Markdown files on save?      false   
 lang                  input    cSpell.language (for example, es, en, fr, it...)      en      
 prettier              confirm  Would you like to configure Prettier for TypeScript?  false   

# invocamos el trabajo con las respuestas
$ gattuso -g git -c vscode r settings -A formatMarkdownOnSave=false -A lang=en -A prettier=false
> Create .vscode/settings
  - fs: create dir '/tmp/akro/.vscode' if not exists ok (1 ms)
  - file: write content to '/tmp/akro/.vscode/settings.json' ok (1 ms)
```

## Componentes de un catálogo

### Campo *spec*

El **campo *spec*** (*spec field*) indica la versión de la especificación utilizada para definir el catálogo.
Actualmente, debe ser **v1.0**.

### Campo *desc*

El **campo *desc*** (*desc field*) contiene una breve descripción del objeto del catálogo.

### Campo *dataset*

El **campo *dataset*** (*dataset field*) contiene las variables y constantes disponibles en el catálogo.
Podemos utilizarlas en cualquier punto del catálogo mediante el uso de expresiones.
En un capítulo posterior se describen detenidamente los conjuntos de datos.

### Campo *jobs*

El **campo *jobs*** (*jobs field*) contiene los trabajos definidos en el catálogo, los cuales podemos invocar desde la línea de comandos.

## Trabajos

Un **trabajo** (*job*) representa una tarea o proceso automatizado.
Cada trabajo se define mediante un determinado tipo de operación, ya sea simple o compuesta.
Una **operación** (*operation*) no es más que una acción como, por ejemplo, hacer una copia de seguridad, crear un contenedor de **Docker**, realizar una compilación, etc.

Una **operación simple** (*simple operation*) es la unidad de operación más pequeña e indivisible.
Mientras que una **operación compuesta** (*composite operation*) representa una colección de operaciones (simples o compuestas).

Los trabajos se definen mediante objetos, del campo ***jobs*** del catálogo.
Cada uno de ellos puede ser de un tipo u otro, pero tienen algunos campos comunes:

Campo | Tipo de datos | Descripción
-- | -- | --
***title*** | Texto | Título a mostrar en los informes de ejecución.
***desc*** | Texto | Breve descripción de lo que hace el trabajo.
***ini*** | Lista | Lista de operaciones iniciales a ejecutar.
***fin*** | Lista | Lista de operaciones finales a ejecutar.

## Operaciones

### Macros

Una **macro** (*macro*) es una secuencia ordenada de operaciones.
Se definen mediante un objeto con un campo **macro**, el cual se utiliza para indicar que estamos ante una macro y, por otra parte, su nombre.
Vamos a ver un ejemplo ilustrativo:

```yaml
- macro: create
  title: Create new job catalog
  ini:
    - quiet: $answers = inquire $(newJobCatalogQ) $(answers)
  steps:
    - fs.createDir $(dst)
    - quiet: $item = cr.getItem $(src)/job-catalog.yaml.hbs
    - quiet: $content = hbs.render $(item.value) $(answers)
    - file.write $(content) $(dst)/$(answers.name).yaml
```

Las operaciones realizadas por las macros se indican mediante los campos ***ini***, ***steps*** y ***fin***.
***ini*** contiene las iniciales, si fuera necesario.
***steps*** contiene las operaciones que debe ejecutar la macro.
Y ***fin*** contiene las finales, si fuera necesario.
En el ejemplo anterior, la macro necesita datos del usuario que deseamos solicitar al usuario.
Como esos datos son necesarios para el cuerpo de la macro, las añadimos a ***ini***.

## Ejemplo

A continuación, vamos a mostrar otro ejemplo más completo que utiliza **Docker** para levantar un entorno de desarrollo.
Este entorno estará formado por una instancia de **Redis**, otra de **PostgreSQL** y un servicio web cuyo código se encuentra en un directorio particular de nuestro equipo.

Veamos algunas cosas que debemos tener en cuenta antes de comenzar a diseñar nuestro catálogo y el trabajo que realizará el levantamiento de este entorno:

- El servicio web debe poder acceder a las instancias de **Redis** y **PostgreSQL** mediante los nombres de sus contendedores.
  Para ello, debemos crear una red de **Docker** y añadir los tres contenedores a ella.

- Realizaremos un *ping* automático desde el servicio web a los contenedores de bases de datos para comprobar que todo está bien configurado.

- El servicio web contendrá su código en el disco local.
  Por ello, tendremos que utilizar un 