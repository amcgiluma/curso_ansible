---
title: "Templates y handlers"
slug: "templates-handlers"
order: 2
summary: "Generar configuración con Jinja2 y reiniciar servicios solo si cambia algo."
---

# Templates y handlers

Cuando gestionas servicios, no quieres copiar ficheros a mano ni reiniciar siempre. Templates y handlers resuelven eso.

## Teoría

Un template Jinja2 mezcla texto fijo con variables:

```jinja2
server_name = {{ inventory_hostname }}
docker_log_driver = {{ docker_log_driver }}
```

Un handler es una tarea especial que se ejecuta solo si una tarea lo notifica con `notify`. Esto mantiene idempotencia: si el fichero no cambia, el servicio no se reinicia.

## Manos a la obra

Inspecciona los handlers del rol Docker:

```compare
# CMD
cd examples/ansible-homelab
find roles/docker -maxdepth 2 -type f -print
ansible-playbook -i inventory.ini playbooks/install-docker.yml --list-tasks
# OUT
roles/docker/tasks/main.yml
roles/docker/handlers/main.yml

playbook: playbooks/install-docker.yml

  play #1 (docker_hosts): Instalar Docker en hosts Linux
    tasks:
      docker : Instalar paquetes base
      docker : Instalar Docker Engine
      ...
```

## Flags y variantes

| Recurso | Para qué sirve |
| --- | --- |
| `ansible.builtin.template` | Renderiza un fichero desde Jinja2 |
| `notify` | Avisa a un handler |
| `handlers:` | Define tareas diferidas |
| `--list-tasks` | Lista tareas sin ejecutarlas |
| `--tags` | Ejecuta solo tareas con ciertos tags |

## Pruébalo tú

1. Ejecuta `--list-tasks` sobre `install-docker.yml`.
2. Abre `roles/docker/handlers/main.yml`.
3. Añade mentalmente cuándo tendría sentido reiniciar Docker.
4. Ejecuta el playbook dos veces y observa cuándo hay cambios.
5. Si modificas configuración gestionada, comprueba que el handler se dispara.

## Errores comunes

- **Reiniciar servicios con una tarea normal siempre**: rompe la idempotencia operativa.
- **Usar `copy` para ficheros variables**: usa `template`.
- **Notificar handlers con nombres mal escritos**: el nombre debe coincidir.

> Idea clave: templates gestionan ficheros variables; handlers aplican cambios solo cuando esos ficheros cambian.
