---
title: Instalación
permalink: /es/instalacion
---

**Akromio** es una *suite* de automatización desarrollada en **Node.js**, por lo que podemos descargar sus aplicaciones fácilmente del registro **NPM**.
Podemos hacerlo, manualmente y, en caso de **GitHub Actions**, mediante acciones.
Las aplicaciones disponibles actualmente son:

Aplicación | Paquete | URL | Descripción
-- | -- | -- | --
**gattuso** | **@akromio/gattuso** | [https://www.npmjs.com/package/@akromio/gattuso](https://www.npmjs.com/package/@akromio/gattuso) | Automatización de propósito general.

## Requisitos

Tener instalado **Node.js** como mínimo la versión **{{ site.data.dep.nodejs.version }}**.
Actualmente, los sistemas operativos probados son: **{{ site.data.dep.nodejs.os }}**.

## Instalación local

Para instalar las aplicaciones localmente, podemos utilizar los siguientes comandos:

Aplicación | Comando de instalación
-- | --
**gattuso** | `npm i -g @akromio/gattuso`

Después de la instalación, es buena práctica comprobar que tenemos acceso a las aplicaciones instaladas mostrando su ayuda o su versión:

```bash
gattuso -v
gattuso help
```

## Instalación  en flujo de *GitHub Actions*

Para **GitHub Actions**, están disponibles las siguientes acciones:

Aplicación | Acción | URL
-- | -- | --
**gattuso** | `akromio/setup-gattuso` | [https://github.com/marketplace/actions/setup-gattuso](https://github.com/marketplace/actions/setup-gattuso)

Ejemplo:

```yaml
- name: Set up Gattuso
  uses: akromio/setup-gattuso@v1
```

## Comandos de las aplicaciones

### Variables de entorno

Para conocer las variables de entorno utilizadas por **Akromio**, podemos utilizar el comando **env**:

```bash
# lista todas las variables de entorno
gattuso e

# lista la variable de entorno indicada
gattuso e VARIABLE
```

### Información del sistema

De cara a informar de un *bug*, es buena práctica obtener cierta información del sistema.
Para esta situación, podemos utilizar el comando **sys**:

```bash
$ gattuso sys
OS: linux (5.15.0-56-generic) #62-Ubuntu SMP Tue Nov 22 19:54:14 UTC 2022
Arch: x64
Node.js: v18.9.0
```
