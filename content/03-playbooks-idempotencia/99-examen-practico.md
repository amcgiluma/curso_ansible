---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Debes leer un playbook real, ejecutar tasks con criterio y demostrar que entiendes la sintaxis básica de Ansible desde cero.

## Teoría

Un playbook útil no es solo una lista de comandos: combina contexto de ejecución, tasks bien escritas, módulos adecuados e idempotencia.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --syntax-check
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --list-tasks
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --check --diff
# OUT
playbook: playbooks/task-syntax-lab.yml

playbook: playbooks/task-syntax-lab.yml

  play #1 (docker_hosts): Laboratorio de sintaxis de tasks   TAGS: []
    tasks:
      Asegurar directorio de laboratorio                     TAGS: [files]
      Crear fichero de ejemplo                               TAGS: [files]
      Crear directorios para cada item                       TAGS: [files, loop]
      Leer versión de Docker sin marcar cambio               TAGS: [command]
      Mostrar versión detectada                              TAGS: [command, debug]
      ...

PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
```

## Flags y variantes

| Flag | Para qué sirve |
| --- | --- |
| `--syntax-check` | Valida el playbook |
| `--list-tasks` | Enumera las tasks del playbook |
| `--check` | Simula |
| `--diff` | Enseña cambios |
| `--tags <tag>` | Ejecuta solo una parte del laboratorio |

## Pruébalo tú

1. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --syntax-check`.
2. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --list-tasks`.
3. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --tags files`.
4. Ejecuta el playbook completo dos veces y compara el `PLAY RECAP`.
5. Explica con tus palabras qué hacen `name`, el módulo, `register`, `when`, `loop`, `tags` y `changed_when` dentro de una task.

## Errores comunes

- **Describir una task por su sintaxis pero no por su intención**: si no entiendes el `name`, no entiendes la automatización.
- **Mirar solo si "funciona"**: el examen pide también entender por qué una task marca `ok` o `changed`.
- **Saltar `ansible-doc`**: si no conoces un módulo, primero se consulta; no se inventa.

> Idea clave: si puedes leer una task completa, elegir el built-in correcto y repetir el playbook sin cambios innecesarios, ya puedes empezar a escribir Ansible propio con criterio.
