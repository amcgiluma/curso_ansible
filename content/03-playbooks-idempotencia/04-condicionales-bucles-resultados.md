---
title: "Condicionales, bucles y resultados"
slug: "condicionales-bucles-resultados"
order: 4
summary: "Usar when, loop, register y retries para que una task tome decisiones con datos reales."
---

# Condicionales, bucles y resultados

Las tasks no viven aisladas: suelen depender de datos previos, repetirse para varios elementos o reaccionar a resultados de comandos. En esta lección vas a unir esas piezas.

## Teoría

Los patrones más comunes son estos:

```yaml
- name: Leer version de Docker
  ansible.builtin.command: docker --version
  register: docker_version
  changed_when: false

- name: Mostrar version si existe salida
  ansible.builtin.debug:
    var: docker_version.stdout
  when: docker_version.stdout is defined

- name: Crear varios directorios
  ansible.builtin.file:
    path: "/tmp/{{ item }}"
    state: directory
    mode: "0755"
  loop:
    - ansible-demo-a
    - ansible-demo-b
```

Conceptos clave:

| Clave | Para qué sirve |
| --- | --- |
| `register` | Guarda el resultado completo de una task |
| `when` | Filtra si una task debe ejecutarse |
| `loop` | Repite la task por cada `item` |
| `until` | Reintenta hasta que se cumpla una condición |
| `retries` | Número máximo de reintentos |
| `delay` | Espera entre reintentos |

Un `register` no guarda solo `stdout`. También suele incluir `rc`, `stderr`, `stdout_lines`, `changed`, `failed` y más campos según el módulo.

> Cuando una automatización empieza a decidir cosas por sí sola, casi siempre aparecen `register`, `when` y `loop`.

## Manos a la obra

Ejecuta el laboratorio completo y fíjate en las tasks que usan `loop`, `register`, `when` y `until`:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml
# OUT
PLAY [Laboratorio de sintaxis de tasks] **************************************

TASK [Crear directorios para cada item] **************************************
changed: [vm-docker-01] => (item=ansible-loop-a)
changed: [vm-docker-01] => (item=ansible-loop-b)

TASK [Leer version de Docker sin marcar cambio] *******************************
ok: [vm-docker-01]

TASK [Mostrar version detectada] *********************************************
ok: [vm-docker-01] => {
    "docker_version.stdout": "Docker version <...>"
}

TASK [Esperar a que exista el fichero de laboratorio] *************************
ok: [vm-docker-01]

PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
```

## Flags y variantes

| Clave o flag | Para qué sirve |
| --- | --- |
| `register` | Guarda el resultado de una task |
| `when` | Ejecuta la task solo si la expresión es verdadera |
| `loop` | Itera sobre una lista |
| `until` | Reintenta hasta cumplir una expresión |
| `retries` | Número de intentos para `until` |
| `delay` | Segundos entre intentos |

## Pruébalo tú

1. Ejecuta `ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --list-tasks`.
2. Ejecuta el playbook y busca una task con `loop`.
3. Abre el playbook y localiza el `register: docker_version`.
4. Cambia temporalmente la condición `when` para forzar que una task se salte.
5. Vuelve a dejar el playbook como estaba.

## Errores comunes

- **Pensar que `register` guarda solo texto**: guarda una estructura completa, no solo `stdout`.
- **Escribir `{{ }}` dentro de `when`**: normalmente no hace falta; `when` evalúa expresiones directamente.
- **Usar `loop` sin mirar el valor de `item`**: luego no entiendes qué elemento está fallando.

> Idea clave: `register`, `when` y `loop` convierten una lista lineal de tasks en una automatización que observa y decide.
