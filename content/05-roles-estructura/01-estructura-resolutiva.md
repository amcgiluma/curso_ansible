---
title: "Estructura resolutiva de proyecto"
slug: "estructura-resolutiva"
order: 1
summary: "Ordenar inventario, playbooks, variables y roles sin sobrediseñar."
---

# Estructura resolutiva de proyecto

Ansible escala mejor cuando separas intención, datos y reutilización. No necesitas una arquitectura enorme; sí una estructura estable.

## Teoría

La estructura del ejemplo será la base del curso:

```output
examples/ansible-homelab/
├── ansible.cfg
├── inventory.ini.example
├── group_vars/
├── requirements.yml
├── playbooks/
└── roles/
```

Regla práctica:

| Carpeta | Responsabilidad |
| --- | --- |
| `playbooks/` | Casos de uso ejecutables |
| `roles/` | Lógica reutilizable |
| `group_vars/` | Datos comunes |
| `requirements.yml` | Colecciones necesarias |
| `ansible.cfg` | Configuración local del proyecto |

## Manos a la obra

Lista la estructura:

```compare
# CMD
cd examples/ansible-homelab
find . -maxdepth 3 -type f | sort
# OUT
./ansible.cfg
./group_vars/all.yml.example
./inventory.ini.example
./playbooks/install-docker.yml
./playbooks/ping.yml
./playbooks/provision-docker-vm.yml
./playbooks/proxmox-create-vm.yml
./requirements.yml
./roles/docker/handlers/main.yml
./roles/docker/tasks/main.yml
...
```

## Flags y variantes

| Fichero / opción | Para qué sirve |
| --- | --- |
| `ansible.cfg` | Define inventario por defecto y opciones SSH |
| `roles_path` | Dónde buscar roles |
| `collections_paths` | Dónde buscar colecciones |
| `retry_files_enabled` | Evita ficheros `.retry` antiguos |
| `host_key_checking` | Controla verificación de hosts SSH |

## Pruébalo tú

1. Entra en `examples/ansible-homelab`.
2. Copia los `.example` necesarios.
3. Ejecuta `ansible --version` dentro de la carpeta y mira qué `config file` detecta.
4. Ejecuta `ansible-playbook playbooks/ping.yml --list-hosts`.
5. Comprueba que no necesitas escribir `-i inventory.ini` si `ansible.cfg` apunta al inventario.

## Errores comunes

- **Playbooks enormes sin roles**: se vuelven difíciles de mantener.
- **Roles que hacen demasiadas cosas**: un rol debe tener una responsabilidad clara.
- **Config global que sorprende**: usa `ansible.cfg` local del proyecto.

> Idea clave: una estructura simple y constante evita que Ansible se convierta en una carpeta de scripts inconexos.
