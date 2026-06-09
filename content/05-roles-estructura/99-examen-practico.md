---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Debes preparar el proyecto de ejemplo para ejecutarse sin flags repetitivos y con colecciones instaladas.

## Teoría

Una estructura resolutiva reduce errores: sabes dónde están datos, playbooks y roles.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
cp inventory.ini.example inventory.ini
cp group_vars/all.yml.example group_vars/all.yml
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/ping.yml --list-hosts
# OUT
  hosts (1):
    vm-docker-01
```

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `cp *.example` | Crea ficheros locales editables |
| `--list-hosts` | Comprueba alcance |
| `--list-tags` | Comprueba tags |
| `ansible-galaxy` | Instala dependencias Ansible |

## Pruébalo tú

1. Copia los ficheros `.example`.
2. Ajusta IP y usuario.
3. Instala colecciones.
4. Lista hosts y tags.
5. Ejecuta `ping.yml`.

## Errores comunes

- **Editar `.example` directamente**: perderás la plantilla limpia.
- **No revisar `--list-hosts` antes de un playbook destructivo**: acostúmbrate a validar alcance.

> Idea clave: cuando el proyecto está ordenado, ejecutar Ansible deja de ser improvisación.
