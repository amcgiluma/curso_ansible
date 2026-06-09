# Curso de Ansible para Homelab

Curso self-paced y práctico para aprender lo básico de Ansible con un objetivo claro: crear máquinas virtuales en Proxmox automáticamente, dejarlas accesibles por SSH e instalar Docker para futuros despliegues de homelab.

La plataforma está construida con React + Vite + Tailwind en el frontend y FastAPI en el backend. Las lecciones son Markdown y cada una muestra comandos junto a salidas esperadas para comparar manualmente.

## Qué vas a aprender

| # | Módulo | Contenido |
| --- | --- | --- |
| 01 | Entorno, WSL y SSH | Preparar WSL Ubuntu, instalar Ansible y conectar por clave SSH |
| 02 | Inventarios y ad-hoc | Hosts, grupos, variables de conexión y comandos rápidos |
| 03 | Playbooks e idempotencia | YAML, tareas, módulos, check mode, diff y recap |
| 04 | Variables, templates y handlers | `group_vars`, facts, Jinja2 y reinicios controlados |
| 05 | Roles y estructura | Proyecto Ansible ordenado, tags y colecciones |
| 06 | Docker con Ansible | Instalar Docker Engine y Compose plugin en VMs Linux |
| 07 | Proxmox y cloud-init | API tokens, colección `community.general` y clonación de plantillas |
| 08 | Proyecto final | Crear VM en Proxmox, esperar SSH, instalar Docker y validar |
| 09 | Linux, CachyOS y Semaphore | Instalar Ansible en Linux, compartir playbooks por Git y ejecutarlos desde Semaphore UI |

El ejemplo central vive en `examples/ansible-homelab/` e incluye inventario, variables, playbooks y roles mínimos.

También hay un ejemplo en `examples/semaphore/` para levantar Semaphore UI con Docker Compose y conectar el repositorio de playbooks desde Windows/WSL, CachyOS o cualquier otro nodo de control.

## Requisitos del alumno

- Windows con WSL Ubuntu, o una máquina Linux equivalente.
- Una VM Linux de pruebas para los primeros módulos.
- Proxmox VE ya instalado para los módulos 7 y 8.
- Una plantilla cloud-init Ubuntu/Debian preparada en Proxmox.
- Conocimientos básicos de terminal, SSH y YAML.

## Arranque rápido de la plataforma

### Con Docker Compose

```bash
docker compose -f docker-compose.dev.yml up --build
```

- Frontend: http://localhost:5173
- Backend: http://localhost:8000/api/modules

Producción local:

```bash
docker compose up --build -d
```

- App: http://localhost:8080

### Local sin contenedores

Backend:

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Frontend:

```bash
cd frontend
npm install
npm run dev
```

## Estructura

```text
.
├── AGENTS.md
├── README.md
├── backend/
├── content/
├── examples/
│   └── ansible-homelab/
├── frontend/
├── scripts/
├── docker-compose.dev.yml
└── docker-compose.yml
```

## Cómo funcionan las lecciones

Cada lección sigue el mismo patrón: teoría breve, comandos, salida esperada, flags y variantes, práctica reproducible, errores comunes e idea clave.

Los bloques `compare` muestran comando y salida esperada lado a lado. La verificación es manual: ejecutas el comando en tu entorno y comparas con la salida esperada. Los valores variables aparecen como `<...>`.

## Ejemplo Ansible

```bash
cd examples/ansible-homelab
cp inventory.ini.example inventory.ini
cp group_vars/all.yml.example group_vars/all.yml
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/ping.yml
```

No guardes tokens reales de Proxmox en ficheros versionados.

## Ejemplo Semaphore

```bash
cd examples/semaphore
cp .env.example .env
docker compose up -d
```

Después abre `http://localhost:3000` y conecta el repositorio `https://github.com/amcgiluma/curso_ansible.git` como fuente de playbooks.

## Deploy estático en Vercel

La plataforma puede generar un `content.json` estático:

```bash
node scripts/build-content.mjs
cd frontend
npm run build
```

`frontend/public/content.json` está ignorado por Git.

## Stack técnico

- Frontend: React 19, Vite, TypeScript, Tailwind CSS v4.
- Backend: FastAPI, Uvicorn, Pydantic, PyYAML.
- Infra local: Docker Compose para servir la plataforma.
- Curso: Ansible, SSH, Proxmox API, cloud-init y Docker Engine.
- Orquestación visual opcional: Semaphore UI para lanzar playbooks desde navegador.

## Licencia

MIT. Úsalo y adáptalo para aprender Ansible y automatizar tu homelab.
