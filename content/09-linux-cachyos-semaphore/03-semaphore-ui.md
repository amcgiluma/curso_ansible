---
title: "Ejecutar playbooks con Semaphore"
slug: "semaphore-ui"
order: 3
summary: "Levantar Semaphore UI con Docker Compose y conectarlo al repositorio de Ansible."
---

# Ejecutar playbooks con Semaphore

Semaphore UI te da una interfaz web para lanzar playbooks, guardar inventarios, definir credenciales y compartir ejecuciones entre tus máquinas.

## Teoría

No sustituye a Ansible: lo envuelve. La CLI sigue siendo la base para aprender y depurar; Semaphore sirve para ejecutar tareas repetibles sin entrar por terminal cada vez.

| Pieza | Responsabilidad |
| --- | --- |
| GitHub | Repositorio de playbooks |
| Semaphore | Interfaz, proyectos, plantillas y ejecuciones |
| Key Store | Credenciales SSH, tokens y secretos |
| Inventory | Hosts gestionados |
| Task Template | Qué playbook se ejecuta y con qué parámetros |

Para homelab, la instalación más resolutiva es Docker Compose con SQLite o base de datos externa. El ejemplo del curso usa SQLite para empezar rápido.

## Manos a la obra

Levanta el ejemplo incluido:

```compare
# CMD
cd examples/semaphore
cp .env.example .env
docker compose up -d
docker compose ps
# OUT
NAME                  IMAGE                         SERVICE     STATUS
semaphore-semaphore   semaphoreui/semaphore:latest  semaphore   Up ...
```

Abre la interfaz:

```compare
# CMD
curl -I http://localhost:3000
# OUT
HTTP/1.1 200 OK
...
```

Después crea en la UI:

| Sección | Valor inicial |
| --- | --- |
| Project | `homelab` |
| Repository | `https://github.com/amcgiluma/curso_ansible.git` |
| Inventory | Tu inventario real o uno pegado desde `inventory.ini` |
| Environment | Variables no secretas |
| Key Store | Clave SSH y tokens |
| Task Template | `examples/ansible-homelab/playbooks/ping.yml` |

## Flags y variantes

| Opción | Uso |
| --- | --- |
| `docker compose up -d` | Arranca Semaphore en segundo plano |
| `docker compose logs -f semaphore` | Sigue logs de la aplicación |
| `SEMAPHORE_ADMIN` | Usuario administrador inicial |
| `SEMAPHORE_ADMIN_PASSWORD` | Contraseña inicial |
| `SEMAPHORE_ACCESS_KEY_ENCRYPTION` | Clave usada para cifrar credenciales |

## Pruébalo tú

1. Entra en `examples/semaphore`.
2. Copia `.env.example` a `.env`.
3. Cambia contraseña y `SEMAPHORE_ACCESS_KEY_ENCRYPTION`.
4. Ejecuta `docker compose up -d`.
5. Abre `http://localhost:3000`.
6. Crea un proyecto conectado al repo de GitHub.
7. Lanza primero `playbooks/ping.yml`, no el playbook de Proxmox.

## Errores comunes

- **Empezar por el playbook destructivo**: valida primero `ping.yml` y `--list-hosts`.
- **Poner secretos en GitHub**: usa Key Store o `.env` local.
- **No instalar colecciones**: añade un template o tarea previa para `ansible-galaxy collection install -r requirements.yml`.
- **Usar `localhost` desde el contenedor pensando en tu host**: dentro del contenedor, `localhost` es el propio contenedor.

> Idea clave: Semaphore convierte tus playbooks versionados en tareas ejecutables desde navegador, pero la seguridad depende de separar repo, inventario y secretos.
