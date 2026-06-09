---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Reto final para crear una VM en Proxmox con Docker instalado y validado."
---

# Examen práctico

El reto final es crear una VM en Proxmox automáticamente, esperar SSH, instalar Docker y validar que ejecuta contenedores.

## Teoría

Has unido las piezas: WSL como nodo de control, inventario, playbooks, roles, Docker y Proxmox cloud-init.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/provision-docker-vm.yml
ansible docker_hosts -m command -a "docker run --rm hello-world"
# OUT
PLAY RECAP *******************************************************************
localhost    : ok=<n> changed=<n> unreachable=0 failed=0
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0

vm-docker-01 | CHANGED | rc=0 >>
Hello from Docker!
...
```

## Flags y variantes

| Elemento | Para qué sirve |
| --- | --- |
| `provision-docker-vm.yml` | Flujo completo |
| `proxmox-create-vm.yml` | Solo infraestructura |
| `install-docker.yml` | Solo configuración Docker |
| `--check --diff` | Revisión previa cuando aplique |
| `--limit` | Evitar tocar hosts no deseados |

## Pruébalo tú

1. Prepara variables reales.
2. Ejecuta instalación de colecciones.
3. Lanza el playbook final.
4. Verifica VM en Proxmox.
5. Verifica SSH y Docker.
6. Escribe qué cambiarías para crear una segunda VM.

## Errores comunes

- **No tener plantilla cloud-init lista**: el playbook no sustituye esa preparación.
- **IP/VMID repetidos**: define una convención antes de crear muchas VMs.
- **No validar Docker al final**: crear VM no significa que esté lista para contenedores.

> Idea clave: si puedes repetir este examen, ya tienes una base real para automatizar tu homelab.
