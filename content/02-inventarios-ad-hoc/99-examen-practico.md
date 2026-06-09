---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Vas a demostrar que puedes declarar una VM en un inventario y operarla con comandos ad-hoc.

## Teoría

El inventario da contexto; los comandos ad-hoc validan que el contexto funciona antes de escribir automatización más grande.

## Manos a la obra

```compare
# CMD
cd examples/ansible-homelab
ansible-inventory -i inventory.ini --graph
ansible docker_hosts -i inventory.ini -m ping
# OUT
@all:
  |--@ungrouped:
  |--@docker_hosts:
  |  |--vm-docker-01
vm-docker-01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Flags y variantes

| Flag / comando | Para qué sirve |
| --- | --- |
| `--graph` | Ver estructura de grupos |
| `-m ping` | Probar conexión Ansible |
| `-m setup` | Ver facts |
| `--limit` | Acotar el alcance |

## Pruébalo tú

1. Crea `inventory.ini`.
2. Añade un grupo `docker_hosts`.
3. Ejecuta `ansible-inventory --graph`.
4. Ejecuta `ping`, `hostname` y `uptime`.
5. Documenta qué error aparece si cambias la IP por una inexistente.

## Errores comunes

- **Inventario válido pero host inaccesible**: son problemas distintos; primero lista inventario, luego prueba conexión.
- **Ejecutar contra `all` sin querer**: en homelab pequeño parece inocuo, pero acostúmbrate a grupos concretos.

> Idea clave: si puedes listar hosts y hacer `ping`, ya tienes el inventario mínimo operativo.
