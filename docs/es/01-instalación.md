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
