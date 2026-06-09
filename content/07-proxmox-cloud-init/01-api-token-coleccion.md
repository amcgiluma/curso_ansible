---
title: "API token, permisos y colección Proxmox"
slug: "api-token-coleccion"
order: 1
summary: "Preparar Proxmox para que Ansible pueda crear VMs usando un token controlado."
---

# API token, permisos y colección Proxmox

Antes de crear una VM desde Ansible hay que preparar dos cosas: Proxmox debe aceptar llamadas de automatización y tu proyecto Ansible debe tener los módulos adecuados para hablar con su API.

## Teoría

Ansible no entra en la interfaz web de Proxmox ni simula clics. Llama a la API HTTP de Proxmox con un usuario, un token y una serie de parámetros. Esa API es la misma vía que usa la interfaz web por debajo, pero con una ventaja importante: puedes guardar la infraestructura como código.

El flujo que vamos a construir es este:

| Paso | Resultado |
| --- | --- |
| Crear un usuario o usar uno existente | Identidad con la que Ansible se presenta ante Proxmox |
| Crear un API token | Secreto no interactivo para ejecutar playbooks sin contraseña |
| Dar permisos al usuario/token | Capacidad de clonar, configurar y arrancar VMs |
| Instalar `community.general` | Módulos Ansible como `community.general.proxmox_kvm` |
| Guardar variables fuera del código sensible | Playbooks reutilizables sin subir secretos a Git |

Para empezar en tu homelab puedes usar `root@pam` con un token, porque reduce fricción y te permite avanzar. Para un entorno más serio, crea un usuario dedicado como `ansible@pve` y dale solo los permisos necesarios.

| Opción | Ventaja | Coste |
| --- | --- | --- |
| `root@pam` + token | Rápido para laboratorio local | Mucho poder si filtras el token |
| Usuario dedicado + token | Mejor separación de permisos | Requiere configurar ACLs en Proxmox |

En ambos casos, el secreto real nunca debe ir al repositorio. En el curso usamos `group_vars/all.yml.example` como plantilla y tú creas tu `group_vars/all.yml` local.

## Manos a la obra

### 1. Crear el token en Proxmox

En la interfaz web de Proxmox:

1. Entra en `Datacenter`.
2. Abre `Permissions`.
3. Entra en `API Tokens`.
4. Pulsa `Add`.
5. Elige el usuario, por ejemplo `root@pam`.
6. Pon un nombre claro al token, por ejemplo `ansible`.
7. Guarda el `Token ID` y el `Secret`.

Proxmox te mostrará el secreto una sola vez. En Ansible lo separaremos así:

```yaml
proxmox_api_user: "root@pam"
proxmox_api_token_id: "ansible"
proxmox_api_token_secret: "el-secreto-que-te-da-proxmox"
```

Con esos tres valores, Ansible se autenticará como `root@pam!ansible`. En los módulos de Ansible no escribimos el usuario completo con `!`; el módulo recibe `api_user`, `api_token_id` y `api_token_secret` por separado.

### 2. Dar permisos al token

Si usas `root@pam`, normalmente el token heredará permisos suficientes si no marcas separación estricta de privilegios. Para un usuario dedicado, la idea sería darle permisos en `Datacenter > Permissions` sobre la ruta que necesites.

Para este curso, los playbooks necesitan poder:

| Acción | Por qué |
| --- | --- |
| Leer nodos, storages y VMs | Comprobar estado y encontrar la plantilla |
| Clonar una template | Crear la VM nueva |
| Modificar configuración cloud-init | Inyectar usuario, clave SSH y red |
| Arrancar la VM | Dejarla lista para conectarse por SSH |

En un homelab cerrado puedes empezar con permisos amplios para aprender el flujo. Cuando funcione, vuelves y reduces permisos.

### 3. Instalar la colección de Ansible

Desde la carpeta del ejemplo:

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

El archivo `requirements.yml` declara las colecciones externas que usa el proyecto:

```yaml
---
collections:
  - name: community.general
```

La colección `community.general` contiene el módulo `proxmox_kvm`, que es el que clona y configura VMs KVM en Proxmox. Sin esta colección, Ansible entendería tus playbooks básicos, pero fallaría al llegar a las tareas específicas de Proxmox.

### 4. Crear tus variables reales

Copia la plantilla:

```bash
cp group_vars/all.yml.example group_vars/all.yml
```

El archivo real debe quedar con tus datos. Este es el bloque importante:

```yaml
proxmox_api_host: "192.168.1.10"
proxmox_api_user: "root@pam"
proxmox_api_token_id: "ansible"
proxmox_api_token_secret: "PROXMOX_API_TOKEN_SECRET"
proxmox_validate_certs: false

proxmox_node: "pve"
proxmox_storage: "local-lvm"
proxmox_bridge: "vmbr0"
proxmox_template: "ubuntu-2404-cloudinit-template"
```

Qué hace cada variable:

| Variable | Qué controla |
| --- | --- |
| `proxmox_api_host` | IP o DNS del nodo Proxmox al que llamará Ansible |
| `proxmox_api_user` | Usuario propietario del token |
| `proxmox_api_token_id` | Nombre corto del token creado en Proxmox |
| `proxmox_api_token_secret` | Secreto real del token |
| `proxmox_validate_certs` | Si Ansible valida el certificado HTTPS de Proxmox |
| `proxmox_node` | Nodo físico donde se creará la VM |
| `proxmox_storage` | Storage donde irá el disco de la VM |
| `proxmox_bridge` | Bridge de red que usará la tarjeta virtual |
| `proxmox_template` | Nombre o VMID de la template cloud-init que clonaremos |

En muchos homelabs `proxmox_validate_certs: false` es normal porque Proxmox usa certificado autofirmado. Cuando tengas dominio interno y CA propia puedes cambiarlo.

## Flags y variantes

| Variable / comando | Para qué sirve |
| --- | --- |
| `proxmox_api_host` | Host o IP de Proxmox |
| `proxmox_api_user` | Usuario API, por ejemplo `root@pam` o usuario dedicado |
| `proxmox_api_token_id` | ID del token |
| `proxmox_api_token_secret` | Secreto del token |
| `ansible-galaxy collection install` | Instala módulos externos |
| `ansible-doc community.general.proxmox_kvm` | Muestra parámetros soportados por el módulo |
| `proxmox_validate_certs` | Controla la validación TLS contra la API |

## Pruébalo tú

1. Crea el token en Proxmox y guarda su secreto en un gestor de contraseñas.
2. Entra en `examples/ansible-homelab`.
3. Instala la colección con `ansible-galaxy collection install -r requirements.yml`.
4. Copia `group_vars/all.yml.example` a `group_vars/all.yml`.
5. Rellena host, usuario, token, nodo, storage, bridge y nombre de template.
6. Ejecuta `ansible-doc community.general.proxmox_kvm` para confirmar que el módulo está disponible.
7. No ejecutes todavía el playbook de creación. Primero prepararemos bien la template cloud-init.

## Errores comunes

- **Guardar el secreto en Git**: solo se versiona `all.yml.example`; tu `all.yml` real debe quedarse local.
- **Confundir token ID con secreto**: el ID suele ser `ansible`; el secreto es la cadena larga que Proxmox muestra una vez.
- **No instalar la colección**: si falta `community.general`, Ansible no sabrá qué es `community.general.proxmox_kvm`.

> Idea clave: primero damos a Ansible una identidad controlada en Proxmox; después los playbooks podrán crear VMs sin depender de clics ni contraseñas interactivas.
