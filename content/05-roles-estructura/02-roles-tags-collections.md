---
title: "Roles, tags y colecciones"
slug: "roles-tags-collections"
order: 2
summary: "Reutilizar tareas, ejecutar partes concretas e instalar módulos externos."
---

# Roles, tags y colecciones

Los roles empaquetan tareas. Los tags te permiten ejecutar partes. Las colecciones aportan módulos externos como los de Proxmox.

## Teoría

Un rol mínimo contiene:

```output
roles/docker/
├── tasks/main.yml
└── handlers/main.yml
```

Las colecciones se declaran en `requirements.yml`:

```yaml
collections:
  - name: community.general
```

Para Proxmox usaremos módulos de `community.general`, por eso conviene fijar este paso antes del proyecto final.

## Manos a la obra

Instala colecciones y lista tareas por tag:

```compare
# CMD
cd examples/ansible-homelab
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/install-docker.yml --list-tags
# OUT
Starting galaxy collection install process
...

playbook: playbooks/install-docker.yml

  play #1 (docker_hosts): Instalar Docker en hosts Linux
      TASK TAGS: [docker]
```

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `ansible-galaxy collection install -r requirements.yml` | Instala colecciones declaradas |
| `--list-tags` | Lista tags disponibles |
| `--tags docker` | Ejecuta solo tareas con ese tag |
| `--skip-tags docker` | Omite tareas con ese tag |
| `roles:` | Aplica roles en un play |

## Pruébalo tú

1. Ejecuta la instalación de colecciones.
2. Lista tags de `install-docker.yml`.
3. Ejecuta el playbook con `--tags docker --check`.
4. Abre `roles/docker/tasks/main.yml` y localiza el tag.
5. Explica por qué Proxmox necesita una colección externa.

## Errores comunes

- **Olvidar instalar colecciones**: el playbook fallará con módulos no encontrados.
- **Usar tags como sustituto de buena estructura**: tags ayudan, pero no arreglan roles mal diseñados.
- **No versionar `requirements.yml`**: otra persona no sabrá qué instalar.

> Idea clave: roles ordenan la lógica, tags controlan ejecución y colecciones amplían Ansible sin meter scripts propios.
