---
title: "Compartir playbooks entre Windows y Linux"
slug: "repo-compartido-git"
order: 2
summary: "Usar GitHub como punto común para ejecutar los mismos playbooks desde WSL, CachyOS o Semaphore."
---

# Compartir playbooks entre Windows y Linux

El objetivo es no tener una copia distinta de tus scripts en cada máquina. GitHub será la fuente de verdad y cada nodo de control solo clonará o actualizará el repositorio.

## Teoría

Ansible funciona mejor cuando todo lo repetible está versionado:

| Elemento | Se versiona | No se versiona |
| --- | --- | --- |
| Playbooks | Sí | |
| Roles | Sí | |
| `requirements.yml` | Sí | |
| Inventario de ejemplo | Sí, como `.example` | |
| Inventario real con IPs privadas | Depende | Si contiene datos sensibles |
| Tokens de Proxmox | No | Usa `.env`, Vault o Semaphore Key Store |

Tu flujo recomendado:

1. Editas playbooks en Windows/WSL o CachyOS.
2. Haces commit y push.
3. En la otra máquina haces pull.
4. Semaphore también apunta al mismo repositorio.

## Manos a la obra

Clona el repositorio del curso o de tus playbooks:

```compare
# CMD
mkdir -p ~/homelab
cd ~/homelab
git clone https://github.com/amcgiluma/curso_ansible.git
cd curso_ansible/examples/ansible-homelab
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/ping.yml --list-hosts
# OUT
playbook: playbooks/ping.yml

  play #1 (docker_hosts): Comprobar conectividad Ansible
    pattern: ['docker_hosts']
    hosts (<n>):
      vm-docker-01
```

Configura identidad Git en CachyOS:

```compare
# CMD
git config --global user.name "Juanma"
git config --global user.email "tu-email@example.com"
git config --global init.defaultBranch main
git config --global --list
# OUT
user.name=Juanma
user.email=tu-email@example.com
init.defaultbranch=main
```

## Flags y variantes

| Comando | Uso |
| --- | --- |
| `git clone <url>` | Crea una copia local del repo |
| `git pull --ff-only` | Actualiza sin crear merges accidentales |
| `git status --short` | Revisa cambios locales |
| `ansible-galaxy collection install -r requirements.yml` | Instala dependencias declaradas |
| `--list-hosts` | Comprueba alcance antes de ejecutar |

## Pruébalo tú

1. Clona el repositorio en CachyOS dentro de `~/homelab`.
2. Entra en `examples/ansible-homelab`.
3. Copia `inventory.ini.example` a `inventory.ini` y ajusta una IP real.
4. Ejecuta `ansible-playbook playbooks/ping.yml --list-hosts`.
5. Haz un cambio pequeño en un README, commitea y comprueba que puedes hacer pull desde la otra máquina.

## Errores comunes

- **Versionar secretos**: no subas tokens de Proxmox, contraseñas ni claves privadas.
- **Trabajar en dos máquinas sin hacer pull antes**: usa `git pull --ff-only` al empezar.
- **Semaphore no ve tus cambios**: revisa rama, credenciales y que hayas hecho push.

> Idea clave: GitHub sincroniza el conocimiento; Ansible y Semaphore ejecutan lo que está versionado.
