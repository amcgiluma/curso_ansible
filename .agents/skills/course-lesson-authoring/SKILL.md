---
name: course-lesson-authoring
description: "Redacta lecciones del Curso de Ansible para Homelab con formato consistente: teoría breve, comandos, salida esperada, ejercicios reproducibles y errores comunes. Usar siempre que se cree o edite contenido en content/."
category: education
---

# Course Lesson Authoring (Curso de Ansible)

Escribe lecciones en español para un curso práctico de Ansible orientado a homelab, Proxmox, cloud-init y Docker. El objetivo no es cubrir Ansible de forma enciclopédica, sino enseñar lo necesario para resolver automatizaciones reales.

## Idioma y tono

- Español correcto, práctico y directo.
- Mantener sin tildes slugs, rutas, comandos, variables e identificadores.
- No incluir secretos reales.
- Usar placeholders como `<version>`, `<ip>`, `<hash>`, `PROXMOX_API_TOKEN_SECRET`.
- Si una salida varía, marcarla con `<...>` o comentario.

## Estructura

Cada módulo vive en `content/NN-slug/` y contiene `module.json` y lecciones `.md`.

Cada lección debe tener frontmatter:

```markdown
---
title: "Título"
slug: "slug"
order: 1
summary: "Resumen breve."
---
```

## Secciones obligatorias

1. `# Título`.
2. Párrafo introductorio.
3. `## Teoría`.
4. `## Manos a la obra`.
5. `## Flags y variantes`.
6. `## Pruébalo tú`.
7. `## Errores comunes`.
8. `> Idea clave:`.

## Bloques de código

Preferir bloques `compare`:

```compare
# CMD
ansible --version
# OUT
ansible [core <version>]
...
```

Usar `bash` para comandos sin salida, `output` para salidas sueltas, `yaml` para playbooks, `ini` para inventarios y `json` para JSON.

## Ejemplos reproducibles

Cuando una lección requiera ficheros, referenciar `examples/ansible-homelab/` o incluir bloques copiables. No obligar al alumno a inventar estructura.

## Examen práctico

Cada módulo termina con `99-examen-practico.md`, frontmatter:

```markdown
---
title: "Examen práctico"
slug: "examen-practico"
order: 99
summary: "Retos prácticos para comprobar que puedes aplicar lo aprendido en el módulo."
---
```

La verificación es manual. No añadir autocorrección.
