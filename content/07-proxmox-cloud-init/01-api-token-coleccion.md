---
title: "API token y colección Proxmox"
slug: "api-token-coleccion"
order: 1
summary: "Configurar el acceso de Ansible a Proxmox sin usar contraseñas interactivas."
---

# API token y colección Proxmox

Para crear VMs desde Ansible necesitas hablar con la API de Proxmox. El camino limpio es usar un token con permisos limitados.

## Teoría

El flujo recomendado:

| Paso | Resultado |
| --- | --- |
| Crear usuario o usar uno existente | Identidad para automatización |
| Crear API token | Secreto no interactivo |
| Asignar permisos mínimos | Menos riesgo |
| Instalar `community.general` | Módulos Proxmox disponibles |
| Guardar variables | Playbooks parametrizados |

No guardes tokens reales en ficheros versionados. Para el curso se usan placeholders y `.example`.

## Manos a la obra

Instala la colección y comprueba que Ansible la ve:

```compare
# CMD
cd examples/ansible-homelab
ansible-galaxy collection install -r requirements.yml
ansible-doc community.general.proxmox_kvm | head -n 5
# OUT
Starting galaxy collection install process
...
> COMMUNITY.GENERAL.PROXMOX_KVM
...
```

## Flags y variantes

| Variable / comando | Para qué sirve |
| --- | --- |
| `proxmox_api_host` | Host o IP de Proxmox |
| `proxmox_api_user` | Usuario API, por ejemplo `root@pam` o usuario dedicado |
| `proxmox_api_token_id` | ID del token |
| `proxmox_api_token_secret` | Secreto del token |
| `ansible-galaxy collection install` | Instala módulos externos |

## Pruébalo tú

1. Crea o identifica un usuario API en Proxmox.
2. Crea un token para automatización.
3. Copia `group_vars/all.yml.example` a `group_vars/all.yml`.
4. Rellena host, usuario, token ID y token secret.
5. Ejecuta `ansible-doc community.general.proxmox_kvm`.

## Errores comunes

- **Usar contraseña interactiva**: para automatización usa token.
- **Token sin permisos suficientes**: la API conecta, pero la creación de VM falla.
- **Commitear el secreto**: usa `.example` y excluye el fichero real si versionas el proyecto.

> Idea clave: Proxmox se automatiza de forma limpia con API tokens y módulos Ansible, no con clics manuales.
