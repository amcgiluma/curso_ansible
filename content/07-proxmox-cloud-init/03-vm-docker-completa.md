---
title: "Flujo completo: VM creada y Docker instalado"
slug: "vm-docker-completa"
order: 3
summary: "Encadenar Proxmox, SSH y Docker para dejar una VM útil desde un solo playbook."
---

# Flujo completo: VM creada y Docker instalado

Ahora juntamos las piezas: Ansible crea la VM en Proxmox, espera a que SSH responda y después instala Docker dentro de la máquina. Este es el primer flujo realmente útil para tu homelab.

## Teoría

El playbook `provision-docker-vm.yml` tiene tres jugadas separadas. Separarlas hace que el flujo sea más fácil de entender:

| Jugada | Dónde se ejecuta | Qué hace |
| --- | --- | --- |
| Crear VM | `localhost` | Llama a la API de Proxmox y clona la template |
| Esperar SSH | `localhost` | Comprueba que la IP de la VM ya acepta conexiones |
| Instalar Docker | VM nueva | Entra por SSH y aplica el rol `docker` |

La primera parte no necesita SSH contra la VM porque la VM todavía no existe. Usa la API de Proxmox. La tercera parte sí necesita SSH porque ya está configurando el sistema operativo invitado.

## Manos a la obra

### 1. Variables finales que usará el flujo

En `examples/ansible-homelab/group_vars/all.yml` debes tener un bloque parecido a este:

```yaml
proxmox_vm_id: 9101
proxmox_vm_name: "vm-docker-01"
proxmox_vm_cores: 2
proxmox_vm_memory: 2048
proxmox_vm_disk: "20G"
proxmox_vm_ciuser: "ubuntu"
proxmox_vm_ip: "192.168.1.50"
proxmox_vm_gateway: "192.168.1.1"
proxmox_vm_ipconfig: "ip=192.168.1.50/24,gw=192.168.1.1"
proxmox_vm_ssh_public_key: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_ed25519.pub') }}"
```

Qué significa:

| Variable | Qué decide |
| --- | --- |
| `proxmox_vm_id` | ID único de la VM nueva en Proxmox |
| `proxmox_vm_name` | Nombre de la VM y del host temporal en Ansible |
| `proxmox_vm_cores` | vCPU asignadas |
| `proxmox_vm_memory` | RAM en MB |
| `proxmox_vm_disk` | Tamaño deseado del disco |
| `proxmox_vm_ciuser` | Usuario creado por cloud-init |
| `proxmox_vm_ip` | IP que Ansible usará para esperar SSH |
| `proxmox_vm_ipconfig` | Configuración que Proxmox inyecta por cloud-init |
| `proxmox_vm_ssh_public_key` | Clave pública que entrará en `authorized_keys` |

Para tu servidor actual, evita sobredimensionar las VMs. Una VM Docker base con `2 vCPU` y `2 GB RAM` está bien para empezar. Reserva RAM para Proxmox, ZFS/caché si aplica y servicios de red.

### 2. Crear la VM y añadirla al inventario temporal

La primera jugada del playbook es esta:

```yaml
- name: Crear VM cloud-init en Proxmox
  hosts: localhost
  gather_facts: false
  connection: local

  roles:
    - role: proxmox_vm
      tags: ["proxmox"]

  post_tasks:
    - name: Añadir VM creada al inventario temporal
      ansible.builtin.add_host:
        name: "{{ proxmox_vm_name }}"
        groups: created_docker_hosts
        ansible_host: "{{ proxmox_vm_ip }}"
        ansible_user: "{{ proxmox_vm_ciuser }}"
        ansible_become: true
```

`add_host` no modifica tu `inventory.ini`. Crea un host solo para esta ejecución. Eso permite que el mismo playbook cree una VM y después la gestione sin que tengas que editar inventario a mano entre medias.

### 3. Esperar a que SSH esté listo

La VM puede aparecer en Proxmox antes de estar lista para aceptar SSH. Por eso hay una espera explícita:

```yaml
- name: Esperar SSH en la VM nueva
  hosts: localhost
  gather_facts: false
  connection: local

  tasks:
    - name: Esperar a que SSH responda
      ansible.builtin.wait_for:
        host: "{{ proxmox_vm_ip }}"
        port: 22
        delay: 10
        timeout: 300
        state: started
```

`delay: 10` da margen al arranque inicial. `timeout: 300` permite hasta cinco minutos, suficiente para el primer arranque de una imagen cloud.

### 4. Instalar Docker dentro de la VM

La tercera jugada ya apunta al grupo temporal:

```yaml
- name: Instalar Docker en la VM creada
  hosts: created_docker_hosts
  become: true
  gather_facts: true

  roles:
    - role: docker
      tags: ["docker"]
```

Aquí Ansible entra por SSH como `ubuntu`, eleva privilegios con `become` y ejecuta el rol `docker`.

Después valida la instalación:

```yaml
post_tasks:
  - name: Validar Docker con hello-world
    ansible.builtin.command: docker run --rm hello-world
    register: docker_hello_world
    changed_when: false

  - name: Mostrar resultado de validación
    ansible.builtin.debug:
      var: docker_hello_world.stdout_lines
```

`changed_when: false` evita marcar la validación como cambio real de infraestructura. Es una comprobación, no una modificación deseada.

### 5. Ejecutar el flujo completo

Desde `examples/ansible-homelab`:

```compare
# CMD
ansible-playbook playbooks/provision-docker-vm.yml
# OUT
PLAY [Crear VM cloud-init en Proxmox] *****************************************
...
TASK [proxmox_vm : Clonar VM desde template cloud-init] ***********************
changed: [localhost]
...
PLAY [Esperar SSH en la VM nueva] *********************************************
...
TASK [Esperar a que SSH responda] *********************************************
ok: [localhost]
...
PLAY [Instalar Docker en la VM creada] ****************************************
...
TASK [docker : Instalar paquetes Docker] **************************************
changed: [vm-docker-01]
...
TASK [Mostrar resultado de validación] ****************************************
ok: [vm-docker-01] => {
  "docker_hello_world.stdout_lines": [
    "Hello from Docker!",
    "This message shows that your installation appears to be working correctly."
  ]
}
```

### 6. Resultado que buscamos

Al acabar debes tener:

| Recurso | Estado |
| --- | --- |
| VM en Proxmox | Creada desde template cloud-init |
| IP | Asignada por cloud-init |
| SSH | Acceso con clave pública |
| Usuario | `ubuntu` con sudo |
| Docker | Instalado y validado |

Ese será el patrón que reutilizarás para servicios del homelab: crear máquina, esperar acceso, aplicar roles y validar.

## Flags y variantes

| Opción | Para qué sirve |
| --- | --- |
| `--tags proxmox` | Ejecuta solo la parte de creación si el playbook está preparado para tags |
| `--tags docker` | Ejecuta solo el rol Docker sobre hosts ya conocidos |
| `--limit` | Limita ejecución a un host o grupo concreto |
| `-e proxmox_vm_name=...` | Sobrescribe variables puntuales desde CLI |
| `ansible.builtin.add_host` | Crea inventario temporal durante la ejecución |
| `ansible.builtin.wait_for` | Espera a que un puerto o archivo esté disponible |

## Pruébalo tú

1. Confirma que la template cloud-init existe y no está arrancada como VM normal.
2. Ajusta `proxmox_vm_id`, `proxmox_vm_name` e IP en `group_vars/all.yml`.
3. Ejecuta `ansible-playbook playbooks/provision-docker-vm.yml`.
4. Entra por SSH en la VM creada.
5. Ejecuta `docker ps` y `docker run --rm hello-world`.
6. Anota en tu documentación qué recursos consumió y qué servicios quieres desplegar ahí.

## Errores comunes

- **Ejecutar dos veces con el mismo VMID**: el segundo intento no creará otra VM; cambia VMID/nombre o destruye la prueba anterior manualmente.
- **IP fija ya ocupada**: `wait_for` puede conectar al host equivocado o fallar. Reserva IPs para automatización.
- **Clave pública incorrecta**: la VM arranca, pero Ansible no puede entrar por SSH.

> Idea clave: este playbook ya no es una demo; es una cadena completa de provisión para crear una VM útil en tu homelab.
