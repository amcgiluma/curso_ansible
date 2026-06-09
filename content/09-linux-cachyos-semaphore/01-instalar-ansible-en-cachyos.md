---
title: "Instalar Ansible en CachyOS"
slug: "instalar-ansible-en-cachyos"
order: 1
summary: "Preparar una máquina Linux real como nodo de control Ansible usando pacman o pipx."
---

# Instalar Ansible en CachyOS

En Windows usas WSL como nodo de control. En CachyOS puedes ejecutar Ansible directamente desde Linux, con las mismas claves SSH y el mismo repositorio de playbooks.

## Teoría

CachyOS está basado en Arch, así que el flujo natural es `pacman`. Para este curso hay dos opciones válidas:

| Opción | Cuándo usarla |
| --- | --- |
| `pacman -S ansible-core` | Integración limpia con el sistema y paquetes de la distro |
| `pipx install --include-deps ansible` | Versión del paquete completo aislada del Python del sistema |

La decisión práctica: empieza con `pacman` si quieres simplicidad. Usa `pipx` si quieres mantener Ansible aislado y más parecido al flujo usado en WSL.

## Manos a la obra

Instalación directa con pacman:

```compare
# CMD
sudo pacman -Syu
sudo pacman -S ansible-core git openssh python
ansible --version
# OUT
ansible [core <version>]
  config file = <ruta>
  configured module search path = [...]
  python version = <version>
```

Alternativa con pipx:

```compare
# CMD
sudo pacman -Syu
sudo pacman -S python python-pipx git openssh
pipx ensurepath
pipx install --include-deps ansible
ansible --version
# OUT
ansible [core <version>]
  executable location = /home/<usuario>/.local/bin/ansible
  python version = <version>
```

## Flags y variantes

| Comando | Uso |
| --- | --- |
| `sudo pacman -Syu` | Sincroniza repositorios y actualiza el sistema antes de instalar |
| `sudo pacman -S ansible-core` | Instala Ansible Core desde repositorios |
| `pipx ensurepath` | Añade binarios de pipx al `PATH` del usuario |
| `pipx install --include-deps ansible` | Instala el paquete completo de Ansible aislado |
| `ansible --version` | Verifica ejecutable, versión y Python usado |

## Pruébalo tú

1. Abre una terminal en CachyOS.
2. Elige `pacman` o `pipx`; no mezcles ambos al principio.
3. Ejecuta `ansible --version`.
4. Crea una carpeta de trabajo: `mkdir -p ~/homelab/ansible && cd ~/homelab/ansible`.
5. Copia o clona ahí el repositorio de ejemplos cuando lo subas a GitHub.

## Errores comunes

- **`ansible: command not found` con pipx**: cierra y reabre la terminal o ejecuta `source ~/.bashrc`, `source ~/.zshrc` o el fichero de shell que uses.
- **Actualizar solo la base de datos con `pacman -Sy`**: evita actualizaciones parciales; usa `sudo pacman -Syu`.
- **Instalar Ansible en varias rutas**: si `which ansible` apunta a un sitio inesperado, limpia una de las instalaciones.

> Idea clave: CachyOS puede ser un nodo de control Ansible completo; lo importante es mantener una sola instalación clara y verificarla con `ansible --version`.
