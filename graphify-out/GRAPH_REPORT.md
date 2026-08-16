# Graph Report - BreteAI  (2026-08-15)

## Corpus Check
- 85 files · ~50,487 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 100 nodes · 110 edges · 20 communities (8 shown, 12 thin omitted)
- Extraction: 86% EXTRACTED · 14% INFERRED · 0% AMBIGUOUS · INFERRED: 15 edges (avg confidence: 0.92)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `75bbdde3`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- Capa de IA y Perfil
- Backend, DB e Infra
- Pipeline por Etapas (Staging)
- Fase 2 — IA ✅ IMPLEMENTADA
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
2. `Guía de Implementación - BreteAI` - 9 edges
3. `Tabla ofertas (estructura fija)` - 8 edges
4. `v1 (MVP funcional)` - 7 edges
5. `Fase 2 — IA` - 6 edges
6. `Servicio db (Postgres 16-alpine)` - 6 edges
7. `Fase 2 — IA ✅ IMPLEMENTADA` - 5 edges
8. `Fase 3 — Correo 🔨 A IMPLEMENTAR` - 5 edges
9. `Tabla ofertas_raw (staging/cola)` - 5 edges
10. `Entorno local de desarrollo (PC del usuario)` - 5 edges

## Surprising Connections (you probably didn't know these)
- `Reglas de negocio clave` --semantically_similar_to--> `Principio: solo fuentes legales`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `Integración graphify` --semantically_similar_to--> `Graphify (requisito esencial)`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `perfil.toon (configuración del perfil)` --semantically_similar_to--> `perfil.toon (perfil del candidato)`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `Estructura multi-repo con submódulos` --semantically_similar_to--> `Infraestructura Docker (breteai-net)`  [INFERRED] [semantically similar]
  CLAUDE.md → docs/requirements.md
- `BreteAI (contexto del proyecto)` --references--> `v1 (MVP funcional)`  [EXTRACTED]
  CLAUDE.md → docs/ROADMAP.md

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
Cohesion: 0.13
Nodes (17): BreteAI-Backend, BreteAI-Frontend, BreteAI-Infra, Backup diario pg_dump ~17:00, Esquema de DB (DDL borrador), MailHog (SMTP falso para tests), Dashboard de analítica, Python + FastAPI (+9 more)

### Community 2 - "Pipeline por Etapas (Staging)"
Cohesion: 0.25
Nodes (9): BreteAI (contexto del proyecto), Idempotencia (UNIQUE fuente + id_externo), Modelo canónico de oferta, Pipeline por etapas (diseño), Scheduler (APScheduler, 4x/día), Tabla corridas, Tabla ofertas_raw (staging/cola), Pipeline por etapas (staging) (+1 more)

### Community 3 - "Fase 2 — IA ✅ IMPLEMENTADA"
Cohesion: 0.40
Nodes (5): Decisiones que quedaron abiertas a propósito, Fase 2 — IA ✅ IMPLEMENTADA, No hacer en esta fase (para no salirse de alcance), Orden de implementación, Tests de esta fase (ver `design.md` §4-B)

### Community 4 - "Docker Compose de Postgres"
Cohesion: 0.26
Nodes (12): Healthcheck del servicio db (pg_isready), Montaje ./db/init → /docker-entrypoint-initdb.d, Red Docker breteai-net, Servicio db (Postgres 16-alpine), Volumen db_data, DBeaver Community (cliente DB nativo por pacman), DBeaver requiere Java 21+, Activación de Docker (systemctl + grupo docker) (+4 more)

### Community 5 - "Alcance y Backlog"
Cohesion: 0.33
Nodes (7): Reglas de negocio clave, Apify (respaldo opcional), Estados de oferta, Principio: solo fuentes legales, ATS Workday (pospuesto), Fases posteriores (backlog), Fuera de alcance

### Community 6 - "Pruebas, CI/CD y Graphify"
Cohesion: 0.33
Nodes (6): Integración graphify, Estrategia de pruebas por flujo, Bruno (cliente API), CI/CD con GitHub Actions, Graphify (requisito esencial), Transversal — CI/CD, smoke, Bruno, Graphify

### Community 7 - "Entorno Local de Desarrollo"
Cohesion: 0.12
Nodes (15): Cómo usar esta guía, Decisiones abiertas (resolver al implementar y dejarlo escrito en el código), Estructura de módulos (árbol completo), Fase 1 — Scraping + DB ✅ IMPLEMENTADA, Fase 3 — Correo 🔨 A IMPLEMENTAR, Fase 4 — Portal (resumen), Fase 5 — Dashboard (resumen), Guía de Implementación - BreteAI (+7 more)

## Knowledge Gaps
- **41 isolated node(s):** `Cómo usar esta guía`, `Orden de implementación (`BreteAI-Backend/app/`)`, `Tests de esta fase (ver `design.md` §4-A)`, `No hacer en esta fase (para no salirse de alcance)`, `Orden de implementación` (+36 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **12 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Fase 1 — Scraping + DB` connect `Docker Compose de Postgres` to `Backend, DB e Infra`, `Pipeline por Etapas (Staging)`, `Pruebas, CI/CD y Graphify`?**
  _High betweenness centrality (0.196) - this node is a cross-community bridge._
- **Why does `v1 (MVP funcional)` connect `Backend, DB e Infra` to `Capa de IA y Perfil`, `Pipeline por Etapas (Staging)`, `Docker Compose de Postgres`, `Pruebas, CI/CD y Graphify`?**
  _High betweenness centrality (0.191) - this node is a cross-community bridge._
- **Why does `Tabla ofertas (estructura fija)` connect `Capa de IA y Perfil` to `Pipeline por Etapas (Staging)`, `Alcance y Backlog`?**
  _High betweenness centrality (0.123) - this node is a cross-community bridge._
- **What connects `Cómo usar esta guía`, `Orden de implementación (`BreteAI-Backend/app/`)`, `Tests de esta fase (ver `design.md` §4-A)` to the rest of the system?**
  _41 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Backend, DB e Infra` be split into smaller, more focused modules?**
  _Cohesion score 0.1323529411764706 - nodes in this community are weakly interconnected._
- **Should `Entorno Local de Desarrollo` be split into smaller, more focused modules?**
  _Cohesion score 0.125 - nodes in this community are weakly interconnected._