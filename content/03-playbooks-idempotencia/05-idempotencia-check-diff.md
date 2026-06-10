---
title: "Idempotencia, check mode y diff"
slug: "idempotencia-check-diff"
order: 5
summary: "Entender por qué Ansible puede ejecutarse varias veces sin romper el sistema y cómo verificarlo."
---

# Idempotencia, check mode y diff

La promesa práctica de Ansible es que puedes ejecutar un playbook varias veces y converger al mismo estado. Eso se llama idempotencia, y es lo que convierte una lista de comandos en automatización confiable.

## Teoría

Una task idempotente describe estado deseado, no una acción ciega:

| Imperativo | Declarativo e idempotente |
| --- | --- |
| "Ejecuta `apt install nginx`" | "El paquete `nginx` debe estar presente" |
| "Haz `echo >> fichero`" | "Este fichero debe contener esta línea" |
| "Reinicia siempre Docker" | "Reinicia solo si cambió la configuración" |

En la salida de Ansible:

| Campo | Qué indica |
| --- | --- |
| `ok` | La task ya estaba en el estado correcto |
| `changed` | La task hizo un cambio real |
| `failed` | La task se ejecutó pero falló |
| `unreachable` | Ansible ni siquiera pudo entrar al host |

Cuando un módulo no sabe reflejar bien si hubo cambio, puedes ajustar el comportamiento con `changed_when` y `failed_when`.

## Manos a la obra

Ejecuta primero el laboratorio en simulación y luego repítelo en real:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --check --diff
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml
# OUT
PLAY [Laboratorio de sintaxis de tasks] **************************************
...
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
...
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
...
PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=0 unreachable=0 failed=0
```

También revisa `roles/docker/tasks/main.yml` y localiza esta decisión:

```yaml
- name: Comprobar version de Docker
  ansible.builtin.command: docker --version
  register: docker_version
  changed_when: false
```

Sin `changed_when: false`, esa comprobación marcaría cambio cada vez aunque solo esté leyendo información.

## Flags y variantes

| Flag o clave | Para qué sirve |
| --- | --- |
| `--check` | Simula cambios cuando el módulo lo soporta |
| `--diff` | Muestra diferencias en ficheros gestionados |
| `--step` | Pide confirmación antes de cada task |
| `changed_when` | Ajusta cuándo una task cuenta como cambio |
| `failed_when` | Ajusta cuándo una task cuenta como fallo |
| `--start-at-task` | Reanuda desde una task concreta |

## Pruébalo tú

1. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --check --diff`.
2. Ejecuta el playbook en real dos veces seguidas.
3. Busca una task que solo deba dar `ok` en la segunda pasada.
4. Abre `roles/docker/tasks/main.yml` y explica por qué `changed_when: false` tiene sentido.
5. Piensa qué tasks tuyas futuras deberían usar `notify` en vez de reiniciar siempre.

## Errores comunes

- **Usar `shell` para tareas que tienen built-in**: suele degradar la idempotencia.
- **Pensar que `--check` predice todo**: algunos módulos no pueden simular el 100% del cambio.
- **Aceptar `changed` constante como normal**: si siempre cambia, probablemente tu task está mal modelada.

> Idea clave: automatizar bien no es solo ejecutar; es poder repetir con seguridad y entender exactamente cuándo Ansible cambia algo.
