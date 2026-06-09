---
title: "VM docker-ready en Proxmox"
slug: "vm-docker-ready"
order: 1
summary: "Unir Proxmox y Docker en un flujo de aprovisionamiento completo."
---

# VM docker-ready en Proxmox

Este es el objetivo del curso: lanzar una VM desde Proxmox y dejarla con Docker instalado usando Ansible.

## Teoría

El flujo completo tiene dos fases:

| Fase | Dónde corre | Qué hace |
| --- | --- | --- |
| Crear VM | `localhost` contra API Proxmox | Clona plantilla cloud-init |
| Provisionar VM | SSH contra la VM nueva | Instala Docker y valida |

Separar fases evita mezclar API de Proxmox con tareas de Linux. El playbook final usa `provision-docker-vm.yml` para expresar ese flujo.

## Manos a la obra

Ejecuta el playbook final:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/provision-docker-vm.yml
# OUT
PLAY [Crear VM cloud-init en Proxmox] *****************************************
changed: [localhost]

PLAY [Esperar SSH en la VM nueva] *********************************************
ok: [localhost]

PLAY [Instalar Docker en la VM creada] ****************************************
ok: [vm-docker-01]

PLAY RECAP *******************************************************************
localhost    : ok=<n> changed=<n> unreachable=0 failed=0
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
```

## Flags y variantes

| Opción | Para qué sirve |
| --- | --- |
| `wait_for` | Espera a que SSH esté disponible |
| `delegate_to: localhost` | Ejecuta comprobaciones desde el nodo de control |
| `add_host` | Añade dinámicamente la VM recién creada al inventario |
| `serial` | Controla despliegues por lotes |
| `--limit` | Acota si reutilizas inventarios grandes |

## Pruébalo tú

1. Configura variables reales en `group_vars/all.yml`.
2. Ejecuta `provision-docker-vm.yml`.
3. Espera a que la VM arranque y acepte SSH.
4. Valida `docker run --rm hello-world`.
5. Guarda el playbook como base para futuras VMs del homelab.

## Errores comunes

- **Intentar provisionar antes de SSH**: usa espera explícita.
- **IP dinámica desconocida**: para el primer flujo usa IP fija o registra la IP esperada.
- **Mezclar creación y configuración en un rol gigante**: mantén roles separados.

> Idea clave: infraestructura y configuración son fases distintas; juntas te dan una VM lista para ejecutar contenedores.
