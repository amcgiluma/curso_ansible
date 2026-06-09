---
title: "Checkpoint: primera VM automatizada"
slug: "examen-practico"
order: 99
summary: "Comprobar que ya puedes crear una VM cloud-init útil desde Ansible."
---

# Checkpoint: primera VM automatizada

Este checkpoint no va de memorizar comandos. Va de confirmar que tienes una cadena mínima de infraestructura como código: Proxmox preparado, template cloud-init lista, variables claras y un playbook que crea una VM gestionable por SSH.

## Teoría

El módulo queda bien cerrado cuando puedes explicar y demostrar estas piezas:

| Pieza | Qué debes tener claro |
| --- | --- |
| API token | Qué usuario usa Ansible y dónde está guardado el secreto |
| Template cloud-init | Cómo se creó y por qué no es una VM normal |
| Variables | Qué cambia entre una VM y otra |
| Playbook | Qué parte llama a Proxmox y qué parte entra por SSH |
| Resultado | VM creada, arrancada y accesible |

La prueba importante no es que Proxmox muestre una VM cualquiera. La prueba importante es que puedas repetir el proceso cambiando `proxmox_vm_id`, `proxmox_vm_name` e IP.

## Manos a la obra

Desde el ejemplo del curso:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/proxmox-create-vm.yml
# OUT
PLAY [Crear VM cloud-init en Proxmox] *****************************************
...
TASK [proxmox_vm : Clonar VM desde template cloud-init] ***********************
changed: [localhost]
...
TASK [proxmox_vm : Arrancar VM] ***********************************************
changed: [localhost]

PLAY RECAP *******************************************************************
localhost : ok=<n> changed=2 unreachable=0 failed=0
```

Después valida SSH:

```compare
# CMD
ssh ubuntu@192.168.1.50 hostname
# OUT
vm-docker-01
```

Si quieres cerrar el módulo con el flujo completo:

```bash
ansible-playbook playbooks/provision-docker-vm.yml
```

Ese playbook crea la VM, espera SSH e instala Docker. Es el entregable más útil de este bloque.

## Flags y variantes

| Variable | Para qué sirve |
| --- | --- |
| `proxmox_node` | Nodo donde crear la VM |
| `proxmox_template` | Plantilla cloud-init |
| `proxmox_vm_id` | ID único de la VM |
| `proxmox_vm_name` | Nombre de la VM |
| `proxmox_vm_ipconfig` | Red inyectada por cloud-init |
| `proxmox_vm_ssh_public_key` | Clave pública autorizada para SSH |

## Pruébalo tú

1. Documenta el VMID de la template.
2. Documenta el nombre real del storage y bridge.
3. Crea una VM de prueba con `proxmox-create-vm.yml`.
4. Entra por SSH sin contraseña.
5. Crea una segunda VM cambiando VMID, nombre e IP.
6. Ejecuta el flujo completo con Docker cuando la clonación esté controlada.

## Errores comunes

- **El playbook crea la VM pero no puedes entrar**: revisa `ciuser`, clave pública e IP.
- **El playbook falla antes de crear nada**: revisa token, permisos, nombre de nodo y nombre de template.
- **La segunda VM falla**: normalmente es VMID duplicado o IP ocupada.

> Idea clave: este checkpoint existe para confirmar que ya tienes una fábrica básica de VMs, no para aprobar un examen.
