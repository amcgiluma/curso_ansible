---
title: "Checkpoint: Ansible operado desde Semaphore"
slug: "examen-practico"
order: 99
summary: "Validar que puedes ejecutar tus playbooks desde CLI y desde Semaphore."
---

# Checkpoint: Ansible operado desde Semaphore

El objetivo de este módulo es que tengas dos formas de operar tu homelab: terminal para construir y depurar, Semaphore para ejecutar tareas repetibles desde navegador.

## Teoría

Antes de dar por cerrado el módulo, confirma estas relaciones:

| Prueba | Qué confirma |
| --- | --- |
| `ansible --version` | Tu nodo de control puede ejecutar Ansible |
| `git pull --ff-only` | Estás usando el repo correcto y actualizado |
| `ansible-playbook ping.yml --list-hosts` | El inventario apunta a los hosts esperados |
| Template `Ping VMs` en Semaphore | La UI puede clonar el repo y ejecutar Ansible |
| Key Store configurado | Las credenciales no viven dentro del repo |

Semaphore debe ejecutar el mismo playbook que ya entiendes desde CLI. Esa simetría es lo que hace mantenible el sistema.

## Manos a la obra

Primero valida desde terminal:

```compare
# CMD
ansible --version
cd ~/homelab/curso_ansible/examples/ansible-homelab
git pull --ff-only
ansible-playbook playbooks/ping.yml --list-hosts
# OUT
ansible [core <version>]
Already up to date.
playbook: playbooks/ping.yml

  play #1 (docker_hosts): Comprobar conectividad Ansible
    hosts (<n>):
      vm-docker-01
```

Después crea en Semaphore una task template con:

```output
Repository: curso-ansible
Playbook: examples/ansible-homelab/playbooks/ping.yml
Inventory: homelab
Key Store: ssh-homelab
```

Cuando `ping.yml` funcione, añade una segunda plantilla para:

```output
examples/ansible-homelab/playbooks/install-docker.yml
```

Y deja la plantilla de Proxmox para el final:

```output
examples/ansible-homelab/playbooks/provision-docker-vm.yml
```

## Flags y variantes

| Comando u opción | Uso |
| --- | --- |
| `--list-hosts` | Validar alcance sin tocar sistemas |
| `git pull --ff-only` | Actualizar sin crear merges |
| `docker compose logs -f semaphore` | Ver errores de Semaphore |
| Key Store | Guardar claves y tokens fuera del repo |
| Environment | Pasar variables al playbook desde la UI |

## Pruébalo tú

1. Ejecuta `ping.yml --list-hosts` desde CLI.
2. Ejecuta `ping.yml` desde CLI.
3. Levanta Semaphore desde `examples/semaphore`.
4. Crea proyecto, repo, inventario, clave SSH y environment.
5. Ejecuta `ping.yml` desde Semaphore.
6. Ejecuta `install-docker.yml` desde Semaphore contra una VM de pruebas.
7. Documenta qué task templates quedan listas para tu homelab.

## Errores comunes

- **CLI funciona pero Semaphore no**: revisa credenciales, ruta del playbook, rama del repo y colecciones instaladas.
- **El repo local no coincide con Semaphore**: ambos deben apuntar al mismo remoto y rama.
- **El inventario real solo vive en una máquina**: decide si lo documentas, lo generas o lo subes cifrado.

> Idea clave: cuando CLI y Semaphore ejecutan el mismo playbook, ya tienes una base práctica para operar el homelab sin depender siempre de la terminal.
