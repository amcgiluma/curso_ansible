---
title: "Built-ins esenciales"
slug: "builtins-esenciales"
order: 3
summary: "Conocer los módulos ansible.builtin más útiles para escribir tasks propias sin depender de shell."
---

# Built-ins esenciales

No necesitas memorizar todos los módulos de Ansible para ser productivo. Necesitas saber qué familias existen, cuáles usas a diario y cómo consultar la documentación correcta cuando te falte uno.

## Teoría

Estos `ansible.builtin` cubren la mayoría del trabajo práctico de un homelab:

| Familia | Módulos | Cuándo usarlos |
| --- | --- | --- |
| Conectividad y diagnóstico | `ping`, `debug`, `assert` | Verificar acceso, inspeccionar variables, validar precondiciones |
| Paquetes | `apt`, `package` | Instalar o quitar software de forma idempotente |
| Servicios | `service`, `systemd` | Arrancar, parar, habilitar servicios |
| Ficheros | `copy`, `template`, `file`, `lineinfile`, `blockinfile` | Gestionar contenido y permisos |
| Usuarios | `user`, `group` | Crear cuentas, grupos y membresías |
| Comandos | `command`, `shell` | Ejecutar órdenes concretas cuando no hay módulo mejor |
| HTTP y descargas | `get_url`, `uri` | Descargar artefactos o consultar endpoints |
| Datos y flujo | `set_fact`, `stat`, `include_tasks`, `import_tasks` | Guardar datos, consultar estado, dividir playbooks |

Reglas prácticas:

- Empieza buscando un built-in antes de usar `command` o `shell`.
- Usa `command` si no necesitas expansión de shell ni tuberías.
- Reserva `shell` para casos donde realmente necesitas redirecciones, pipes o sintaxis de shell.
- Consulta siempre `ansible-doc` antes de asumir nombres de argumentos.

Ejemplos reales del repositorio:

```yaml
- name: Instalar paquetes base
  ansible.builtin.apt:
    name:
      - ca-certificates
      - curl
    state: present

- name: Asegurar servicio Docker activo
  ansible.builtin.service:
    name: docker
    state: started
    enabled: true

- name: Mostrar version de Docker
  ansible.builtin.debug:
    var: docker_version.stdout
```

## Manos a la obra

Consulta la documentación local de varios built-ins y luego ejecuta el laboratorio:

```compare
# CMD
cd examples/ansible-homelab
ansible-doc ansible.builtin.copy
ansible-doc ansible.builtin.file
ansible-playbook -i inventory.ini playbooks/task-syntax-lab.yml --tags files
# OUT
> ANSIBLE.BUILTIN.COPY    (<...>)
> ANSIBLE.BUILTIN.FILE    (<...>)
...
PLAY [Laboratorio de sintaxis de tasks] **************************************

TASK [Asegurar directorio de laboratorio] ************************************
changed: [vm-docker-01]

TASK [Crear fichero de ejemplo] **********************************************
changed: [vm-docker-01]

PLAY RECAP *******************************************************************
vm-docker-01 : ok=<n> changed=<n> unreachable=0 failed=0
```

## Flags y variantes

| Comando o flag | Para qué sirve |
| --- | --- |
| `ansible-doc ansible.builtin.<modulo>` | Muestra la documentación de un módulo |
| `ansible-doc -l` | Lista módulos disponibles |
| `ansible-doc -t module copy` | Fuerza el tipo `module` al consultar docs |
| `--tags files` | Ejecuta solo tasks relacionadas con ficheros |
| `--tags packages` | Ejecuta solo tasks relacionadas con paquetes |
| `--list-tasks` | Enumera las tasks del playbook sin ejecutarlas |

## Pruébalo tú

1. Ejecuta `ansible-doc ansible.builtin.apt`.
2. Busca en la documentación qué hace `state: present`.
3. Ejecuta `ansible-doc ansible.builtin.user`.
4. Recorre `playbooks/task-syntax-lab.yml` e identifica qué task usa cada built-in.
5. Reescribe mentalmente una task con `shell` usando un módulo mejor si existe.

## Errores comunes

- **Tratar `shell` como martillo universal**: es cómodo, pero no es la mejor base para idempotencia.
- **No leer `ansible-doc`**: acabas inventando argumentos que el módulo no soporta.
- **Confundir `service` con `systemd`**: ambos sirven para servicios, pero `systemd` expone más opciones específicas.

> Idea clave: dominar Ansible no es memorizar mil módulos; es reconocer las familias útiles y saber encontrar rápido el built-in correcto.
