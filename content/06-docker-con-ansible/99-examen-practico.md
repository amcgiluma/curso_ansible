---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Debes instalar Docker en una VM y demostrar que queda utilizable.

## Teoría

La instalación automatizada solo cuenta si el resultado se puede validar con comandos objetivos.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/install-docker.yml
ansible docker_hosts -m command -a "docker run --rm hello-world"
# OUT
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0

vm-docker-01 | CHANGED | rc=0 >>
Hello from Docker!
...
```

## Flags y variantes

| Elemento | Para qué sirve |
| --- | --- |
| `become: true` | Instalar paquetes y gestionar servicio |
| `docker_users` | Usuarios que podrán ejecutar Docker |
| `--check` | Simulación parcial |
| `--tags docker` | Ejecutar solo el rol Docker |

## Pruébalo tú

1. Ejecuta instalación.
2. Reconecta SSH si cambió el grupo del usuario.
3. Valida Docker y Compose.
4. Ejecuta `hello-world`.
5. Repite el playbook y revisa idempotencia.

## Errores comunes

- **No reconectar después de cambiar grupos**: el permiso no aparece en la sesión actual.
- **Confundir Docker Compose v1 con plugin v2**: el comando esperado es `docker compose`, no `docker-compose`.

> Idea clave: este módulo te da la pieza que necesitarás en la VM creada automáticamente por Proxmox.
