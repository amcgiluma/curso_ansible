---
title: "Instalar Docker Engine"
slug: "instalar-docker-engine"
order: 1
summary: "Automatizar la instalación de Docker en hosts Ubuntu/Debian."
---

# Instalar Docker Engine

El objetivo no es estudiar Docker por dentro, sino dejarlo instalado de forma repetible en cada VM que crees para tu homelab.

## Teoría

Para una VM Ubuntu/Debian, la instalación práctica incluye:

| Paso | Motivo |
| --- | --- |
| Paquetes base | Certificados, curl y herramientas APT |
| Paquetes Docker | Motor y plugin Compose disponibles desde APT |
| Docker Engine | Servicio `docker` |
| Compose plugin | `docker compose` moderno |
| Grupo `docker` | Permite usar Docker sin sudo al usuario elegido |

El rol `docker` del ejemplo usa módulos Ansible (`apt`, `service`, `user`) en vez de comandos shell sueltos. Para un homelab es suficiente como base; si más adelante necesitas una versión concreta, puedes adaptar el rol al repositorio oficial de Docker.

## Manos a la obra

Ejecuta la instalación:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/install-docker.yml
ansible docker_hosts -m command -a "docker --version"
# OUT
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0

vm-docker-01 | CHANGED | rc=0 >>
Docker version <version>, build <hash>
```

## Flags y variantes

| Recurso | Para qué sirve |
| --- | --- |
| `ansible.builtin.apt` | Instala paquetes de forma idempotente |
| `update_cache: true` | Actualiza caché APT cuando hace falta |
| `ansible.builtin.service` | Gestiona el servicio Docker |
| `ansible.builtin.user` | Añade el usuario al grupo `docker` |
| `become: true` | Ejecuta tareas con privilegios |

## Pruébalo tú

1. Asegúrate de que tu VM está en `docker_hosts`.
2. Ejecuta `install-docker.yml`.
3. Comprueba `docker --version`.
4. Cierra y reabre sesión SSH si acabas de añadir el usuario al grupo `docker`.
5. Ejecuta el playbook una segunda vez y revisa cambios.

## Errores comunes

- **El usuario sigue sin poder usar Docker**: los grupos se aplican al iniciar una nueva sesión.
- **Usar paquetes antiguos de distro sin querer**: decide si los paquetes del sistema te bastan o si necesitas fijar el repositorio oficial de Docker.
- **Instalar con scripts curl pipe shell**: puede funcionar, pero es menos auditable que tareas Ansible.

> Idea clave: Ansible convierte la instalación de Docker en una operación repetible para cualquier VM nueva.
