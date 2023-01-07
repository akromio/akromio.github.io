---
title: Disparadores
permalink: /es/disparadores
---

Los trabajos de un catálogo se pueden ejecutar de manera manual o bien bajo disparadores.
Un **disparador** (*trigger*) no es más que un componente que tiene la capacidad de ejecutar un trabajo cuando se produce un determinado tipo de evento.
Un **evento** (*event*) es algo acaecido en el sistema y que tiene la capacidad de disparar o producir la ejecución de trabajos.

La ejecución dirigida por eventos recae en **Cavani**.

## Propiedad *on* de los catálogos

Estos disparadores se definen mediante la propiedad ***on*** del catálogo.
Cada disparador tiene su propio identificador de tipo.
Esta propiedad consiste en una lista de objetos donde cada uno de ellos representa un determinado disparador.
Podemos tener varios disparadores del mismo tipo, pero siempre con sus configuraciones particulares.

Ejemplo:

```yaml
# defaultTriggerName: nombreDeDisparadorPredeterminado
on:
  - trigger: interval # nombre para identificar el disparador
    impl: interval    # si no se indica, se usa el identificador
    job: nombre       # nombre del trabajo a ejecutar
    args: valor       # argumentos a pasar al trabajo a ejecutar
```

## Comando *cavani run*

Mediante el comando **cavani run** (o **cavani r**) solicitamos la ejecución de un trabajo mediante un disparador.

Ejemplos:

```bash
# ejecuta el disparador predeterminado
cavani r

# ejecuta el disparador indicado
cavani r disparador
```

Se puede pasar argumentos al trabajo a ejecutar con la opción **-a**, al igual que con el comando **gattuso run**.

## Disparador a intervalos

El **disparador a intervalos** (*interval trigger*) es aquel que ejecuta un determinado trabajo de manera periódica.
Se configura mediante las propiedades siguientes del disparador:

Propiedad | Descripción
-- | --
***interval*** | Cada cuánto tiempo se debe ejecutar como, por ejemplo, *1h*, *10m* o *1000s*.
***immediate*** | ¿La primera ejecución es inmediata? *true* por defecto.
***times*** | Número de veces que debe ejecutarse. Si no se indica, infinitamente.
***job*** | Nombre del trabajo a ejecutar.
***args*** | Argumentos a pasar, si es necesario, al trabajo.

Ejemplo:

```yaml
on:
  - trigger: cada-hora
    impl: interval
    interval: 1h
    job: backup
```

## Ejemplo

A continuación, se muestra un ejemplo con el que hacer una copia de seguridad de un determinado archivo cada hora si no se especifica otro intervalo explícitamente:

```yaml
spec: v1.0
desc: Realiza una copia de seguridad de un archivo.

dataset:
  - const: interval
    value: $(args.interval)
    defaultValue: 1h

  - const: src
    value: $(args.filePath)
  
  - const: dst
    value: $(src).bak

on:
  - trigger: interval
    interval: $(interval)
    job: backup

jobs:
  - macro: backup
    dataset: [ts]
    steps:
      - $ts = timestamp.now
      - fs.copy $(src) $(dst).$(ts)
```

Aquí un ejemplo de su ejecución:

```bash
cavani r interval -a filePath=/mi/archivo.txt -a interval=12h
```
