# Semaphore UI para el homelab

Ejemplo mínimo para levantar Semaphore UI y conectarlo al repositorio de playbooks del curso.

## Uso rápido

```bash
cp .env.example .env
# Edita .env antes de exponerlo en red
docker compose up -d
```

Abre `http://localhost:3000` y crea:

1. Un proyecto `homelab`.
2. Un repositorio apuntando a `https://github.com/amcgiluma/curso_ansible.git`.
3. Un inventario con tus hosts reales.
4. Una clave SSH en Key Store.
5. Una plantilla para `examples/ansible-homelab/playbooks/ping.yml`.

## Seguridad

No subas `.env`, claves SSH, inventarios con secretos ni tokens de Proxmox. Este ejemplo usa SQLite para empezar rápido; para uso más serio, cambia a PostgreSQL o MySQL siguiendo la documentación oficial de Semaphore UI.
