---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Debes ejecutar un playbook, entender su recap y comprobar que repetirlo no produce cambios innecesarios.

## Teoría

Un playbook útil no es solo un conjunto de comandos: debe poder repetirse y dejar el sistema en el estado esperado.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/ping.yml --syntax-check
ansible-playbook -i inventory.ini playbooks/ping.yml
# OUT
playbook: playbooks/ping.yml

PLAY RECAP *******************************************************************
vm-docker-01 : ok=1 changed=0 unreachable=0 failed=0
```

## Flags y variantes

| Flag | Para qué sirve |
| --- | --- |
| `--syntax-check` | Valida el playbook |
| `--check` | Simula |
| `--diff` | Enseña cambios |
| `--limit` | Acota hosts |

## Pruébalo tú

1. Ejecuta syntax check.
2. Ejecuta `ping.yml`.
3. Ejecuta `install-docker.yml --check --diff`.
4. Explica la diferencia entre `ok`, `changed`, `failed` y `unreachable`.

## Errores comunes

- **Confundir `failed` con `unreachable`**: `unreachable` es conexión; `failed` es una tarea que llegó a ejecutarse.
- **No leer el recap**: es el resumen mínimo para saber qué pasó.

> Idea clave: si entiendes el recap y puedes repetir playbooks, ya estás trabajando de forma resolutiva.
