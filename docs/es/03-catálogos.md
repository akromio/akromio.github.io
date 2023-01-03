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
    - quiet: $answers = inquire $(createQ) $(answers)
  steps:
    - fs.createDir $(dst)
    - quiet: $item = cr.getItem $(src)/catalog.yaml.hbs
    - quiet: $content = hbs.render $(item.value) $(answers)
    - file.write $(content) $(dst)/$(answers.name).yaml
```

Las operaciones realizadas por las macros se indican mediante los campos ***ini***, ***steps*** y ***fin***.
***ini*** contiene las iniciales, si fuera necesario.
***steps*** contiene las operaciones que debe ejecutar la macro.
Y ***fin*** contiene las finales, si fuera necesario.
En el ejemplo anterior, la macro necesita datos del usuario que deseamos solicitar al usuario.
Como esos datos son necesarios para el cuerpo de la macro, las añadimos a ***ini***.

Si ***ini*** o ***fin*** contienen un único paso, se puede indicar en línea.
Ejemplo:

```yaml
- macro: create-attackers
  title: Create the attacker containers
  ini: exec sudo docker pull $(attacker.image)
  forEach: $i = range 1 5
  steps:
    - exec sudo docker run --name $(attacker.conttainer)$(i) --rm $(attacker.image)
```

#### Macros en bucle

Las macros también pueden ejecutarse en bucle para cada valor de una lista.
Se definen exactamente igual que una macro normal, salvo que añaden el campo ***forEach*** con el que se indica la operación que debe devolver la lista por la que iterar.
En el ejemplo anterior, se usa la operación ***range*** la cual devuelve una lista con una secuencia de números que van de uno a otro, ambos inclusive.
Así pues, lo anterior sería lo mismo que lo siguiente:

```yaml
- macro: create-attackers
  title: Create the attacker containers
  ini: exec sudo docker pull $(attacker.image)
  steps:
    - exec sudo docker run --name $(attacker.conttainer)1 --rm $(attacker.image)
    - exec sudo docker run --name $(attacker.conttainer)2 --rm $(attacker.image)
    - exec sudo docker run --name $(attacker.conttainer)3 --rm $(attacker.image)
    - exec sudo docker run --name $(attacker.conttainer)4 --rm $(attacker.image)
    - exec sudo docker run --name $(attacker.conttainer)5 --rm $(attacker.image)
```

La ejecución de una macro en bucle es como sigue:

01. Se ejecutan las operaciones definidas en el ***ini***, si las hubiera.

02. Se ejecuta la operación definida en el campo ***forEach*** para obtener la lista por la que iterar.

03. Se ejecutan los pasos de la macro una vez para cada elemento de la lista.

04. Se ejecutan las operaciones definidas en el ***fin***, si las hubiera.

#### Ejecución aleatoria de pasos

En algunas ocasiones, puede ser útil ejecutar los pasos aleatoriamente, por ejemplo, para evitar patrones de ejecución.
Esto no siempre es posible, pero en algunas ocasiones sí.
Para este fin, fijaremos la propiedad ***random*** de la macro a ***true***.

## Pasos

Un **paso** (*step*) no es más que una acción a realizar en una operación compuesta.
Se puede indicar en las secciones ***ini***, ***steps*** o ***fin***.
Se pueden expresar de varias formas.

### Pasos textuales

El paso contiene en una cadena de texto la operación y los argumentos a pasar.
Por ejemplo:

```yaml
- cr.copy $(src)/_gitignore $(dst)/.gitignore
```

### Pasos en lista

El paso se indica como una lista de elementos donde el primero es la operación y los restantes son sus argumentos.
Ejemplo:

```yaml
- [cr.copy, $(src)/_gitignore, $(dst)/.gitignore]
```

### Pasos en objetos

Otra posibilidad es indicar el paso en forma de objeto.
Sintaxis:

```yaml
# paso
- step: pasoTextualOEnLista

# paso silencioso
- quiet: pasoTextualOEnLista

# paso que muestra su log
- log: pasoTextualOEnLista
```

### Pasos condicionales

De manera predeterminada, todo paso se ejecuta cuando le llega el turno.
En ocasiones, es preferible un **paso condicional** (*conditional step*), aquel que sólo debe ejecutarse si una condición asociada se cumple.
Este tipo de pasos se indican en modo objeto y en su **campo *if*** (*if field*) indicaremos la condición a cumplirse:

```yaml
- step: pasoTextualOEnLista
  if: condición
```

Aunque actualmente se puede indicar cualquier expresión **JavaScript** válida, sólo debe utilizar los operadores de indexación (`[]`), acceso a campo (`.`) y condicionales (`==`, `!=`, `<`, `<=`, `>`, `>=`).
Y use sólo las comillas simples para los textos literales (`'`), no las dobles (`"`).
En el futuro, el evaluador sólo aceptará esto.
El contexto de datos con los que trabajará el evaluador de la expresión es el *dataset* disponible en ese momento.
Ejemplo:

```yaml
if: platform == 'linux'
```

## Constructor de un catálogo de trabajos

Podemos construir un catálogo de trabajos fácilmente mediante el siguiente comando:

```bash
$ gattuso -g git -c job r create
```
