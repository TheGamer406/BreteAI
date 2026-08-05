# ROADMAP - BreteAI

Control de alcance: qué entra en la **v1** y qué queda para **fases posteriores**.
Marcar `[x]` al completar.

---

## v1 (MVP funcional)

### Fase 1 — Scraping + DB
- [ ] Repo `breteai-infra`: docker-compose con PostgreSQL + red `breteai-net` + volúmenes.
- [ ] Esquema de DB (tabla `ofertas` con estados, JSON crudo, fuente, score, etc.).
- [ ] Backup diario automático ~17:00.
- [ ] Conectores de fuentes legales:
  - [ ] Remotive
  - [ ] RemoteOK
  - [ ] Arbeitnow
  - [ ] Jobicy
  - [ ] Himalayas
  - [ ] Adzuna (API key gratis, 1k/mes)
  - [ ] ATS Greenhouse (board tokens de empresas objetivo)
  - [ ] ATS Lever
  - [ ] ATS Ashby
- [ ] Scheduler 4x/día (05:00, 11:00, 16:00, 22:00 CR).
- [ ] Deduplicación básica (empresa + puesto).
- [ ] Alerta si un conector se rompe.

### Fase 2 — IA
- [ ] Ollama en el server con Qwen 2.5 7B (Q4_K_M) — comparar vs Llama 3.2.
- [ ] Pipeline: resumen + extracción de campos + score 0-100 contra `perfil.toon`.
- [ ] Enriquecimiento "info de segunda mano" (ej: proveedor vs empresa real).
- [ ] Dedup semántico (embeddings).
- [ ] Estimación de salario (referencia web) cuando no viene en la oferta.
- [ ] Feedback simple: correcciones ajustan criterios del prompt.

### Fase 3 — Correo
- [ ] Gmail SMTP con App Password.
- [ ] Correo con top 5-10 (no aplicadas) en cards + link al portal.

### Fase 4 — Portal (Next.js)
- [ ] Login seguro (single user, password hasheada).
- [ ] Vistas: visor, no aplicadas, aplicadas, último correo.
- [ ] Gestión tipo ticket: estados, etiquetas, comentarios, archivos.
- [ ] Filtros: empresa, modalidad, ubicación, estado, score.
- [ ] Responsivo (PC-first).

### Fase 5 — Dashboard
- [ ] Aplicadas vs no aplicadas.
- [ ] Estados y tasa de respuesta.
- [ ] Modalidad (presencial/virtual/mixto).
- [ ] Ubicación (CR vs fuera + países).
- [ ] Navegación con filtros.

### Transversal
- [ ] CI/CD: GitHub Actions (runner gratis) → auto-deploy SSH sobre Tailscale.
- [ ] Graphify integrado al pipeline (relaciones entre y dentro de repos).
- [ ] README por repo + documentación.

---

## Fases posteriores (backlog)

- [ ] Fuentes Costa Rica por scraping HTML: Computrabajo CR, elempleo, Brete.cr.
- [ ] Workday (requiere render JS).
- [ ] Monitor del server GPU con ping/alerta → ver `alertServer.md` (crear).
- [ ] Post en LinkedIn explicando cómo funciona.
- [ ] Multiusuario / que otra persona lo despliegue fácil.
- [ ] (Opcional) Apify como respaldo puntual (plan gratis $5 crédito/mes).

---

## Fuera de alcance (por ahora)
- Scraping directo de LinkedIn/Indeed (contra sus términos).
- Re-entrenamiento real del modelo de IA.
- Tablero Jira completo (solo gestión tipo ticket).
- API de pago para IA (se usa Ollama local, costo $0).
