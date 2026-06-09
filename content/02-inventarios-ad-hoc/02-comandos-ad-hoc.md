---
title: "Comandos ad-hoc"
slug: "comandos-ad-hoc"
order: 2
summary: "Ejecutar comprobaciones rápidas sin escribir un playbook completo."
---

# Comandos ad-hoc

Los comandos ad-hoc sirven para preguntar o actuar una vez: hacer ping Ansible, ver uptime, instalar un paquete puntual o comprobar conectividad.

## Teoría

La forma general es:

```bash
ansible <patron> -i <inventario> -m <modulo> -a '<argumentos>'
```

El patrón puede ser un grupo, un host o `all`. El módulo define qué acción se ejecuta.

| Módulo | Uso típico |
| --- | --- |
| `ping` | Comprueba conexión Ansible y Python remoto |
| `command` | Ejecuta un comando sin shell |
| `shell` | Ejecuta un comando con shell |
| `setup` | Recoge facts del host |
| `apt` | Gestiona paquetes en Debian/Ubuntu |

## Manos a la obra

Comprueba conexión y datos básicos:

```compare
# CMD
cd examples/ansible-homelab
ansible docker_hosts -i inventory.ini -m ping
ansible docker_hosts -i inventory.ini -m command -a "uptime"
# OUT
vm-docker-01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
vm-docker-01 | CHANGED | rc=0 >>
 <hora> up <tiempo>,  <usuarios> user,  load average: <...>
```

## Flags y variantes

| Flag / opción | Para qué sirve |
| --- | --- |
| `-m ping` | Usa el módulo `ping` |
| `-a "..."` | Pasa argumentos al módulo |
| `-b` | Ejecuta con `become`/sudo |
| `--limit <host>` | Limita la ejecución a un subconjunto |
| `-o` | Salida compacta de una línea |
| `-v`, `-vv`, `-vvv` | Aumenta detalle para depurar |

## Pruébalo tú

1. Usa `ansible docker_hosts -i inventory.ini -m ping`.
2. Ejecuta `ansible all -i inventory.ini -m command -a "hostname"`.
3. Prueba `ansible all -i inventory.ini -m setup -a "filter=ansible_distribution*"`.
4. Ejecuta un comando con `--limit vm-docker-01`.
5. Repite con `-vv` si quieres ver más detalle de conexión.

## Errores comunes

- **`UNREACHABLE`**: problema SSH, usuario o IP.
- **`FAILED` en `ping` por Python**: la máquina remota no tiene Python suficiente.
- **Usar `shell` para todo**: prefiere módulos específicos o `command`; `shell` tiene más riesgo.

> Idea clave: los ad-hoc son el laboratorio rápido; cuando repites una acción, conviértela en playbook.
