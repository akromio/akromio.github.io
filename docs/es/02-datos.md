---
title: Datos
permalink: /es/datos
layout: page
---

Un **conjunto de datos** (*dataset*) es una colección de datos que podemos usar.
Atendiendo a quién los define, se distingue entre conjuntos de datos globales y locales.

El **conjunto de datos global** (*global dataset*) es aquel que viene con **Akromio**.
La colección de datos disponible en este conjunto se puede consultar fácilmente con el siguiente comando:

```bash
gattuso -g git -c empty dataset
```

Por otra parte, tenemos los **conjuntos de datos locales** (*local datasets*), aquellos que definimos nosotros mismos en un catálogo.

## Declaraciones de conjuntos de datos locales

### Declaración del conjunto de datos local de un catálogo

Los catálogos pueden tener su propio conjunto de datos local, el cual se define mediante el **campo *dataset*** (*dataset field*).
Lo siguiente es un ejemplo extraído de uno de los catálogos del registro predefinido [builtin-registry](https://github.com/akromio/builtin-registry):

```yaml
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
```

### Declaración del conjunto de datos local de una operación

Las operaciones, ya sean simples o compuestas, pueden definir su propio conjunto de datos local.
Deben hacerlo en su propio campo ***dataset***.

## Declaración de datos

Cada dato se define mediante un elemento objeto en el campo ***dataset***.
Disponemos de las siguientes posibilidades:

```yaml
# dato variable
- var: nombre
  value: valor a asignar

# dato constante, no modificable
- const: nombre
  value: valor a asignar
```

Los campos que pueden contener las declaraciones son los siguientes:

Campo | Descripción
-- | --
***desc*** | Breve descripción del campo.
***value*** | El valor a asignar. Puede contener expresiones.
***defaultValue*** | Valor a asignar si ***value*** es nulo.
***tags*** | Lista de etiquetas asociadas al dato.
***dataType*** | Tipo de dato esperado:***any***,  ***bool***, ***list***, ***map***, ***num*** o ***text***.
***options*** | Lista con los posibles valores asignables.

También es posible añadir datos de tipo **entrada** (*input*) que indican que su valor se tomará del argumento homónimo.
En vez de usar ***var*** o ***const***, se usa ***input***.
Lo siguiente es similar:

```yaml
# mediante constante
- const: nombre
  value: $(args.nombre)
  tags: [input]
  required: true

# mediante input
- input: nombre
```

## Acceso al conjunto de datos

**Akromio** permite el uso de expresiones en los campos.
Estas expresiones pueden contener referencias al conjunto de datos disponible mediante la siguiente sintaxis:

```yaml
$(variable)
```

El conjunto de datos global proporciona la constante ***workDir*** que contiene la ruta al directorio de trabajo.
Para hacer uso de ella, podemos hacer algo como lo siguiente:

```yaml
$(workDir)/.vscode
```

En un contexto local, el conjunto de datos estará formado por los datos globales y los locales.
Si lo deseamos, podemos declarar un dato en un contexto local con el mismo nombre que uno global, lo que ocultará el global.

## Argumentos

Un **argumento** (*argumento*) es un dato proporcionado en la línea de comandos o en una variable de entorno.
Se acceden en el catálogo mediante la constante ***args*** como, por ejemplo:

```yaml
value: $(args.nombre)
```

### Argumentos en la línea de comandos

Desde la línea de comandos, se utiliza la opción ***-a*** con uno de los siguientes formatos:

```
-a nombre=valor
-a archivo.yaml
-a archivo.json
```

Cuando se indica un archivo, este deberá ser un objeto o mapa, donde cada una de sus propiedades se considera un argumento.
Se debe indicar la extensión del archivo para así determinar en qué formato se encuentra el archivo.

Se puede indicar tantas opciones ***-a*** como sea necesario.

### Argumentos mediante variables de entorno

También es posible definir argumentos mediante variables de entorno.
Toda variable de entorno que comience por **KRM_ARG_** se considerará un argumento que se añadirá automáticamente a ***args***:

```
KRM_ARG_nombre=valor
```

El nombre del argumento sigue al prefijo indicado y su valor será el de la variable de entorno.

### Valores codificados

Cuando se utiliza una opción ***-a nombre=valor*** o una variable de entorno ***KRM_ARG_nombre=valor***, el valor puede precederse por el formato en el que se encuentra codificado.
Así, se pueden pasar valores compuestos como, por ejemplo, listas u objetos.
Para ello, es necesario que el valor esté precedido por los siguientes indicadores de protocolo:

Prefijo | Descripción
-- | --
***json://*** | Codificado en **JSON**.
***json+base64://*** | Codificado en **JSON** y su resultado en **base64**.

La segunda opción se recomienda cuando el valor puede contener espacios, si no los contiene, no hace falta.
Esto es muy útil cuando se despliega **Akromio** en un contenedor y se configura mediante variables de entorno.

Ejemplo:

```bash
export KRM_ARG_lista=json://[1,2,3,4]
```

Si dispone de un archivo local de argumentos, puede obtener sus representaciones en **json** o **json+base64** con el comando ***encode*** de las aplicaciones de la suite como, por ejemplo:

```bash
$ carboni encode args.yaml -x
export KRM_ARG_pauseDuration=json+base64://IjMwcyI=
export KRM_ARG_botnet=json+base64://eyJpbXBsIjoiY29uc29sZSIsImJvdHMiOlt7ImJvdCI6ImNhdmFuaTEifSx7ImJvdCI6ImNhdmFuaTIifSx7ImJvdCI6ImNhdmFuaTMifV19
```

Si necesita sólo uno de los argumentos, utilice la opción ***-p*** para indicar qué propiedad del archivo desea transformar:

```bash
$ carboni encode args.yaml -p botnet -x
export KRM_ARG_botnet=json+base64://eyJpbXBsIjoiY29uc29sZSIsImJvdHMiOlt7ImJvdCI6ImNhdmFuaTEifSx7ImJvdCI6ImNhdmFuaTIifSx7ImJvdCI6ImNhdmFuaTMifV19
```

La opción ***-x***  indica que prefije la salida con ***export*** para facilitar el copia y pega.
