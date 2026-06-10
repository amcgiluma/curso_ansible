---
title: "Anatomía de una task"
slug: "anatomia-task"
order: 2
summary: "Aprender qué significa cada parte de una task y cómo se escribe bien en YAML."
---

# Anatomía de una task

La unidad real de trabajo en Ansible es la task. Si entiendes la sintaxis de una task, puedes empezar a automatizar casi cualquier cosa con criterio.

## Teoría

Esta task tiene casi todas las piezas que verás en automatización real:

```yaml
- name: Instalar Docker Engine y Compose plugin
  ansible.builtin.apt:
    name:
      - docker.io
      - docker-compose-v2
    state: present
    update_cache: true
  register: docker_install
  when: ansible_os_family == "Debian"
  notify: Reiniciar Docker
  tags: ["docker", "packages"]
```

Qué significa cada parte:

| Parte | Qué hace |
| --- | --- |
| `name` | Describe la intención de la task |
| `ansible.builtin.apt` | Módulo que se ejecuta |
| `name`, `state`, `update_cache` | Argumentos del módulo |
| `register` | Guarda el resultado en una variable |
| `when` | Ejecuta la task solo si se cumple una condición |
| `notify` | Lanza un handler si la task cambia algo |
| `tags` | Permite seleccionar o saltar tasks por categoría |

Otras claves muy comunes:

| Clave | Uso habitual |
| --- | --- |
| `loop` | Repetir la task para varios valores |
| `become` | Elevar privilegios solo en esa task |
| `changed_when` | Reescribir la lógica de cambio |
| `failed_when` | Reescribir la lógica de error |
| `ignore_errors` | Continuar aunque falle la task |
| `check_mode` | Forzar o evitar check mode en esa task |
| `delegate_to` | Ejecutar la task en otro host |

Reglas prácticas de sintaxis:

- Usa siempre `name` salvo pruebas muy rápidas.
- Prefiere FQCN como `ansible.builtin.copy` frente a `copy`.
- Los argumentos del módulo van indentados debajo del módulo, no al mismo nivel que `register` o `when`.
- `register`, `when`, `loop`, `notify` y `tags` van al nivel de la task, no dentro del módulo.

## Manos a la obra

Ejecuta el laboratorio de sintaxis y observa cómo cambian distintas tasks:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --syntax-check
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --limit vm-docker-01
# OUT
playbook: playbooks/task-syntax-lab.yml

PLAY [Laboratorio de sintaxis de tasks] **************************************

TASK [Asegurar directorio de laboratorio] ************************************
changed: [vm-docker-01]

TASK [Crear fichero de ejemplo] **********************************************
changed: [vm-docker-01]

TASK [Leer versión de Docker sin marcar cambio] *******************************
ok: [vm-docker-01]

TASK [Mostrar versión detectada] *********************************************
ok: [vm-docker-01] => {
    "docker_version.stdout": "Docker version <...>"
}

PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
```

## Flags y variantes

| Flag | Para qué sirve |
| --- | --- |
| `--start-at-task "<name>"` | Retoma desde una task concreta |
| `--step` | Pide confirmación antes de cada task |
| `--tags <tag>` | Ejecuta solo las tasks etiquetadas |
| `--skip-tags <tag>` | Omite tareas por tag |
| `--check` | Simula lo que pueda simular cada task |
| `--diff` | Muestra diferencias en tareas sobre ficheros |

## Pruébalo tú

1. Entra en `examples/ansible-homelab`.
2. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --tags files`.
3. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --start-at-task "Crear fichero de ejemplo"`.
4. Localiza en el playbook qué líneas pertenecen al módulo y qué líneas pertenecen a la task.
5. Cambia temporalmente el `name` de una task por algo poco claro y observa la diferencia en la salida.

## Errores comunes

- **Meter `when` o `register` dentro del módulo**: Ansible lo interpretará mal o dará error.
- **Usar `shell` cuando un módulo idempotente ya existe**: pierdes control del estado.
- **No usar FQCN**: funciona muchas veces, pero leer `ansible.builtin.copy` deja claro de dónde viene el módulo.

> Idea clave: una task combina intención, módulo y control de ejecución; si sabes separar esas tres capas, ya sabes escribir Ansible con orden.
