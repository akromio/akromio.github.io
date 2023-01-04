---
title: Instalación
permalink: /es/instalacion
---

**Akromio** es una *suite* de automatización desarrollada en **Node.js**, por lo que podemos descargar sus aplicaciones fácilmente del registro **NPM**.
Podemos hacerlo, manualmente y, en caso de **GitHub Actions**, mediante acciones.
También hay disponible una imagen en el registro de **Docker**.
Las aplicaciones disponibles actualmente son:

Aplicación | Paquete | URL | Descripción
-- | -- | -- | --
**gattuso** | **@akromio/gattuso** | [https://www.npmjs.com/package/@akromio/gattuso](https://www.npmjs.com/package/@akromio/gattuso) | Automatización de propósito general.
**carboni** | **@akromio/carboni** | [https://www.npmjs.com/package/@akromio/carboni](https://www.npmjs.com/package/@akromio/carboni) | Generador y distribuidor de solicitudes de ejecución a agentes **Cavani**.
**cavani** | **@akromio/cavani** | [https://www.npmjs.com/package/@akromio/cavani](https://www.npmjs.com/package/@akromio/cavani) | Agente de ejecución en un entorno distribuido.

## Requisitos

Tener instalado **Node.js**, como mínimo la versión **{{ site.data.dep.nodejs.version }}**.
Actualmente, los sistemas operativos probados son: **{{ site.data.dep.nodejs.os }}**.

## Instalación local

Para instalar las aplicaciones localmente, podemos utilizar los siguientes comandos:

Aplicación | Comando de instalación
-- | --
**gattuso** | `npm i -g @akromio/gattuso`
**carboni** | `npm i -g @akromio/carboni`
**cavani** | `npm i -g @akromio/cavani`

Después de la instalación, es buena práctica comprobar que tenemos acceso a las aplicaciones instaladas mostrando su ayuda o su versión, por ejemplo:

```bash
gattuso -v
gattuso help
```

## Instalación en flujos de trabajo de *GitHub Actions*

Para **GitHub Actions**, están disponibles las siguientes acciones:

Aplicación | Acción | URL
-- | -- | --
**gattuso** | `akromio/setup-gattuso` | [https://github.com/marketplace/actions/setup-gattuso](https://github.com/marketplace/actions/setup-gattuso)

Ejemplo:

```yaml
- name: Set up Gattuso
  uses: akromio/setup-gattuso@v1
```

## Uso de la imagen de *Docker*

Cada aplicación de la *suite* dispone de su propia imagen de **Docker**:

Imagen | URL
-- | --
**akromio/gattuso** | [https://hub.docker.com/r/akromio/gattuso](https://hub.docker.com/r/akromio/gattuso)
**akromio/carboni** | [https://hub.docker.com/r/akromio/carboni](https://hub.docker.com/r/akromio/carboni)
**akromio/cavani** | [https://hub.docker.com/r/akromio/cavani](https://hub.docker.com/r/akromio/cavani)


A continuación, un ejemplo en el que se indica que el registro que debe utilizar es el contenido en el directorio *registry* del directorio actual, el cual se debe montar en la ruta **/registry** del contenedor:

```bash
# crea un contenedor que ejecute:
#   gattuso -c udp-flood r attack -a at=1671365957564
$ sudo docker run --name=udp-attacker5 --network=udp-flood -v $PWD/registry:/registry -id akromio/gattuso:latest gattuso -c udp-flood r attack -a at=1671365957564
```

## Comandos de las aplicaciones

### Variables de entorno

Para conocer las variables de entorno utilizadas por **Akromio**, podemos utilizar el comando **env**:

```bash
# lista todas las variables de entorno
gattuso e
carboni e
cavani e

# lista la variable de entorno indicada
gattuso e VARIABLE
carboni e VARIABLE
cavani e VARIABLE
```

### Información del sistema

De cara a informar de un *bug*, es buena práctica obtener cierta información del sistema.
Para esta situación, podemos utilizar el comando **sys** de las aplicaciones como, por ejemplo:

```bash
$ gattuso sys
gattuso: v0.13.1
OS: linux (5.15.0-56-generic) #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022
Arch: x64
Node.js: v18.9.0
```
