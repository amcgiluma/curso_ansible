---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Reto para validar Ansible en CachyOS y ejecutar un playbook desde Semaphore."
---

# Examen práctico

El reto es dejar tu máquina CachyOS lista como nodo de control y lanzar un playbook desde Semaphore usando el repositorio compartido.

## Teoría

Debes demostrar tres cosas:

| Prueba | Qué confirma |
| --- | --- |
| `ansible --version` | Ansible instalado y usable en CachyOS |
| `git pull --ff-only` | Repo compartido y actualizado |
| Template en Semaphore | Ejecución reproducible desde navegador |

## Manos a la obra

Secuencia mínima:

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

En Semaphore, ejecuta una plantilla apuntando a:

```output
examples/ansible-homelab/playbooks/ping.yml
```

## Flags y variantes

| Comando u opción | Uso |
| --- | --- |
| `--list-hosts` | Validar alcance sin tocar sistemas |
| `git pull --ff-only` | Actualizar sin merges |
| `docker compose logs -f semaphore` | Depurar Semaphore |
| Key Store | Guardar claves y tokens fuera del repo |

## Pruébalo tú

1. Instala Ansible en CachyOS.
2. Clona `curso_ansible`.
3. Configura un inventario real.
4. Levanta Semaphore desde `examples/semaphore`.
5. Crea un proyecto conectado al repo.
6. Ejecuta `ping.yml` desde la UI.
7. Guarda una captura o nota con fecha, host afectado y resultado.

## Errores comunes

- **La CLI funciona pero Semaphore no**: revisa credenciales, ruta del playbook y rama del repo.
- **El repo clonado en CachyOS no coincide con Semaphore**: ambos deben apuntar al mismo remoto y rama.
- **El inventario real se queda solo en una máquina**: documenta dónde vive o súbelo cifrado si lo necesitas compartido.

> Idea clave: si CachyOS, GitHub y Semaphore ejecutan el mismo `ping.yml`, ya tienes una base compartida para automatizar tu homelab desde varias máquinas.
