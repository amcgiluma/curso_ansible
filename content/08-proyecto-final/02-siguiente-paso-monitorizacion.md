---
title: "Siguiente paso: monitorización"
slug: "siguiente-paso-monitorizacion"
order: 2
summary: "Dejar preparada la base para Prometheus, Grafana y exporters sin alargar este curso."
---

# Siguiente paso: monitorización

Este curso no despliega un stack completo de monitorización. Sí deja la base correcta: VMs reproducibles, Docker instalado y Ansible ordenado.

## Teoría

Para monitorización de homelab, el siguiente curso o proyecto puede usar:

| Componente | Uso |
| --- | --- |
| Prometheus | Recoger métricas |
| Grafana | Visualizar dashboards |
| node-exporter | Métricas del host |
| cAdvisor | Métricas de contenedores |
| Alertmanager | Alertas |

La VM docker-ready que has creado puede ser el host donde desplegar ese stack con Compose o con otro rol Ansible.

## Manos a la obra

Valida que la base está lista:

```compare
# CMD
cd examples/ansible-homelab
ansible docker_hosts -m command -a "docker --version"
ansible docker_hosts -m command -a "docker compose version"
ansible docker_hosts -m command -a "hostname"
# OUT
vm-docker-01 | CHANGED | rc=0 >>
Docker version <version>, build <hash>
vm-docker-01 | CHANGED | rc=0 >>
Docker Compose version v<version>
vm-docker-01 | CHANGED | rc=0 >>
<hostname>
```

## Flags y variantes

| Camino | Cuándo elegirlo |
| --- | --- |
| Rol Ansible para Compose | Si quieres gestionar ficheros y servicios desde Ansible |
| Compose manual inicial | Si quieres probar rápido Prometheus/Grafana |
| VM separada para monitorización | Si quieres aislar observabilidad del resto |
| Red dedicada | Si vas a segmentar servicios del homelab |

## Pruébalo tú

1. Crea una VM docker-ready con el playbook final.
2. Comprueba Docker y Compose.
3. Decide si monitorización irá en esa VM o en otra.
4. Esboza un `compose.yml` con Prometheus y Grafana.
5. Anota qué variables convertirías en `group_vars`.

## Errores comunes

- **Meter monitorización antes de tener provisioning estable**: primero crea hosts repetibles.
- **Usar una sola VM para todo sin pensar recursos**: empieza simple, pero documenta CPU/RAM.
- **No versionar configuración**: Ansible y Compose brillan cuando guardas los ficheros.

> Idea clave: el curso termina cuando puedes crear capacidad bajo demanda; la monitorización será una capa encima de esa base.
