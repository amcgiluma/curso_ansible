# Ejemplo: ansible-homelab

Proyecto Ansible para el objetivo real del curso: crear una VM en Proxmox desde una plantilla cloud-init, esperar a que responda por SSH e instalar Docker dentro de ella.

## Flujo recomendado

```bash
cp inventory.ini.example inventory.ini
cp group_vars/all.yml.example group_vars/all.yml
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/ping.yml
```

Antes de lanzar playbooks que cambian infraestructura:

1. Prepara una template cloud-init en Proxmox.
2. Crea un API token de Proxmox.
3. Edita `group_vars/all.yml` con tus datos reales.
4. Comprueba SSH contra una VM existente con `ping.yml`.
5. Ejecuta `proxmox-create-vm.yml` para probar solo la clonación.
6. Ejecuta `provision-docker-vm.yml` para crear la VM e instalar Docker.

## Playbooks

| Playbook | Uso |
| --- | --- |
| `playbooks/ping.yml` | Comprueba conectividad Ansible |
| `playbooks/install-docker.yml` | Instala Docker en `docker_hosts` |
| `playbooks/task-syntax-lab.yml` | Laboratorio para aprender la sintaxis de tasks, built-ins y ejecucion idempotente |
| `playbooks/proxmox-create-vm.yml` | Crea una VM cloud-init en Proxmox |
| `playbooks/provision-docker-vm.yml` | Crea VM, espera SSH e instala Docker |

## Variables importantes

| Variable | Qué controla |
| --- | --- |
| `proxmox_api_host` | IP o DNS del nodo Proxmox |
| `proxmox_api_user` | Usuario propietario del token |
| `proxmox_api_token_id` | Nombre corto del token |
| `proxmox_api_token_secret` | Secreto real del token |
| `proxmox_node` | Nodo donde se crea la VM |
| `proxmox_storage` | Storage principal para el disco |
| `proxmox_bridge` | Bridge de red, normalmente `vmbr0` |
| `proxmox_template` | Template cloud-init que se clonará |
| `proxmox_vm_ipconfig` | Red que cloud-init aplicará en la VM |

No guardes tokens reales en ficheros de ejemplo ni en commits. El archivo `group_vars/all.yml.example` es documentación ejecutable; tu `group_vars/all.yml` es configuración privada.
