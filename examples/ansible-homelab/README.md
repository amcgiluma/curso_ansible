# Ejemplo: ansible-homelab

Proyecto Ansible mínimo para practicar el curso: probar conectividad, instalar Docker en VMs Ubuntu/Debian y crear una VM en Proxmox desde una plantilla cloud-init.

## Uso rápido

```bash
cp inventory.ini.example inventory.ini
cp group_vars/all.yml.example group_vars/all.yml
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/ping.yml
```

Edita `inventory.ini` y `group_vars/all.yml` antes de ejecutar playbooks reales.

## Playbooks

| Playbook | Uso |
| --- | --- |
| `playbooks/ping.yml` | Comprueba conectividad Ansible |
| `playbooks/install-docker.yml` | Instala Docker en `docker_hosts` |
| `playbooks/proxmox-create-vm.yml` | Crea una VM cloud-init en Proxmox |
| `playbooks/provision-docker-vm.yml` | Crea VM, espera SSH e instala Docker |

No guardes tokens reales en ficheros de ejemplo ni en commits.
