---
title: "Conectar Semaphore con tus playbooks"
slug: "semaphore-ui"
order: 3
summary: "Levantar Semaphore y dejar plantillas listas para ejecutar Ansible desde navegador."
---

# Conectar Semaphore con tus playbooks

Semaphore UI es la capa cómoda para ejecutar tus playbooks desde navegador. No sustituye a aprender Ansible por terminal; lo que hace es convertir flujos repetibles en botones controlados, con logs, credenciales y permisos.

## Teoría

El orden correcto es importante. Primero haces que el playbook funcione desde CLI. Después lo llevas a Semaphore. Si intentas depurar Proxmox, SSH, Docker y Semaphore a la vez, no sabrás dónde está el fallo.

El mapa mental es este:

| Pieza | Responsabilidad |
| --- | --- |
| GitHub | Repositorio de playbooks |
| Semaphore | Interfaz, proyectos, plantillas, logs y ejecuciones |
| Repository | Copia del repo que Semaphore descarga para ejecutar |
| Key Store | Claves SSH, tokens y secretos cifrados |
| Inventory | Lista de hosts gestionados |
| Environment | Variables no secretas o JSON de configuración |
| Task Template | Playbook exacto que se ejecutará y con qué opciones |

Para el homelab empezamos con Docker Compose y SQLite. Es suficiente para una primera instalación interna. Más adelante puedes migrar a PostgreSQL si Semaphore se vuelve una pieza central.

## Manos a la obra

### 1. Levantar Semaphore

Entra en el ejemplo:

```compare
# CMD
cd examples/semaphore
cp .env.example .env
docker compose up -d
docker compose ps
# OUT
NAME        IMAGE                         SERVICE     STATUS
semaphore   semaphoreui/semaphore:latest  semaphore   Up ...
```

El `docker-compose.yml` crea un servicio:

```yaml
services:
  semaphore:
    image: semaphoreui/semaphore:latest
    container_name: semaphore
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      SEMAPHORE_DB_DIALECT: sqlite
      SEMAPHORE_DB: /etc/semaphore/semaphore.sqlite
      SEMAPHORE_PLAYBOOK_PATH: /tmp/semaphore
```

Qué significa:

| Campo | Qué hace |
| --- | --- |
| `ports: "3000:3000"` | Publica la UI en `http://localhost:3000` |
| `SEMAPHORE_DB_DIALECT: sqlite` | Usa una base de datos simple en archivo |
| `SEMAPHORE_DB` | Ruta donde se guarda la base de datos |
| `SEMAPHORE_PLAYBOOK_PATH` | Carpeta temporal donde Semaphore clona repos y ejecuta playbooks |
| `semaphore-config` | Volumen persistente para configuración y SQLite |
| `semaphore-tmp` | Volumen para ejecuciones temporales |

Revisa que responde:

```compare
# CMD
curl -I http://localhost:3000
# OUT
HTTP/1.1 200 OK
...
```

### 2. Entrar y crear el proyecto

Abre `http://localhost:3000` e inicia sesión con los valores de `.env`:

| Variable | Uso |
| --- | --- |
| `SEMAPHORE_ADMIN` | Usuario inicial |
| `SEMAPHORE_ADMIN_PASSWORD` | Contraseña inicial |
| `SEMAPHORE_ACCESS_KEY_ENCRYPTION` | Clave con la que Semaphore cifra secretos |

Cambia la contraseña y genera una clave de cifrado real antes de usarlo en serio.

Dentro de la UI:

1. Crea un proyecto llamado `homelab`.
2. Entra en el proyecto.
3. Crea los recursos en este orden: Repository, Key Store, Inventory, Environment y Task Template.

### 3. Añadir el repositorio

En `Repositories`, añade el repo donde está el curso o tu fork:

| Campo | Valor recomendado |
| --- | --- |
| Name | `curso-ansible` |
| URL | `https://github.com/amcgiluma/curso_ansible.git` |
| Branch | `main` |
| Access Key | Ninguna si el repo es público |

Semaphore descargará el repo antes de ejecutar una tarea. Por eso los playbooks deben estar versionados y los secretos no.

### 4. Añadir credenciales en Key Store

Necesitas como mínimo una clave SSH para conectar a tus VMs:

| Campo | Valor |
| --- | --- |
| Type | SSH Key |
| Name | `ssh-homelab` |
| Private key | Tu clave privada para entrar en las VMs |

Si vas a ejecutar playbooks contra Proxmox, también tendrás que guardar el token. Hay dos formas:

| Forma | Cuándo usarla |
| --- | --- |
| Environment cifrado en Semaphore | Cómodo para empezar |
| Ansible Vault | Mejor si quieres mantener secretos versionables cifrados |

Para empezar, usa Semaphore para guardar los secretos y evita subir `group_vars/all.yml` real a GitHub.

### 5. Crear inventario

Para el primer test usa un inventario simple. Puedes pegar algo parecido a:

```ini
[docker_hosts]
vm-docker-01 ansible_host=192.168.1.50 ansible_user=ubuntu ansible_become=true
```

Este inventario sirve para playbooks que gestionan VMs ya existentes, como `ping.yml` o `install-docker.yml`.

Para el playbook `provision-docker-vm.yml`, recuerda que la primera parte corre en `localhost` dentro del contenedor de Semaphore. Eso significa que el contenedor debe tener Ansible, colecciones y acceso de red a Proxmox.

### 6. Crear environment

Un environment en Semaphore permite pasar variables al playbook. Para valores no secretos puedes usar JSON:

```json
{
  "proxmox_api_host": "192.168.1.10",
  "proxmox_api_user": "root@pam",
  "proxmox_api_token_id": "ansible",
  "proxmox_node": "pve",
  "proxmox_storage": "local-lvm",
  "proxmox_bridge": "vmbr0",
  "proxmox_template": "ubuntu-2404-cloudinit-template",
  "proxmox_vm_id": 9101,
  "proxmox_vm_name": "vm-docker-01",
  "proxmox_vm_ip": "192.168.1.50",
  "proxmox_vm_ipconfig": "ip=192.168.1.50/24,gw=192.168.1.1"
}
```

El secreto `proxmox_api_token_secret` no debería quedar en texto claro dentro de un repo. Si lo pones en Semaphore, trátalo como dato sensible y limita quién puede ver o editar el proyecto.

### 7. Crear la primera Task Template: ping

Empieza por una plantilla que no cambie nada:

| Campo | Valor |
| --- | --- |
| Name | `Ping VMs` |
| Playbook | `examples/ansible-homelab/playbooks/ping.yml` |
| Inventory | Inventario del homelab |
| Repository | `curso-ansible` |
| Environment | El environment creado, si aplica |
| Vault / SSH Key | `ssh-homelab` |

Lanza la tarea y revisa logs. El objetivo es ver `pong` contra una VM existente.

### 8. Crear la plantilla de Docker

Cuando `ping.yml` funcione:

| Campo | Valor |
| --- | --- |
| Name | `Instalar Docker` |
| Playbook | `examples/ansible-homelab/playbooks/install-docker.yml` |
| Inventory | Inventario del homelab |
| SSH Key | `ssh-homelab` |

Esta tarea modifica la VM, así que úsala solo contra hosts que quieras gestionar.

### 9. Crear la plantilla de provisión Proxmox

La plantilla más potente será:

| Campo | Valor |
| --- | --- |
| Name | `Crear VM Docker en Proxmox` |
| Playbook | `examples/ansible-homelab/playbooks/provision-docker-vm.yml` |
| Inventory | Puede ser mínimo; el playbook usa `localhost` y `add_host` |
| Environment | Variables de Proxmox y VM |
| SSH Key | Clave para entrar en la VM creada |

Antes de lanzarla desde Semaphore, ejecútala una vez desde CLI. Semaphore debe ser la capa de operación, no el primer sitio donde pruebas algo destructivo.

### 10. Instalar requirements antes de ejecutar

Semaphore necesita que la colección `community.general` esté disponible en el entorno de ejecución. Tienes tres opciones:

| Opción | Ventaja |
| --- | --- |
| Instalar colecciones dentro del contenedor | Simple para laboratorio |
| Añadir una tarea previa en Semaphore | Repetible desde UI |
| Crear una imagen propia de Semaphore con Ansible preparado | Mejor para uso estable |

Para empezar puedes abrir shell en el contenedor:

```bash
docker exec -it semaphore sh
cd /tmp/semaphore/repository_*
ansible-galaxy collection install -r examples/ansible-homelab/requirements.yml
```

Más adelante crearás una imagen propia para no depender de instalaciones manuales.

## Flags y variantes

| Opción | Uso |
| --- | --- |
| `docker compose up -d` | Arranca Semaphore en segundo plano |
| `docker compose logs -f semaphore` | Sigue logs de la aplicación |
| `SEMAPHORE_ADMIN` | Usuario administrador inicial |
| `SEMAPHORE_ADMIN_PASSWORD` | Contraseña inicial |
| `SEMAPHORE_ACCESS_KEY_ENCRYPTION` | Clave usada para cifrar credenciales |
| Repository | Fuente Git de playbooks |
| Inventory | Hosts o grupos contra los que ejecuta Ansible |
| Environment | Variables entregadas al playbook |
| Task Template | Definición reutilizable de una ejecución |

## Pruébalo tú

1. Entra en `examples/semaphore`.
2. Copia `.env.example` a `.env`.
3. Cambia contraseña y `SEMAPHORE_ACCESS_KEY_ENCRYPTION`.
4. Ejecuta `docker compose up -d`.
5. Abre `http://localhost:3000`.
6. Crea el proyecto `homelab`.
7. Añade el repositorio del curso.
8. Añade la clave SSH en Key Store.
9. Crea un inventario con una VM existente.
10. Crea y lanza primero `ping.yml`.
11. Crea después `install-docker.yml`.
12. Deja `provision-docker-vm.yml` para cuando ya funcione desde CLI.

## Errores comunes

- **Probar primero el playbook de Proxmox**: empieza con `ping.yml` para separar problemas de Semaphore, SSH y API.
- **Olvidar colecciones**: instala `community.general` en el entorno donde Semaphore ejecuta Ansible.
- **Confundir localhost**: dentro de Semaphore, `localhost` es el contenedor, no tu PC ni Proxmox.
- **Guardar secretos en el repo**: Semaphore debe recibirlos por Key Store, environment protegido o Ansible Vault.

> Idea clave: Semaphore no arregla playbooks; los operacionaliza. Primero funciona en CLI, después lo conviertes en tarea reproducible desde navegador.
