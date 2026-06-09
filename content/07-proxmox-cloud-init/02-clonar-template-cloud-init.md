---
title: "Crear la plantilla cloud-init en Proxmox"
slug: "clonar-template-cloud-init"
order: 2
summary: "Preparar paso a paso la template que Ansible clonará después."
---

# Crear la plantilla cloud-init en Proxmox

La template cloud-init es la pieza más importante del flujo. Si está bien hecha, crear una VM nueva será cuestión de cambiar variables en Ansible. Si está mal hecha, la VM puede clonar pero arrancar sin usuario, sin red o sin clave SSH.

## Teoría

Cloud-init es un sistema de inicialización usado por imágenes cloud. En vez de instalar Ubuntu a mano con un asistente gráfico, descargas una imagen preparada y le pasas metadatos en el primer arranque.

En Proxmox, esos metadatos salen de la unidad `CloudInit Drive`. Ahí Proxmox escribe cosas como:

| Dato | Qué consigue |
| --- | --- |
| Usuario inicial | Crea el usuario con el que entrará Ansible |
| Clave SSH pública | Permite login sin contraseña |
| Red | DHCP o IP fija desde el primer arranque |
| DNS | Resolución de nombres dentro de la VM |
| Hostname | Nombre coherente de la máquina |

Nuestro objetivo no es tener una VM concreta, sino una fábrica de VMs. La template será la base y Ansible decidirá el nombre, VMID, CPU, RAM, IP y clave.

## Manos a la obra

Los comandos de esta sección se ejecutan en el host Proxmox, por SSH o desde la shell web de Proxmox. Ajusta nombres de storage si tu instalación usa otros.

### 1. Descargar una imagen cloud

Usaremos Ubuntu Server 24.04 cloud image como base:

```compare
# CMD
cd /var/lib/vz/template/iso
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
ls -lh noble-server-cloudimg-amd64.img
# OUT
-rw-r--r-- 1 root root <tamano> <fecha> noble-server-cloudimg-amd64.img
```

No es una ISO de instalación tradicional. Es un disco ya preparado para arrancar y recibir configuración por cloud-init.

### 2. Crear la VM base vacía

Reserva un VMID que no estés usando. En el ejemplo uso `9000`, porque los IDs altos suelen ir bien para templates.

```bash
qm create 9000 \
  --name ubuntu-2404-cloudinit-template \
  --memory 2048 \
  --cores 2 \
  --net0 virtio,bridge=vmbr0 \
  --agent enabled=1
```

Qué hace cada opción:

| Opción | Qué configura |
| --- | --- |
| `9000` | VMID de la máquina base |
| `--name` | Nombre visible en Proxmox; luego Ansible lo usará como `clone` |
| `--memory 2048` | RAM inicial de la template; Ansible podrá cambiarla al clonar |
| `--cores 2` | CPU inicial de la template |
| `--net0 virtio,bridge=vmbr0` | Tarjeta de red rápida conectada a tu bridge principal |
| `--agent enabled=1` | Activa QEMU Guest Agent, útil para ver IPs y estado desde Proxmox |

### 3. Importar el disco cloud al storage principal

En tu caso tiene sentido usar el disco sano como storage principal. Si tu storage principal es `local-lvm`, importa ahí:

```bash
qm importdisk 9000 /var/lib/vz/template/iso/noble-server-cloudimg-amd64.img local-lvm
```

Esto crea un disco dentro del storage de Proxmox, pero aún no está conectado a la VM. Solo lo hemos importado.

### 4. Conectar el disco como SCSI

```bash
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0
qm set 9000 --boot c --bootdisk scsi0
```

`virtio-scsi-pci` suele ser una buena elección para Linux moderno. La segunda línea le dice a Proxmox que arranque desde ese disco.

Si tu disco importado no se llama exactamente `vm-9000-disk-0`, míralo con:

```bash
qm config 9000
```

### 5. Añadir la unidad cloud-init

```bash
qm set 9000 --ide2 local-lvm:cloudinit
```

Esta unidad no es el sistema operativo. Es el canal por el que Proxmox entrega a la VM los datos de usuario, SSH y red.

### 6. Configurar consola serie

Las imágenes cloud suelen funcionar mejor con consola serie:

```bash
qm set 9000 --serial0 socket --vga serial0
```

Esto evita pantallas negras o consolas raras cuando abras la VM desde Proxmox.

### 7. Dejar valores cloud-init base

Puedes poner valores por defecto en la template. Ansible los sobrescribirá al clonar, pero ayudan a validar que cloud-init funciona:

```bash
qm set 9000 --ciuser ubuntu
qm set 9000 --sshkeys ~/.ssh/id_ed25519.pub
qm set 9000 --ipconfig0 ip=dhcp
```

Si ejecutas esto desde el host Proxmox, asegúrate de que `~/.ssh/id_ed25519.pub` existe en Proxmox. Si tu clave está en tu portátil, copia el contenido y usa un archivo temporal.

### 8. Redimensionar el disco base si quieres

La imagen cloud puede venir con un disco pequeño. Puedes ampliarlo:

```bash
qm resize 9000 scsi0 20G
```

Ansible también define `proxmox_vm_disk`, pero conviene que la base no sea ridículamente pequeña.

### 9. Convertir la VM en template

Antes de convertir, no hace falta arrancarla. De hecho, una template limpia normalmente se deja sin primer arranque.

```compare
# CMD
qm template 9000
qm config 9000 | sed -n '1,20p'
# OUT
agent: enabled=1
boot: c
bootdisk: scsi0
ide2: local-lvm:vm-9000-cloudinit,media=cdrom
name: ubuntu-2404-cloudinit-template
...
template: 1
```

Cuando veas `template: 1`, Proxmox ya la tratará como base clonable.

### 10. Relación con las variables de Ansible

La template que acabas de crear debe coincidir con esta variable:

```yaml
proxmox_template: "ubuntu-2404-cloudinit-template"
```

Y el storage/bridge deben coincidir con tu Proxmox:

```yaml
proxmox_storage: "local-lvm"
proxmox_bridge: "vmbr0"
```

Si cambias el nombre de la template o usas otro storage, actualiza `group_vars/all.yml`.

## Cómo queda el playbook final

El rol del curso traduce variables a parámetros de la API de Proxmox:

```yaml
- name: Clonar VM desde template cloud-init
  community.general.proxmox_kvm:
    api_host: "{{ proxmox_api_host }}"
    api_user: "{{ proxmox_api_user }}"
    api_token_id: "{{ proxmox_api_token_id }}"
    api_token_secret: "{{ proxmox_api_token_secret }}"
    validate_certs: "{{ proxmox_validate_certs }}"
    node: "{{ proxmox_node }}"
    clone: "{{ proxmox_template }}"
    vmid: "{{ proxmox_vm_id }}"
    name: "{{ proxmox_vm_name }}"
    full: true
    storage: "{{ proxmox_storage }}"
    cores: "{{ proxmox_vm_cores }}"
    memory: "{{ proxmox_vm_memory }}"
    net:
      net0: "virtio,bridge={{ proxmox_bridge }}"
    ipconfig:
      ipconfig0: "{{ proxmox_vm_ipconfig }}"
    ciuser: "{{ proxmox_vm_ciuser }}"
    sshkeys: "{{ proxmox_vm_ssh_public_key }}"
    timeout: "{{ proxmox_vm_timeout }}"
    state: present
```

Después arranca la VM:

```yaml
- name: Arrancar VM
  community.general.proxmox_kvm:
    api_host: "{{ proxmox_api_host }}"
    api_user: "{{ proxmox_api_user }}"
    api_token_id: "{{ proxmox_api_token_id }}"
    api_token_secret: "{{ proxmox_api_token_secret }}"
    validate_certs: "{{ proxmox_validate_certs }}"
    node: "{{ proxmox_node }}"
    vmid: "{{ proxmox_vm_id }}"
    state: started
```

Fíjate en la idea: el playbook no sabe cómo se creó la template. Solo necesita que exista y que tenga cloud-init preparado.

## Ejecutar la clonación

Cuando la template exista y `group_vars/all.yml` esté listo:

```compare
# CMD
cd examples/ansible-homelab
ansible-playbook playbooks/proxmox-create-vm.yml
# OUT
PLAY [Crear VM cloud-init en Proxmox] *****************************************

TASK [proxmox_vm : Clonar VM desde template cloud-init] ***********************
changed: [localhost]

PLAY RECAP *******************************************************************
localhost : ok=<n> changed=1 unreachable=0 failed=0
```

Al terminar, entra en Proxmox y revisa la VM nueva:

| Campo | Qué deberías ver |
| --- | --- |
| Nombre | `vm-docker-01` o el nombre que pusiste |
| VMID | `9101` o el ID elegido |
| Hardware | CPU/RAM/red según variables |
| Cloud-Init | Usuario, SSH key e IP configurados |
| Estado | VM arrancada |

Si usas IP fija, también prueba SSH:

```compare
# CMD
ssh ubuntu@192.168.1.50 hostname
# OUT
vm-docker-01
```

## Flags y variantes

| Opción | Para qué sirve |
| --- | --- |
| `full: true` | Clon completo en vez de linked clone |
| `storage` | Storage destino de la VM |
| `net` | Configuración de red virtual |
| `ipconfig` | DHCP o IP fija con gateway |
| `timeout` | Tiempo máximo para operaciones Proxmox |
| `validate_certs` | Validación TLS del API |
| `qm template` | Convierte la VM base en plantilla |
| `qm importdisk` | Importa la imagen cloud al storage de Proxmox |

## Pruébalo tú

1. Descarga la imagen cloud de Ubuntu en Proxmox.
2. Crea la VM base con `qm create`.
3. Importa el disco con `qm importdisk`.
4. Conecta disco, cloud-init drive, boot order, consola serie y QEMU agent.
5. Define usuario, clave SSH e IP base.
6. Convierte la VM en template con `qm template`.
7. Ajusta `group_vars/all.yml` para que `proxmox_template` coincida.
8. Ejecuta `ansible-playbook playbooks/proxmox-create-vm.yml`.
9. Comprueba que la VM aparece y que puedes entrar por SSH.

## Errores comunes

- **Arrancar la template antes de convertirla**: puede consumir el primer arranque de cloud-init. Para una base limpia, conviértela sin usarla como VM normal.
- **Importar al storage equivocado**: si usas `local-lvm` en comandos y tu storage se llama distinto, ajusta todos los pasos.
- **No añadir `ide2: cloudinit`**: la VM clonada no recibirá usuario, SSH key ni red desde Proxmox.
- **Usar una clave SSH inexistente en Proxmox**: `qm set --sshkeys` lee el archivo desde el host Proxmox, no desde tu portátil.

> Idea clave: Ansible clona y parametriza; la template cloud-init es la fábrica que hace que cada VM arranque ya gestionable por SSH.
