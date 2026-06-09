# AGENTS.md

Este fichero es lo primero que debe leer cualquier agente antes de trabajar en este repo.

## Qué es este proyecto

Plataforma web de un curso práctico de Ansible para homelab. El objetivo educativo es que Juanma aprenda Ansible básico de forma resolutiva y termine pudiendo crear máquinas virtuales en Proxmox automáticamente, con Docker instalado y listas para futuros despliegues de monitorización.

La verificación del curso es manual: cada lección muestra comandos y salidas esperadas. El alumno ejecuta los comandos en su entorno y compara resultados. El backend no ejecuta Ansible, Docker ni Proxmox.

## Arquitectura

- `frontend/`: SPA React + Vite + TypeScript + Tailwind. Renderiza Markdown, bloques de código y progreso en `localStorage`.
- `backend/`: API FastAPI de solo lectura. Escanea `content/` y expone módulos/lecciones.
- `content/`: fuente de verdad del curso. Una carpeta por módulo con `module.json` y lecciones Markdown.
- `examples/`: proyecto Ansible reproducible. El ejemplo central es `examples/ansible-homelab/`.

```text
content/NN-slug/module.json
content/NN-slug/NN-leccion.md
```

## Comandos

Desarrollo con Docker Compose:

```bash
docker compose -f docker-compose.dev.yml up --build
```

Producción local:

```bash
docker compose up --build -d
```

Backend local:

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Frontend local:

```bash
cd frontend
npm install
npm run dev
```

Validaciones:

```bash
node scripts/build-content.mjs
npm --prefix frontend run build
npm --prefix frontend run lint
docker compose config
docker compose -f docker-compose.dev.yml config
```

## Convenciones de contenido

Usar la skill local `.agents/skills/course-lesson-authoring/SKILL.md`.

Cada lección debe incluir:

- Frontmatter `title`, `slug`, `order`, `summary`.
- Título H1.
- `## Teoría`.
- `## Manos a la obra`.
- Al menos un bloque `compare`.
- `## Flags y variantes`.
- `## Pruébalo tú`.
- `## Errores comunes`.
- Cierre con `> Idea clave:`.

Cada módulo termina con `99-examen-practico.md`.

## Alcance técnico del curso

- Nodo de control recomendado: WSL Ubuntu.
- Transporte: SSH con clave pública.
- Proxmox: API token + cloud-init + colección `community.general`.
- Docker: instalación práctica de Docker Engine y Compose plugin en Ubuntu/Debian.
- Monitorización: solo preparación de base; no desplegar Prometheus/Grafana en este curso.

## Gotchas

- No incluir tokens reales de Proxmox en ejemplos ni docs.
- Usar placeholders claros para IPs, VMIDs, secretos y claves.
- No convertir el curso en una enciclopedia de Ansible: priorizar resolución práctica.
- Mantener API backend de solo lectura.
- Si se edita `content/`, validar con `node scripts/build-content.mjs`.
