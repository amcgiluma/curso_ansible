---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Debes preparar variables de Proxmox y crear una VM desde plantilla cloud-init.

## Teoría

El éxito del módulo se mide por una VM creada sin entrar al panel de Proxmox para rellenar campos a mano.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/proxmox-create-vm.yml
# OUT
PLAY RECAP *******************************************************************
localhost : ok=<n> changed=1 unreachable=0 failed=0
```

## Flags y variantes

| Variable | Para qué sirve |
| --- | --- |
| `proxmox_node` | Nodo donde crear la VM |
| `proxmox_template` | Plantilla cloud-init |
| `proxmox_vm_id` | ID de VM |
| `proxmox_vm_name` | Nombre de VM |
| `proxmox_vm_ipconfig` | Red cloud-init |

## Pruébalo tú

1. Instala la colección.
2. Configura token API.
3. Configura plantilla y VMID.
4. Ejecuta el playbook.
5. Verifica la VM en Proxmox.

## Errores comunes

- **Permisos insuficientes del token**: ajusta permisos en Proxmox.
- **Certificado self-signed**: define conscientemente `validate_certs: false` solo en homelab si lo aceptas.

> Idea clave: si Ansible crea la VM, ya tienes la parte de infraestructura automatizada.
