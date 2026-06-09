---
title: "Idempotencia, check mode y diff"
slug: "idempotencia-check-diff"
order: 2
summary: "Entender por qué Ansible puede ejecutarse varias veces sin romper el sistema."
---

# Idempotencia, check mode y diff

La promesa práctica de Ansible es que puedes ejecutar un playbook varias veces y converger al mismo estado. Eso se llama idempotencia.

## Teoría

Una tarea idempotente describe el estado deseado:

| Imperativo | Declarativo/idempotente |
| --- | --- |
| "Ejecuta `apt install nginx`" | "El paquete `nginx` debe estar presente" |
| "Añade esta línea con `echo >>`" | "Este fichero debe contener esta línea" |
| "Reinicia siempre" | "Reinicia solo si cambió la config" |

Ansible informa `changed=true` cuando modificó algo. En una segunda ejecución correcta, muchas tareas deberían volver como `ok`.

## Manos a la obra

Ejecuta un playbook primero en simulación:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/install-docker.yml --check --diff
# OUT
PLAY [Instalar Docker en hosts Linux] ****************************************
...
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
```

Después ejecútalo de verdad y repítelo:

```compare
# CMD
ansible-playbook -i inventory.ini playbooks/install-docker.yml
ansible-playbook -i inventory.ini playbooks/install-docker.yml
# OUT
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
...
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=0 unreachable=0 failed=0
```

## Flags y variantes

| Flag | Para qué sirve |
| --- | --- |
| `--check` | Simula cambios cuando el módulo lo soporta |
| `--diff` | Muestra diferencias de ficheros gestionados |
| `--start-at-task` | Reanuda desde una tarea concreta |
| `--step` | Pregunta antes de cada tarea |
| `changed_when` | Ajusta cuándo una tarea se considera cambiada |
| `failed_when` | Ajusta cuándo una tarea se considera fallida |

## Pruébalo tú

1. Ejecuta `playbooks/install-docker.yml` con `--check --diff`.
2. Ejecuta el playbook real.
3. Ejecútalo otra vez y busca `changed=0` o cambios mínimos justificados.
4. Modifica un fichero gestionado y observa si Ansible lo corrige.
5. Anota qué tareas no pueden simularse bien en check mode.

## Errores comunes

- **Usar `shell` para instalar paquetes**: pierde idempotencia; usa `apt`.
- **Pensar que `--check` siempre predice todo**: algunos módulos no pueden simular al 100%.
- **Ignorar `changed`**: si siempre cambia, algo está mal diseñado.

> Idea clave: idempotencia significa poder repetir automatización con seguridad; `--check` y `--diff` te ayudan a revisar antes de tocar sistemas.
