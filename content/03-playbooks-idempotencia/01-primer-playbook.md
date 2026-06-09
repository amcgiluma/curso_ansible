---
title: "Primer playbook"
slug: "primer-playbook"
order: 1
summary: "Escribir y ejecutar un playbook mínimo con tareas ordenadas."
---

# Primer playbook

Un playbook es una receta declarativa: define hosts, privilegios y tareas. Es el formato principal que usarás para preparar VMs.

## Teoría

Un playbook se compone de plays. Cada play apunta a un grupo de hosts y ejecuta tareas con módulos.

```yaml
- name: Comprobar hosts Docker
  hosts: docker_hosts
  tasks:
    - name: Mostrar hostname
      ansible.builtin.command: hostname
```

Las tareas deben tener nombres claros. Cuando algo falle, ese nombre será tu pista principal.

## Manos a la obra

Ejecuta el playbook de ping del ejemplo:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook -i inventory.ini playbooks/ping.yml
# OUT
PLAY [Comprobar conectividad Ansible] ****************************************

TASK [Ping Ansible] **********************************************************
ok: [vm-docker-01]

PLAY RECAP *******************************************************************
vm-docker-01 : ok=1 changed=0 unreachable=0 failed=0
```

## Flags y variantes

| Flag | Para qué sirve |
| --- | --- |
| `-i inventory.ini` | Inventario a usar |
| `--limit <host>` | Ejecuta solo sobre parte del inventario |
| `-b` | Activa privilegios sudo si el playbook no los define |
| `--list-hosts` | Muestra hosts afectados sin ejecutar tareas |
| `--syntax-check` | Valida sintaxis YAML/Ansible |

## Pruébalo tú

1. Entra en `examples/ansible-homelab`.
2. Ejecuta `ansible-playbook --syntax-check -i inventory.ini playbooks/ping.yml`.
3. Ejecuta el playbook normalmente.
4. Cambia el grupo del playbook a un grupo inexistente y observa el aviso.
5. Restaura el fichero.

## Errores comunes

- **Indentación YAML rota**: YAML depende de espacios; no mezcles tabs.
- **Playbook apunta a un grupo que no existe**: Ansible lo avisa, pero no ejecuta nada útil.
- **Usar nombres genéricos como `task 1`**: cuando falle, no sabrás qué hacía.

> Idea clave: un playbook convierte acciones repetibles en un fichero versionable y ejecutable tantas veces como necesites.
