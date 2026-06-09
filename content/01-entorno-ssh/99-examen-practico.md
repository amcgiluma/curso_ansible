---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---

# Examen práctico

Este examen comprueba que puedes preparar tu nodo de control y conectarte a una máquina Linux sin contraseña.

## Teoría

No hay automatización útil si el entorno base falla. Debes poder ejecutar Ansible desde WSL y entrar por SSH a una VM sin introducir contraseña.

## Manos a la obra

```compare
# CMD
ansible --version
ssh usuario@192.168.1.50 'hostname && uptime'
# OUT
ansible [core <version>]
...
<hostname-remoto>
 <hora> up <tiempo>,  <usuarios> user,  load average: <...>
```

## Flags y variantes

| Comando / flag | Para qué sirve |
| --- | --- |
| `ansible --version` | Verifica instalación |
| `ssh usuario@host` | Verifica conectividad |
| `ssh-keygen -R <host>` | Limpia una huella antigua |
| `chmod 600 <clave>` | Corrige permisos de clave privada |

## Pruébalo tú

1. Verifica Ansible desde WSL.
2. Verifica SSH sin contraseña contra una VM.
3. Anota usuario, IP y ruta de clave.
4. Explica en una frase por qué Ansible no necesita agente.

## Errores comunes

- **Probar Ansible antes de probar SSH**: depura primero SSH.
- **Usar una VM sin Python**: en distribuciones mínimas instala Python antes de usar módulos avanzados.

> Idea clave: si WSL, Ansible y SSH funcionan, ya tienes la base para automatizar el homelab.
