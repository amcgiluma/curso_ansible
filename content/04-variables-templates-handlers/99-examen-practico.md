---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Debes demostrar que puedes leer variables, consultar facts y entender cuándo se ejecutaría un handler.

## Teoría

La automatización útil necesita datos. Variables, facts y handlers separan configuración, detección y reacción.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible docker_hosts -i inventory.ini -m setup -a "filter=ansible_default_ipv4"
ansible-playbook -i inventory.ini playbooks/install-docker.yml --list-tasks
# OUT
vm-docker-01 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "<ip>",
            ...
        }
    },
    "changed": false
}
...
```

## Flags y variantes

| Opción | Para qué sirve |
| --- | --- |
| `filter=` | Reduce facts |
| `--list-tasks` | Inspecciona playbooks |
| `group_vars` | Centraliza variables |
| `notify` | Dispara handlers |

## Pruébalo tú

1. Consulta facts de distribución e IP.
2. Localiza variables en `group_vars/all.yml.example`.
3. Explica qué valores cambiarías para tu Proxmox.
4. Localiza el handler del rol Docker.

## Errores comunes

- **Guardar tokens reales en ejemplos**: usa placeholders.
- **No distinguir variables propias de facts**: las propias las defines tú; los facts los detecta Ansible.

> Idea clave: cuando entiendes variables y facts, puedes escribir playbooks que se adaptan sin duplicar lógica.
