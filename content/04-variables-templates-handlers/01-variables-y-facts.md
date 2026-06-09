---
title: "Variables y facts"
slug: "variables-y-facts"
order: 1
summary: "Usar datos propios y datos detectados automáticamente por Ansible."
---

# Variables y facts

Las variables evitan duplicar valores. Los facts permiten tomar decisiones según el host real: distribución, IPs, memoria, arquitectura y más.

## Teoría

Puedes definir variables en varios sitios. Para este curso usaremos principalmente:

| Lugar | Uso |
| --- | --- |
| `inventory.ini` | Datos de conexión por host |
| `group_vars/all.yml` | Valores comunes del proyecto |
| `defaults/main.yml` en roles | Valores por defecto reutilizables |
| `vars:` en un play | Valores locales de un playbook |

Los facts se recogen automáticamente al inicio de un play, salvo que uses `gather_facts: false`.

## Manos a la obra

Consulta facts filtrados:

```compare
# CMD
cd examples/ansible-homelab
ansible docker_hosts -i inventory.ini -m setup -a "filter=ansible_distribution*"
# OUT
vm-docker-01 | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_major_version": "<version>",
        "ansible_distribution_release": "<release>",
        "ansible_distribution_version": "<version>"
    },
    "changed": false
}
```

## Flags y variantes

| Opción | Para qué sirve |
| --- | --- |
| `-m setup` | Recoge facts |
| `filter=...` | Limita facts mostrados |
| `gather_facts: true` | Activa facts en un play |
| `group_vars/all.yml` | Variables para todos los hosts |
| `host_vars/<host>.yml` | Variables específicas de un host |

## Pruébalo tú

1. Copia `group_vars/all.yml.example` como `group_vars/all.yml`.
2. Ajusta variables de Proxmox y Docker sin poner secretos reales si vas a commitear.
3. Ejecuta `setup` con filtro de distribución.
4. Ejecuta `setup` con filtro de arquitectura.
5. Busca una variable fact útil para condicionar tareas.

## Errores comunes

- **Meter secretos en `group_vars/all.yml` versionado**: usa ejemplos o Ansible Vault cuando haya credenciales reales.
- **Sobrescribir variables sin saber precedencia**: empieza simple; inventario + group_vars + defaults.
- **Desactivar facts y luego usarlos**: si `gather_facts: false`, `ansible_facts` no estará disponible.

> Idea clave: variables parametrizan tu intención; facts adaptan esa intención a cada máquina real.
