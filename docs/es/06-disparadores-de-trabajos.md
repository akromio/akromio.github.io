---
title: Disparadores de trabajos
permalink: /es/disparadores-de-trabajos
---


Los trabajos de un catálogo se pueden ejecutar de manera manual o bien bajo disparadores.
Un **disparador** (*trigger*) no es más que un componente que tiene la capacidad de ejecutar un trabajo cuando se produce un determinado tipo de evento.
Un **evento** (*event*) es algo acaecido en el sistema y que tiene la capacidad de disparar o producir la ejecución de trabajos.

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
```

## Comando *trigger*

mediante el comando **trigger** (o **t**) solicitamos la ejecución de un trabajo mediante un disparador.
En vez de usar **run**, debemos usar **trigger**.

Ejemplos:

```bash
# ejecuta el trabajo predeterminado
# bajo el disparador predeterminado
gattuso t

# ejecuta el trabajo indicado
# bajo el disparador predeterminado
gattuso t trabajo

# ejecuta el trabajo predeterminado
# bajo el disparador indicado
gattuso -t disparador t

# ejecuta el trabajo indicado
# bajo el disparador indicado
gattuso -t disparador t trabajo
```

Recuerde que, cuando deseamos usar un disparador específico, se utiliza la opción ***-t***.
El trabajo siempre sin opción, para ser consistentes con el comando **run**.

Se puede pasar argumentos al trabajo a ejecutar con la opción **-a**, al igual que con el comando **run**.

## Disparador a intervalos

El **disparador a intervalos** (*interval trigger*) es aquel que ejecuta un determinado trabajo de manera periódica.
Se configura mediante las propiedades siguientes del disparador:

Propiedad | Descripción
-- | --
***interval*** | Cada cuánto tiempo se debe ejecutar como, por ejemplo, *1h*, *10m* o *1000s*.
***immediate*** | ¿La primera ejecución es inmediata? *true* por defecto.
***times*** | Número de veces que debe ejecutarse. Si no se indica, infinitamente.

Ejemplo:

```yaml
on:
  - trigger: cada-hora
    impl: interval
    desc: Genera el evento cada hora.
    interval: 1h
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

defaultJobName: backup
jobs:
  - macro: backup
    local: [ts]
    steps:
      - $ts = timestamp.now
      - fs.copy $(src) $(dst).$(ts)
```

Aquí un ejemplo de su ejecución:

```bash
gattuso t -t interval -a filePath=/mi/archivo.txt -a interval=12h
```