# Graph Report - .  (2026-08-11)

## Corpus Check
- 4 files · ~6,650 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 79 nodes · 90 edges · 20 communities (8 shown, 12 thin omitted)
- Extraction: 83% EXTRACTED · 17% INFERRED · 0% AMBIGUOUS · INFERRED: 15 edges (avg confidence: 0.92)
- Token cost: 49,260 input · 0 output

## Community Hubs (Navigation)
- Capa de IA y Perfil
- Backend, DB e Infra
- Pipeline por Etapas (Staging)
- Portal, Correo y Dashboard
- Docker Compose de Postgres
- Alcance y Backlog
- Pruebas, CI/CD y Graphify
- Entorno Local de Desarrollo
- Multi-repo y Docker
- Testcontainers
- Fuente Adzuna
- Fuente Arbeitnow
- ATS Ashby
- ATS Greenhouse
- Fuente Himalayas
- Fuente Jobicy
- ATS Lever
- Fuente RemoteOK
- Fuente Remotive
- Acceso Tailscale

## God Nodes (most connected - your core abstractions)
1. `Fase 1 — Scraping + DB` - 10 edges
2. `Tabla ofertas (estructura fija)` - 8 edges
3. `v1 (MVP funcional)` - 7 edges
4. `Fase 2 — IA` - 6 edges
5. `Servicio db (Postgres 16-alpine)` - 6 edges
6. `Tabla ofertas_raw (staging/cola)` - 5 edges
7. `Entorno local de desarrollo (PC del usuario)` - 5 edges
8. `BreteAI (proyecto)` - 4 edges
9. `Principio: solo fuentes legales` - 4 edges
10. `Worker IA (Ollama)` - 4 edges

## Surprising Connections (you probably didn't know these)
- `Reglas de negocio clave` --semantically_similar_to--> `Principio: solo fuentes legales`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `Integración graphify` --semantically_similar_to--> `Graphify (requisito esencial)`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `perfil.toon (configuración del perfil)` --semantically_similar_to--> `perfil.toon (perfil del candidato)`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `Estructura multi-repo con submódulos` --semantically_similar_to--> `Infraestructura Docker (breteai-net)`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `BreteAI (proyecto)` --references--> `BreteAI-Frontend`  [EXTRACTED]
  README.md → BreteAI-Frontend/README.md

## Hyperedges (group relationships)
- **Setup de PostgreSQL vía Docker + aplicación y verificación del schema** — breteai_infra_docker_compose_servicio_db, claude_postgresql_solo_via_docker, docs_roadmap_esquema_db_aplicado_verificado [INFERRED 0.80]
- **Configuración del entorno local de desarrollo (Docker + DBeaver)** — claude_entorno_local, claude_postgresql_solo_via_docker, claude_dbeaver_community, claude_docker_systemctl [INFERRED 0.75]
- **Conectores de fuentes legales (Fase 1)** — docs_roadmap_fase_1_scraping_db, docs_requirements_remotive, docs_requirements_remoteok, docs_requirements_arbeitnow, docs_requirements_jobicy, docs_requirements_himalayas, docs_requirements_adzuna, docs_requirements_greenhouse, docs_requirements_lever, docs_requirements_ashby [EXTRACTED 1.00]
- **Flujo del pipeline por etapas: scheduler → raw → worker IA → ofertas** — docs_design_scheduler, docs_design_tabla_corridas, docs_design_tabla_ofertas_raw, docs_design_worker_ia, docs_design_tabla_ofertas [EXTRACTED 1.00]
- **Herramientas de la estrategia de pruebas** — docs_design_estrategia_de_pruebas, docs_requirements_bruno, docs_design_mailhog, docs_design_testcontainers [EXTRACTED 1.00]
- **Componentes del proyecto (submódulos)** — breteai_backend_readme_backend, breteai_frontend_readme_frontend, breteai_infra_readme_infra [EXTRACTED 0.90]

## Communities (20 total, 12 thin omitted)

### Community 0 - "Capa de IA y Perfil"
Cohesion: 0.17
Nodes (15): perfil.toon (configuración del perfil), Stack CERRADO v1, Riesgo #1: salida del LLM malformada, Tabla correos, Tabla feedback_ia, Tabla oferta_historial, Tabla ofertas (estructura fija), Worker IA (Ollama) (+7 more)

### Community 1 - "Backend, DB e Infra"
Cohesion: 0.22
Nodes (9): BreteAI-Backend, BreteAI-Infra, Backup diario pg_dump ~17:00, Esquema de DB (DDL borrador), Python + FastAPI, Gmail SMTP con App Password, PostgreSQL, Submódulos git (+1 more)

### Community 2 - "Pipeline por Etapas (Staging)"
Cohesion: 0.25
Nodes (9): BreteAI (contexto del proyecto), Idempotencia (UNIQUE fuente + id_externo), Modelo canónico de oferta, Pipeline por etapas (diseño), Scheduler (APScheduler, 4x/día), Tabla corridas, Tabla ofertas_raw (staging/cola), Pipeline por etapas (staging) (+1 more)

### Community 3 - "Portal, Correo y Dashboard"
Cohesion: 0.29
Nodes (8): BreteAI-Frontend, MailHog (SMTP falso para tests), Dashboard de analítica, Next.js (portal web), Fase 3 — Correo, Fase 4 — Portal (Next.js), Fase 5 — Dashboard, v1 (MVP funcional)

### Community 4 - "Docker Compose de Postgres"
Cohesion: 0.48
Nodes (7): Healthcheck del servicio db (pg_isready), Montaje ./db/init → /docker-entrypoint-initdb.d, Red Docker breteai-net, Servicio db (Postgres 16-alpine), Volumen db_data, PostgreSQL solo vía Docker (nunca paquete nativo), Fase 1 — Scraping + DB

### Community 5 - "Alcance y Backlog"
Cohesion: 0.33
Nodes (7): Reglas de negocio clave, Apify (respaldo opcional), Estados de oferta, Principio: solo fuentes legales, ATS Workday (pospuesto), Fases posteriores (backlog), Fuera de alcance

### Community 6 - "Pruebas, CI/CD y Graphify"
Cohesion: 0.33
Nodes (6): Integración graphify, Estrategia de pruebas por flujo, Bruno (cliente API), CI/CD con GitHub Actions, Graphify (requisito esencial), Transversal — CI/CD, smoke, Bruno, Graphify

### Community 7 - "Entorno Local de Desarrollo"
Cohesion: 0.50
Nodes (5): DBeaver Community (cliente DB nativo por pacman), DBeaver requiere Java 21+, Activación de Docker (systemctl + grupo docker), Entorno local de desarrollo (PC del usuario), Esquema de DB aplicado y verificado en DBeaver (2026-08-11)

## Knowledge Gaps
- **25 isolated node(s):** `Submódulos git`, `Integración graphify`, `perfil.toon (configuración del perfil)`, `Estructura multi-repo con submódulos`, `Remotive (feed abierto)` (+20 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **12 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Fase 1 — Scraping + DB` connect `Docker Compose de Postgres` to `Backend, DB e Infra`, `Pipeline por Etapas (Staging)`, `Portal, Correo y Dashboard`, `Pruebas, CI/CD y Graphify`, `Entorno Local de Desarrollo`?**
  _High betweenness centrality (0.317) - this node is a cross-community bridge._
- **Why does `v1 (MVP funcional)` connect `Portal, Correo y Dashboard` to `Capa de IA y Perfil`, `Pipeline por Etapas (Staging)`, `Docker Compose de Postgres`, `Pruebas, CI/CD y Graphify`?**
  _High betweenness centrality (0.308) - this node is a cross-community bridge._
- **Why does `Tabla ofertas (estructura fija)` connect `Capa de IA y Perfil` to `Pipeline por Etapas (Staging)`, `Alcance y Backlog`?**
  _High betweenness centrality (0.199) - this node is a cross-community bridge._
- **What connects `Submódulos git`, `Integración graphify`, `perfil.toon (configuración del perfil)` to the rest of the system?**
  _25 weakly-connected nodes found - possible documentation gaps or missing edges._