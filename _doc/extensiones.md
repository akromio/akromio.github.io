---
title: Extensiones
order: 5
---

La **extensión de un catálogo** (*extension catalog*) es la capacidad de reutilizar un catálogo existente.
Por ejemplo, en el desarrollo de monorepos en **Node.js** es muy útil definir un catálogo común que hereden o extiendan los catálogos particulares de los paquetes.
En **Akromio**, esto se soporta mediante el registro base y la propiedad ***extends*** de los catálogos.

## Registro base

El **registro base** (*base registry*) es un registro definido automáticamente y que permite el acceso a un catálogo definido en un directorio superior.
Por ejemplo, en un monorepo, podemos utilizar los catálogos definidos en el directorio **/.akromio/jobs/catalogs** como el repositorio base.
Cada vez que un paquete de este repositorio quiera extender uno de estos catálogos, no tendrá más que hacer referencia a él mediante el registro **base**.
Ejemplo:

```
base:///_package.yaml
```

Los catálogos que comienzan por un subrayado se conocen como internos y no se listan mediante el comando **gattuso g**.
Se utilizan principalmente para indicar catálogos reutilizables.

## Extensión de un catálogo base

Para indicar que un catálogo extiende otro existente en un registro base, utilizaremos la propiedad ***extends*** del catálogo.
He aquí un ejemplo ilustrativo:

```yaml
spec: v1.0
desc: Catalog for automating the project tasks.
extends: base:///_itg.yaml

dataset:
  - const: redis
    value:
      image: redislabs/rejson
      container: vql-redis
      port: 6379

jobs:
  - group: redis
    jobs:
    - macro: redis/start
      desc: Start the Redis container
      steps:
        - exec.log sudo docker pull $(redis.image)
        - exec.log sudo docker run --name $(redis.container) --rm -d -p $(redis.port):6379 $(redis.image)

    - macro: redis/stop
      desc: Stop the Redis container
      steps:
        - exec.log sudo docker stop $(redis.container)
```

En el catálogo que extiende, podemos añadir nuevos conjuntos de datos, trabajos, etc.
En caso de conflicto, los aquí definidos sobrescribirán a los indicados en el catálogo reutilizable.

## Ejemplo

En el proyecto **ValenciaQL**, [https://github.com/valenciadb/nodejs-vql](https://github.com/valenciadb/nodejs-vql), puede encontrar un ejemplo de reutilización de catálogos en un monorepo de **Node.js**.
