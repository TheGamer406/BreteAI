# BreteAI 🇨🇷

**Buscador de trabajos automatizado con integración de IA.**

BreteAI busca ofertas de trabajo en fuentes legales varias veces al día, las analiza con un modelo de IA **local** (resumen + puntuación de match 0-100 contra tu perfil), las guarda en una base de datos y te notifica por correo con las mejores. Incluye un portal web para revisarlas y gestionarlas como tickets, más un dashboard de analítica.

> *"Brete"* = trabajo, en Costa Rica. Proyecto **open source** y **100% self-hosted** (costo ~$0).

---

## ¿Por qué?

Buscar trabajo es repetitivo: revisar múltiples portales, leer descripciones largas, descartar las que no encajan. BreteAI automatiza eso: centraliza las ofertas, las resume, estima qué tan bien encajás y prioriza dónde aplicar — sin depender de servicios de pago.

## Cómo funciona

```
Fuentes (APIs legales)  ──►  Scraper  ──►  PostgreSQL  ──►  IA local (Ollama)
                                                                   │
        Correo (top ofertas) ◄── Backend (FastAPI) ◄──────────────┘
                                       │
                                 Portal (Next.js): tickets · filtros · dashboard
```

## Arquitectura (repos)

Este es el **repo padre**. Los componentes viven en submódulos:

| Repo | Descripción | Stack |
|------|-------------|-------|
| [`BreteAI-Backend`](./BreteAI-Backend) | Scraping + pipeline de IA + API REST | Python, FastAPI, Ollama |
| [`BreteAI-Frontend`](./BreteAI-Frontend) | Portal web (tickets, filtros, dashboard) | Next.js |
| [`BreteAI-Infra`](./BreteAI-Infra) | Docker Compose, PostgreSQL, red y volúmenes | Docker, PostgreSQL |

## Stack

- **Backend/IA:** Python + FastAPI, Ollama (modelo local, ej. Qwen 2.5 7B) en GPU.
- **Base de datos:** PostgreSQL.
- **Frontend:** Next.js.
- **Correo:** Gmail SMTP (App Password).
- **Infra:** Docker (servicios separados en una red), self-hosted, acceso vía Tailscale.
- **CI/CD:** GitHub Actions (build/test) + auto-deploy por SSH sobre Tailscale.

## Fuentes de ofertas (legales)

APIs oficiales/públicas: Remotive, RemoteOK, Arbeitnow, Jobicy, Himalayas, WeWorkRemotely, Adzuna, y ATS Greenhouse/Lever/Ashby (empresas grandes). *No* se hace scraping de LinkedIn/Indeed.

## Configuración

El perfil del candidato vive en un archivo TOON. Para usarlo:

```bash
cp config/perfil.example.toon resources/perfil.toon
# editá resources/perfil.toon con tus datos (queda fuera de git)
```

## Documentación

- [`docs/requirements.md`](./docs/requirements.md) — requerimientos completos.
- [`docs/ROADMAP.md`](./docs/ROADMAP.md) — plan por fases (v1 y backlog).

## Estado

🚧 En construcción. Ver el [roadmap](./docs/ROADMAP.md).

## Licencia

MIT.
