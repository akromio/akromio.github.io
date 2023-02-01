---
permalink: /
layout: home
---

**Akromio** es una *suite* de automatización para IT con la que se puede:

- Automatizar la ejecución de tareas cotidianas como, por ejemplo, la compilación, la generación de estructuras de directorios, etc.

- Automatizar procesos de instalación.

- Realizar pruebas de carga o ataques de denegación de servicio para comprobar si la configuración de nuestros sistemas reacciona como esperamos.

- Realizar aserciones en pruebas de unidad, integración o sistema.

- Utilizar dobles de pruebas en las pruebas de unidad o integración.

Ejemplo:

```yaml
spec: v1.0
desc: Catalog for working with Wireshark.

jobs:
  - macro: setup
    title: Set up Wireshark
    steps:
      - sudo: apt update
      - sudo: apt install -y wireshark
      - exec.log wireshark --version
  
  - macro: conf
    title: Configure Wireshark
    dataset:
      - const: addToGroup
        desc: Add current user to wireshark group.
        value: $(args.group)
        defaultValue: yes
    steps:
      - sudo: dpkg-reconfigure -fnoninteractive wireshark-common
      - if: addToGroup
        step: exec sudo usermod -aG wireshark $(user.name)
      - banner PLEASE, REBOOT

  - macro: remove
    title: Remove Wireshark if installed
    steps:
      - sudo: apt autoremove -y wireshark
```
