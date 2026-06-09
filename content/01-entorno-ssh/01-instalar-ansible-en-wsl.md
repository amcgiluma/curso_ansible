---
title: "Instalar Ansible en WSL Ubuntu"
slug: "instalar-ansible-en-wsl"
order: 1
summary: "Preparar el nodo de control desde el que ejecutarás todos los playbooks."
---

# Instalar Ansible en WSL Ubuntu

Ansible se ejecuta desde una máquina de control Linux. En Windows, WSL Ubuntu es la opción más directa: tienes herramientas Linux, SSH y Python sin levantar una VM aparte.

## Teoría

Ansible no instala un agente permanente en los servidores. Desde el nodo de control se conecta por SSH, copia módulos temporales, ejecuta acciones y devuelve el resultado.

Para este curso necesitas:

| Pieza | Para qué sirve |
| --- | --- |
| WSL Ubuntu | Entorno Linux local para ejecutar Ansible |
| Python 3 | Runtime usado por Ansible y por muchos módulos |
| `pipx` | Instala herramientas Python aisladas |
| Ansible | CLI, módulos, inventarios y playbooks |
| SSH | Transporte hacia máquinas Linux y VMs Proxmox |

> Nota: Ansible puede instalarse con `apt`, pero `pipx` suele dar una versión más actual y evita mezclar paquetes Python del sistema.

## Manos a la obra

Instala Ansible dentro de WSL Ubuntu:

```compare
# CMD
sudo apt update
sudo apt install -y python3 python3-pip pipx openssh-client sshpass
pipx ensurepath
pipx install --include-deps ansible
ansible --version
# OUT
ansible [core <version>]
  config file = None
  python version = <version>
  executable location = /home/<usuario>/.local/bin/ansible
```

Si acabas de ejecutar `pipx ensurepath`, puede que necesites cerrar y abrir la terminal.

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `sudo apt update` | Actualiza el índice de paquetes de Ubuntu |
| `pipx install --include-deps ansible` | Instala Ansible y dependencias en un entorno aislado |
| `pipx upgrade ansible` | Actualiza Ansible cuando quieras renovar la herramienta |
| `ansible --version` | Comprueba versión, Python usado y ruta del ejecutable |
| `sshpass` | Permite pruebas iniciales con contraseña, aunque el curso usará claves |

## Pruébalo tú

1. Abre WSL Ubuntu.
2. Ejecuta los comandos de instalación.
3. Cierra y reabre la terminal si `ansible` no aparece en el `PATH`.
4. Ejecuta `ansible --version` y confirma que ves `ansible [core ...]`.
5. Crea una carpeta de trabajo: `mkdir -p ~/ansible-homelab && cd ~/ansible-homelab`.

## Errores comunes

- **`ansible: command not found`**: falta recargar el `PATH`; cierra y abre WSL o ejecuta `source ~/.profile`.
- **Instalar Ansible en PowerShell**: no es el flujo recomendado para este curso; trabaja desde WSL.
- **Mezclar `pip install` global con paquetes del sistema**: usa `pipx` para evitar conflictos.

> Idea clave: tu nodo de control será WSL Ubuntu; desde ahí Ansible gestionará máquinas Linux por SSH sin instalar agentes permanentes.
