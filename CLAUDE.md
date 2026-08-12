# BreteAI — Buscador de trabajos automatizado con integración de IA

## Contexto para futuras sesiones

Sistema **open source, 100% local**, que busca ofertas de trabajo en fuentes legales 4 veces al día, las analiza con IA local (resumen + score de match 0-100 contra el perfil), las guarda en PostgreSQL y notifica por correo. Portal web (Next.js) con gestión tipo ticket + dashboard de analítica. Doble propósito: **uso real** (mejor trabajo) + **portafolio** (automatización con IA y buenas prácticas). Configurable para cualquiera vía `perfil.toon`.

## Estructura del repo

- Repo **padre** `BreteAI` con **submódulos**: `BreteAI-Backend`, `BreteAI-Frontend`, `BreteAI-Infra`.
- `docs/` — documentación pública. `config/` — plantillas (`perfil.example.toon`).
- `resources/` — **NO versionado** (gitignored): datos personales, CVs, perfil real, notas de planeación.

## Documentos fuente (leer antes de trabajar)

- **`docs/requirements.md`** — requerimientos completos. **FUENTE DE VERDAD.**
- **`docs/ROADMAP.md`** — qué va en v1 y qué queda para después (checklist por fases).
- **`docs/design.md`** — diseño técnico: pipeline por etapas (raw → cola → IA → ofertas), modelo canónico, DDL borrador, estrategia de tests por flujo (Bruno/pytest/testcontainers/MailHog).
- **`docs/GUIA-IMPLEMENTACION.md`** — índice por fase para implementar código: qué leer, qué archivos tocar y en qué orden. `BreteAI-Backend/app/` y `tests/` ya tienen el esqueleto de Fase 1 (cada archivo con docstring explicando qué implementar ahí). Antes de escribir código de pipeline/conectores, leer este archivo primero.
- **`config/perfil.example.toon`** — plantilla del perfil (pública, sin datos reales).
- **`resources/perfil.toon`** — perfil REAL que consume la IA (privado, TOON).
- **`resources/preguntas.md`** — 76 preguntas respondidas (historial de decisiones).
- **`resources/idea.md`** — planteamiento original.
- **`resources/cv/`** — CVs en PDF (ES/EN), privados.

## Perfil del usuario

Estudiante Ing. Desarrollo de Software (UCenfotec, grad. mar-2027), **Junior**. Intereses: Backend, Data Science, Data Analyst. Skills: Python, JS, Java, C#, Node.js, T-SQL, React, Tailwind. Inglés B2+. Costa Rica; abierto a presencial/remoto/mixto + reubicación internacional. **Datos personales y meta salarial en `resources/perfil.toon` (privado, no versionado).**

## Stack CERRADO (v1)

- **Backend/pipeline:** Python + FastAPI. API REST.
- **Base de datos:** PostgreSQL (mismo server que la GPU). Backup diario ~17:00.
- **IA:** 100% local con Ollama. Modelo recomendado Qwen 2.5 7B Q4_K_M (tiene Llama 3.2). Latencia aceptada. **Sin API de pago** (Claude Pro $20 NO incluye API).
- **Frontend:** Next.js, hospedado en el mismo server (red Docker). Login seguro single-user (password hasheada bcrypt/argon2).
- **Correo:** Gmail SMTP con App Password → destinatario configurable vía `MAIL_TO` en `.env` (no versionar el correo personal en el repo). Top 5-10 no aplicadas en cards con link al portal.
- **Infra:** servicios separados en red Docker `breteai-net`, multi-repo (`breteai-backend`, `breteai-frontend`, `breteai-infra`), `.env` + GitHub Secrets, volúmenes persistentes.
- **CI/CD:** GitHub Actions (runners gratis) build/test → auto-deploy SSH sobre Tailscale (`docker compose pull && up -d`). Flujo DevOps estándar (el usuario quiere aprenderlo).
- **Graphify:** parte del pipeline; relaciones entre y dentro de repos.

## Reglas de negocio clave

- **Solo fuentes legales:** APIs oficiales/públicas (Remotive, RemoteOK, Arbeitnow, Jobicy, Himalayas, WeWorkRemotely, Adzuna) + ATS Greenhouse/Lever/Ashby (IBM/Amazon/Microsoft, etc.). NO scraping de LinkedIn/Indeed.
- **Corridas:** 05:00, 11:00, 16:00, 22:00 (CR). Si hay resultados → correo.
- **Estados de oferta:** nueva, vista, aplicada, enProceso, enEspera, respondida, rechazada.
- **Guardar todo el histórico** (análisis de datos).
- **Salario ausente:** buscar referencia web, si no `null`/"no especificado".
- **Feedback de IA:** correcciones ajustan criterios del prompt (simple, NO re-entrenamiento).
- **Dedup:** por empresa + puesto (y semántico con embeddings); en portal marcar "similar a".

## Roadmap

1. Scraping + DB → 2. IA → 3. Correo → 4. Portal → 5. Dashboard. (Detalle y backlog en `ROADMAP.md`.)

## Reglas del proyecto

- **Todo local.** Gratis/open source, tope duro ~$10/mes.
- Priorizar buenas prácticas (es portafolio).
- Fuentes CR (Computrabajo/Brete.cr/elempleo) y monitor GPU (`alertServer.md`) = fases posteriores.

## Entorno local de desarrollo (PC del usuario)

- **PostgreSQL: solo vía Docker, nunca paquete nativo.** `BreteAI-Infra/docker-compose.yml` levanta `postgres:16-alpine` en la red `breteai-net`, con `.env` (copiado de `.env.example`, no versionado) y `db/init/001_schema.sql` (el DDL de `design.md` §3, se aplica solo la primera vez que se crea el volumen). Para reaplicar el schema desde cero: `docker compose down -v && docker compose up -d` (borra los datos).
- **Cliente de DB: DBeaver Community, instalado nativo por pacman** (`sudo pacman -S dbeaver`, repo `extra`). **No usar la versión Flatpak** — venía preinstalada en el sistema, quedaba desactualizada y generaba confusión con dos instalaciones; se desinstaló.
- **DBeaver requiere Java 21+.** Si tira "incompatible JVM": `archlinux-java status` para ver JDKs disponibles y `sudo archlinux-java set java-26-openjdk` (o la versión ≥21 disponible) para cambiar el default del sistema.
- Docker se activa con `sudo systemctl enable --now docker`; el usuario ya está en el grupo `docker` (no requiere sudo para `docker compose`).

## graphify

Este proyecto tiene un grafo de conocimiento en `graphify-out/graph.json` (usar el intérprete `~/.local/bin/graphify`, versión nueva — NO el `/usr/bin/graphify` viejo).

- **Antes de responder** preguntas sobre la arquitectura/código, consultar el grafo: `graphify query "<pregunta>"`. Si `graphify-out/graph.json` existe, responder desde ahí en vez de re-explorar todo.
- **Después de cambios de código** relevantes, actualizar el grafo: `/graphify --update` (reconstruye AST + semántico). El grafo cubre el repo padre + submódulos.
- **Privacidad:** graphify respeta `.gitignore`, así que `resources/` (datos personales) queda fuera del grafo. Mantener así.
- **Hook opcional:** `~/.local/bin/graphify hook install` deja un post-commit que reconstruye el grafo (solo AST/código) tras cada commit.
