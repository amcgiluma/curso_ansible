---
title: "Anatomía de un playbook"
slug: "anatomia-playbook"
order: 1
summary: "Entender desde cero qué partes tiene un playbook y qué significa cada clave principal."
---

# Anatomía de un playbook

Un playbook es el fichero donde describes a qué hosts apuntas, con qué privilegios trabajas y qué tasks quieres ejecutar. En esta lección vas a aprender a leerlo entero, no solo a copiarlo.

## Teoría

Un playbook es una lista de `plays`. Cada `play` define un contexto de ejecución:

```yaml
---
- name: Comprobar conectividad Ansible
  hosts: docker_hosts
  gather_facts: false

  tasks:
    - name: Ping Ansible
      ansible.builtin.ping:
```

Las claves principales de un `play` son estas:

| Clave | Qué significa |
| --- | --- |
| `name` | Nombre legible del play en la salida |
| `hosts` | Grupo u host del inventario al que se aplica |
| `become` | Si Ansible debe usar privilegios elevados |
| `gather_facts` | Si recoge facts del host antes de ejecutar tasks |
| `vars` | Variables definidas para ese play |
| `tasks` | Lista ordenada de tareas |
| `handlers` | Tareas especiales que se disparan con `notify` |
| `roles` | Roles que encapsulan tasks, handlers y defaults |
| `pre_tasks` | Tasks que corren antes de roles o tasks principales |
| `post_tasks` | Tasks que corren al final del play |

No todo lo que ves a la izquierda en YAML pertenece a una task. `hosts`, `gather_facts` o `roles` son claves del `play`; `register`, `when` o `loop` son claves de una `task`.

> Una buena forma de leer un playbook es de arriba abajo en tres preguntas: a quién afecta, con qué contexto se ejecuta y qué tareas dispara.

## Manos a la obra

Primero valida y luego ejecuta el playbook mínimo del ejemplo:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/ping.yml --syntax-check
ansible-playbook -i inventory.ini playbooks/ping.yml
# OUT
playbook: playbooks/ping.yml

PLAY [Comprobar conectividad Ansible] ****************************************

TASK [Ping Ansible] **********************************************************
ok: [vm-docker-01]

PLAY RECAP *******************************************************************
vm-docker-01 : ok=1 changed=0 unreachable=0 failed=0
```

Ahora abre `playbooks/install-docker.yml` y localiza estas partes:

- Un `play` con `hosts`, `become` y `gather_facts`.
- Un bloque `roles`.
- Un role `docker` que luego baja a `roles/docker/tasks/main.yml`.

## Flags y variantes

| Flag | Para qué sirve |
| --- | --- |
| `--syntax-check` | Valida la sintaxis del playbook sin ejecutarlo |
| `--list-hosts` | Muestra qué hosts entran en el `play` |
| `--limit <host>` | Acota la ejecución a un host o grupo |
| `-b` | Activa `become` desde CLI |
| `-K` | Pide la password de `become` si hace falta |
| `-e clave=valor` | Inyecta variables por línea de comandos |

## Pruébalo tú

1. Entra en `examples/ansible-homelab`.
2. Ejecuta `ansible-playbook -i inventory.ini playbooks/ping.yml --list-hosts`.
3. Ejecuta `ansible-playbook -i inventory.ini playbooks/ping.yml --syntax-check`.
4. Abre `playbooks/install-docker.yml` y anota qué claves son de `play`.
5. Abre `roles/docker/tasks/main.yml` y anota qué claves son de `task`.

## Errores comunes

- **Confundir un `play` con una `task`**: un play agrupa contexto; una task ejecuta una acción concreta.
- **Pensar que `hosts` es opcional**: sin `hosts`, el play no sabe dónde correr.
- **Dejar `gather_facts: true` por defecto sin necesitarlo**: puede ralentizar comprobaciones simples como `ping`.

> Idea clave: antes de escribir tasks propias, tienes que distinguir claramente el nivel `play` del nivel `task`.
