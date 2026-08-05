# Graph Report - /home/josemurillo/Documents/jobs_/BreteAI  (2026-08-05)

## Corpus Check
- Corpus is ~3,614 words - fits in a single context window. You may not need a graph.

## Summary
- 25 nodes · 36 edges · 4 communities
- Extraction: 89% EXTRACTED · 11% INFERRED · 0% AMBIGUOUS · INFERRED: 4 edges (avg confidence: 0.82)
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- IA local, matching y correo
- Portal web, API y datos
- Fuentes de ofertas y roadmap
- Infraestructura y despliegue

## God Nodes (most connected - your core abstractions)
1. `Scraping de ofertas` - 8 edges
2. `BreteAI-Backend` - 6 edges
3. `BreteAI (proyecto)` - 5 edges
4. `BreteAI-Frontend` - 5 edges
5. `PostgreSQL` - 5 edges
6. `BreteAI-Infra` - 4 edges
7. `IA local (Ollama)` - 4 edges
8. `APIs oficiales/públicas de empleo` - 3 edges
9. `Backend FastAPI (Python)` - 3 edges
10. `Roadmap v1 (fases)` - 3 edges

## Surprising Connections (you probably didn't know these)
- `Submódulos git` --references--> `BreteAI (proyecto)`  [EXTRACTED]
  docs/SETUP-GIT.md → README.md
- `BreteAI (proyecto)` --references--> `BreteAI-Backend`  [EXTRACTED]
  README.md → BreteAI-Backend/README.md
- `BreteAI (proyecto)` --references--> `BreteAI-Frontend`  [EXTRACTED]
  README.md → BreteAI-Frontend/README.md
- `BreteAI (proyecto)` --references--> `Roadmap v1 (fases)`  [EXTRACTED]
  README.md → docs/ROADMAP.md
- `BreteAI-Backend` --implements--> `Backend FastAPI (Python)`  [EXTRACTED]
  BreteAI-Backend/README.md → docs/requirements.md

## Hyperedges (group relationships)
- **Pipeline de ofertas (scraping→DB→IA→correo)** — docs_requirements_scraping, docs_requirements_postgresql, docs_requirements_ia_local, docs_requirements_gmail_smtp [INFERRED 0.85]
- **Componentes del proyecto (submódulos)** — breteai_backend_readme_backend, breteai_frontend_readme_frontend, breteai_infra_readme_infra [EXTRACTED 0.90]

## Communities (4 total, 0 thin omitted)

### Community 0 - "IA local, matching y correo"
Cohesion: 0.33
Nodes (7): BreteAI-Backend, Deduplicación de ofertas, Correo Gmail SMTP, IA local (Ollama), Score de match 0-100, perfil.toon (config del candidato), Scheduler 4x/día

### Community 1 - "Portal web, API y datos"
Cohesion: 0.38
Nodes (7): BreteAI-Frontend, Autenticación segura, Dashboard de analítica, Backend FastAPI (Python), Frontend Next.js, PostgreSQL, Gestión tipo ticket

### Community 2 - "Fuentes de ofertas y roadmap"
Cohesion: 0.47
Nodes (6): Contexto del proyecto (CLAUDE.md), ATS Greenhouse/Lever/Ashby, APIs oficiales/públicas de empleo, Scraping de ofertas, Fuentes Costa Rica (fase posterior), Roadmap v1 (fases)

### Community 3 - "Infraestructura y despliegue"
Cohesion: 0.50
Nodes (5): BreteAI-Infra, CI/CD GitHub Actions + Tailscale, Docker (red breteai-net), Submódulos git, BreteAI (proyecto)

## Knowledge Gaps
- **4 isolated node(s):** `perfil.toon (config del candidato)`, `Autenticación segura`, `Fuentes Costa Rica (fase posterior)`, `Submódulos git`
  These have ≤1 connection - possible missing edges or undocumented components.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `Scraping de ofertas` connect `Fuentes de ofertas y roadmap` to `IA local, matching y correo`, `Portal web, API y datos`?**
  _High betweenness centrality (0.389) - this node is a cross-community bridge._
- **Why does `BreteAI-Backend` connect `IA local, matching y correo` to `Portal web, API y datos`, `Fuentes de ofertas y roadmap`, `Infraestructura y despliegue`?**
  _High betweenness centrality (0.335) - this node is a cross-community bridge._
- **Why does `BreteAI (proyecto)` connect `Infraestructura y despliegue` to `IA local, matching y correo`, `Portal web, API y datos`, `Fuentes de ofertas y roadmap`?**
  _High betweenness centrality (0.291) - this node is a cross-community bridge._
- **What connects `perfil.toon (config del candidato)`, `Autenticación segura`, `Fuentes Costa Rica (fase posterior)` to the rest of the system?**
  _4 weakly-connected nodes found - possible documentation gaps or missing edges._