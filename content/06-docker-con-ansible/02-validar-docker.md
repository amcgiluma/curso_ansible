---
title: "Validar Docker instalado"
slug: "validar-docker"
order: 2
summary: "Comprobar servicio, permisos y ejecución de un contenedor de prueba."
---

# Validar Docker instalado

Instalar no basta. Debes comprobar que el servicio está activo, que el usuario puede usarlo y que un contenedor arranca.

## Teoría

Validación mínima:

| Comprobación | Qué demuestra |
| --- | --- |
| `systemctl is-active docker` | Servicio levantado |
| `docker --version` | CLI disponible |
| `docker compose version` | Compose plugin instalado |
| `docker run hello-world` | Motor capaz de descargar y ejecutar |

En automatización, una validación clara evita descubrir errores más tarde cuando despliegues monitorización.

## Manos a la obra

Comprueba Docker desde Ansible:

```compare
# CMD
cd examples/ansible-homelab
ansible docker_hosts -m command -a "systemctl is-active docker"
ansible docker_hosts -m command -a "docker compose version"
ansible docker_hosts -m command -a "docker run --rm hello-world"
# OUT
vm-docker-01 | CHANGED | rc=0 >>
active
vm-docker-01 | CHANGED | rc=0 >>
Docker Compose version v<version>
vm-docker-01 | CHANGED | rc=0 >>
Hello from Docker!
...
```

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `systemctl is-active docker` | Comprueba estado del servicio |
| `docker compose version` | Verifica plugin Compose |
| `docker run --rm hello-world` | Ejecuta prueba limpia |
| `--rm` | Borra el contenedor al terminar |
| `changed_when: false` | Marca comandos de comprobación como no cambiantes en playbooks |

## Pruébalo tú

1. Ejecuta las tres comprobaciones.
2. Si `hello-world` falla por permisos, reconecta SSH.
3. Si falla por red, comprueba DNS y salida a Internet de la VM.
4. Ejecuta `docker ps -a` y confirma que no quedó contenedor si usaste `--rm`.
5. Anota qué comprobación usarías en un playbook final.

## Errores comunes

- **`permission denied` al socket Docker**: usuario fuera del grupo o sesión antigua.
- **`docker compose` no existe**: falta el plugin, no el motor.
- **Marcar checks como `changed`**: para validaciones en playbooks, usa `changed_when: false`.

> Idea clave: una VM docker-ready debe pasar servicio activo, CLI disponible, Compose disponible y contenedor de prueba.
