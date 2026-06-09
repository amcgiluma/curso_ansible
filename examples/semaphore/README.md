# Semaphore UI para el homelab

Ejemplo para levantar Semaphore UI y convertir los playbooks del curso en tareas ejecutables desde navegador.

## Arranque

```bash
cp .env.example .env
# Edita .env antes de exponerlo en red
docker compose up -d
```

Abre `http://localhost:3000` con el usuario definido en `.env`.

## Orden de configuración

1. Proyecto: `homelab`.
2. Repository: repo Git donde estén tus playbooks.
3. Key Store: clave SSH para entrar en las VMs.
4. Inventory: hosts reales para pruebas iniciales.
5. Environment: variables de Proxmox y de la VM.
6. Task Template: primero `examples/ansible-homelab/playbooks/ping.yml`.
7. Task Template: después `examples/ansible-homelab/playbooks/install-docker.yml`.
8. Task Template: finalmente `examples/ansible-homelab/playbooks/provision-docker-vm.yml`.

Antes de lanzar el playbook de Proxmox desde Semaphore, hazlo funcionar desde CLI.

## Requirements de Ansible

El entorno que ejecuta Semaphore necesita las colecciones del proyecto:

```bash
ansible-galaxy collection install -r examples/ansible-homelab/requirements.yml
```

Para empezar puedes instalarlas dentro del contenedor. Para un uso estable, crea una imagen propia de Semaphore con Ansible y las colecciones ya instaladas.

## Seguridad

No subas `.env`, claves SSH, inventarios con secretos ni tokens de Proxmox. Este ejemplo usa SQLite para empezar rápido; para uso más serio, cambia a PostgreSQL o MySQL siguiendo la documentación oficial de Semaphore UI.
