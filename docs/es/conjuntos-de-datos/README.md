---
title: Conjuntos de datos
permalink: /es/conjuntos-de-datos
---

# Conjuntos de datos

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

Las operaciones, ya sean simples o compuestas, pueden definir su propio catálogo local.
Para ello, deben utilizar su **campo *local***, en vez de *dataset*.

## Acceso al conjunto de datos

**Akromio** permite el uso de expresiones en los campos.
Estas expresiones pueden contener referencias al conjunto de datos dispoinible mediante la siguiente sintaxis:

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
