---
title: "Inventario básico"
slug: "inventario-basico"
order: 1
summary: "Definir hosts y grupos para que Ansible sepa a qué máquinas conectarse."
---

# Inventario básico

El inventario es la lista de máquinas que Ansible puede gestionar. Puede ser INI, YAML o dinámico; para empezar, INI es suficiente y fácil de leer.

## Teoría

Un inventario agrupa hosts por función. En homelab, grupos como `docker_hosts`, `monitoring` o `proxmox_vms` te permiten ejecutar playbooks sobre varias máquinas sin repetir IPs.

Ejemplo mínimo:

```ini
[docker_hosts]
vm-docker-01 ansible_host=192.168.1.50 ansible_user=ubuntu
```

Campos útiles:

| Variable | Uso |
| --- | --- |
| `ansible_host` | IP o DNS real del host |
| `ansible_user` | Usuario SSH |
| `ansible_ssh_private_key_file` | Clave privada concreta |
| `ansible_become` | Activar privilegios con sudo |

## Manos a la obra

Crea un inventario y lista sus hosts:

```compare
# CMD
mkdir -p examples/ansible-homelab
cd examples/ansible-homelab
printf "[docker_hosts]\nvm-docker-01 ansible_host=192.168.1.50 ansible_user=ubuntu\n" > inventory.ini
ansible-inventory -i inventory.ini --graph
# OUT
@all:
  |--@ungrouped:
  |--@docker_hosts:
  |  |--vm-docker-01
```

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `-i inventory.ini` | Indica el inventario |
| `ansible-inventory --graph` | Muestra grupos y hosts |
| `ansible-inventory --list` | Devuelve inventario completo en JSON |
| `ansible_host` | IP o DNS usado para conectar |
| `ansible_user` | Usuario remoto |

## Pruébalo tú

1. Entra en `examples/ansible-homelab`.
2. Copia `inventory.ini.example` como `inventory.ini`.
3. Cambia IP y usuario por los de tu VM.
4. Ejecuta `ansible-inventory -i inventory.ini --graph`.
5. Añade un segundo host ficticio y observa cómo aparece en el grafo.

## Errores comunes

- **Confundir alias con DNS**: `vm-docker-01` es el nombre interno de Ansible; la IP real está en `ansible_host`.
- **Inventario en otra carpeta**: usa `-i` o configura `ansible.cfg`.
- **Variables mal escritas**: `ansible_user`, no `ansible_username`.

> Idea clave: el inventario traduce tu homelab a grupos operables; después los playbooks apuntan a grupos, no a IPs sueltas.
