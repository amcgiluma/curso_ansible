---
title: "Clonar plantilla cloud-init"
slug: "clonar-template-cloud-init"
order: 2
summary: "Crear una VM desde una plantilla Proxmox preparada con cloud-init."
---

# Clonar plantilla cloud-init

La forma práctica de crear VMs repetibles en Proxmox es preparar una plantilla cloud-init y clonarla con parámetros: nombre, VMID, CPU, RAM, disco, usuario y clave SSH.

## Teoría

Cloud-init permite que la VM arranque ya personalizada:

| Parámetro | Uso |
| --- | --- |
| `clone` | Nombre o ID de la plantilla |
| `vmid` | ID único de la VM nueva |
| `name` | Nombre visible en Proxmox |
| `cores`, `memory` | Recursos |
| `ciuser` | Usuario inicial |
| `sshkeys` | Clave pública autorizada |
| `ipconfig` | DHCP o IP fija |

Este curso asume que ya existe una plantilla Ubuntu cloud-init en Proxmox.

## Manos a la obra

Ejecuta el playbook de creación con variables reales:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/proxmox-create-vm.yml
# OUT
PLAY [Crear VM cloud-init en Proxmox] *****************************************

TASK [proxmox_vm : Clonar VM desde template cloud-init] ***********************
changed: [localhost]

PLAY RECAP *******************************************************************
localhost : ok=<n> changed=1 unreachable=0 failed=0
```

## Flags y variantes

| Opción | Para qué sirve |
| --- | --- |
| `full: true` | Clon completo en vez de linked clone |
| `storage` | Storage destino de la VM |
| `net` | Configuración de red virtual |
| `ipconfig` | DHCP o IP fija con gateway |
| `timeout` | Tiempo máximo para operaciones Proxmox |
| `validate_certs` | Validación TLS del API |

## Pruébalo tú

1. Confirma que tienes una plantilla cloud-init en Proxmox.
2. Rellena variables de `group_vars/all.yml`.
3. Ejecuta el playbook con `--check` solo para revisar alcance, si el módulo lo soporta en tu versión.
4. Ejecuta `proxmox-create-vm.yml`.
5. Comprueba en Proxmox que aparece la VM.

## Errores comunes

- **VMID duplicado**: Proxmox rechazará la creación.
- **Plantilla no preparada con cloud-init**: la VM clona, pero no aplica usuario/clave/red.
- **Red bridge mal configurada**: la VM arranca sin conectividad útil.

> Idea clave: la plantilla cloud-init es la base; Ansible solo parametriza y dispara clones repetibles.
