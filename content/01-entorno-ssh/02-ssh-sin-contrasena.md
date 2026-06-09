---
title: "SSH sin contraseña"
slug: "ssh-sin-contrasena"
order: 2
summary: "Crear una clave SSH y comprobar acceso remoto antes de usar Ansible."
---

# SSH sin contraseña

Antes de escribir playbooks, necesitas que SSH funcione de forma limpia. Si SSH falla, Ansible también fallará.

## Teoría

Ansible usa SSH para conectarse a Linux. La forma práctica para homelab es usar clave pública:

| Elemento | Ubicación habitual |
| --- | --- |
| Clave privada | `~/.ssh/id_ed25519` |
| Clave pública | `~/.ssh/id_ed25519.pub` |
| Claves autorizadas remotas | `~/.ssh/authorized_keys` |
| Configuración SSH | `~/.ssh/config` |

La clave privada se queda en tu WSL. La pública se copia a cada VM o plantilla cloud-init.

## Manos a la obra

Crea una clave y prueba conexión contra una máquina Linux:

```compare
# CMD
ssh-keygen -t ed25519 -C "ansible-homelab" -f ~/.ssh/id_ed25519
ssh-copy-id usuario@192.168.1.50
ssh usuario@192.168.1.50 'hostname && whoami'
# OUT
Generating public/private ed25519 key pair.
...
<hostname-remoto>
usuario
```

Si la máquina se creó con cloud-init, normalmente pegarás el contenido de `id_ed25519.pub` en la plantilla o en el playbook de Proxmox.

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `ssh-keygen -t ed25519` | Crea una clave moderna y corta |
| `-C "comentario"` | Añade una etiqueta para reconocer la clave |
| `-f <ruta>` | Elige la ruta de la clave |
| `ssh-copy-id usuario@host` | Copia la clave pública al servidor |
| `ssh -i <clave>` | Usa una clave privada concreta |
| `StrictHostKeyChecking` | Controla la comprobación de identidad del host |

## Pruébalo tú

1. Crea tu clave en WSL si no existe.
2. Copia la clave pública a una VM Linux de pruebas.
3. Comprueba que `ssh usuario@ip 'hostname'` entra sin pedir contraseña.
4. Guarda la IP y el usuario: los usarás en el inventario del siguiente módulo.
5. Si vas a usar Proxmox cloud-init, copia el contenido de `~/.ssh/id_ed25519.pub`.

## Errores comunes

- **Permisos demasiado abiertos en `.ssh`**: ejecuta `chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_ed25519`.
- **Copiar la clave privada al servidor**: nunca hace falta; solo se copia la pública.
- **Cambió la IP o reinstalaste la VM**: puede aparecer un error de host key; revisa `ssh-keygen -R <ip>`.

> Idea clave: Ansible empieza por SSH fiable. Una clave bien configurada elimina fricción y evita meter contraseñas en playbooks.
