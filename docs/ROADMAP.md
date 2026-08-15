# ROADMAP - BreteAI

Control de alcance: qué entra en la **v1** y qué queda para **fases posteriores**.
Marcar `[x]` al completar.

---

## v1 (MVP funcional)

### Fase 1 — Scraping + DB
- [x] Repo `breteai-infra`: docker-compose con PostgreSQL + red `breteai-net` + volúmenes.
- [x] Esquema de DB según `design.md` §3: `corridas`, `ofertas_raw` (staging/cola), `ofertas` + tablas de gestión. Aplicado y verificado en DBeaver (2026-08-11).
- [ ] Backup diario automático ~17:00 (+ probar restore una vez).
- [x] Conectores de fuentes legales — implementados y verificados contra las APIs reales (2026-08-12):
  - [x] Remotive
  - [x] RemoteOK
  - [x] Arbeitnow
  - [x] Jobicy
  - [x] Himalayas
  - [x] Adzuna (código listo; pendiente probar con API key real — sin cuenta creada aún)
  - [x] ATS Greenhouse (verificado en vivo contra board público `gitlab`)
  - [x] ATS Lever (código listo; sin board activo disponible para probar en vivo)
  - [x] ATS Ashby (verificado en vivo contra board público `notion`)
- [x] Modelo canónico de oferta (mapeo por conector, `design.md` §2).
- [x] Scheduler 4x/día (05:00, 11:00, 16:00, 22:00 CR) + registro en tabla `corridas` — código y config verificados; sin confirmar disparo real de un cron (requiere esperar horario).
- [x] Idempotencia (`fuente + id_externo`) — verificada con test de integración (corrida duplicada no duplica filas).
- [ ] Deduplicación semántica adicional (empresa + puesto, entre fuentes distintas) — la idempotencia de arriba es por fuente, esto es dedup cross-fuente, queda pendiente.
- [x] Alerta si un conector se rompe: `registrar_fallo()` + tests de contrato (`pytest -m contract`) como detección; falta el canal de notificación real (Fase 3, correo).
- [x] **Tests:** 35 tests (21 unit+integration en CI, 14 de contrato aparte). Fixtures reales de 7/9 fuentes (`tests/fixtures/`), Postgres real vía testcontainers. Pendiente: colección Bruno.

### Fase 2 — IA
- [ ] Ollama en el server con Qwen 2.5 7B (Q4_K_M) — comparar vs Llama 3.2.
- [ ] Pipeline: resumen + extracción de campos + score 0-100 contra `perfil.toon`.
- [ ] Enriquecimiento "info de segunda mano" (ej: proveedor vs empresa real).
- [ ] Dedup semántico (embeddings).
- [ ] Estimación de salario (referencia web) cuando no viene en la oferta.
- [ ] Feedback simple: correcciones ajustan criterios del prompt (tabla `feedback_ia`).
- [ ] Validación estricta de la salida del LLM (Pydantic) + reintentos + raw en `error` reprocesable.
- [ ] **Tests:** unitarios de prompt/validación; worker con mock de Ollama; golden set de scores (solo server, `design.md` §4-B).

### Fase 3 — Correo
- [ ] Gmail SMTP con App Password.
- [ ] Correo con top 5-10 (no aplicadas) en cards + link al portal.
- [ ] Registro de correos enviados (vista "último correo").
- [ ] **Tests:** render del template (unitario) + envío contra MailHog (`design.md` §4-C). Gmail real nunca en tests.

### Fase 4 — Portal (Next.js)
- [ ] Login seguro (single user, password hasheada).
- [ ] Vistas: visor, no aplicadas, aplicadas, último correo.
- [ ] Gestión tipo ticket: estados, etiquetas, comentarios, archivos.
- [ ] Filtros: empresa, modalidad, ubicación, estado, score.
- [ ] Responsivo (PC-first).
- [ ] **Tests:** endpoints con TestClient de FastAPI (auth incluida); colección Bruno de la API propia (contrato para el frontend, corre en CI); Vitest en componentes (`design.md` §4-D).

### Fase 5 — Dashboard
- [ ] Aplicadas vs no aplicadas.
- [ ] Estados y tasa de respuesta.
- [ ] Modalidad (presencial/virtual/mixto).
- [ ] Ubicación (CR vs fuera + países).
- [ ] Navegación con filtros.
- [ ] **Tests:** queries de agregación con seed conocido → números exactos (`design.md` §4-E).

### Transversal
- [ ] **Checklist de seguridad al desplegar en el server** (en la PC de dev se aceptó relajado a propósito, auditoría 2026-08-12):
  - [ ] Password real de Postgres (no `changeme`) + recrear volumen.
  - [ ] Bindear Postgres solo a localhost/red Docker: `"127.0.0.1:5432:5432"` (no `0.0.0.0`).
  - [ ] `chmod 600` a los `.env` del server.
- [ ] CI/CD: GitHub Actions (runner gratis) → auto-deploy SSH sobre Tailscale.
- [ ] Pipeline CI: `ruff` → unit tests → build imágenes → smoke test → deploy.
- [ ] Smoke test: `docker compose up` + healthchecks (`/health`, `pg_isready`, `ollama list`).
- [ ] Workflow scheduled aparte para tests de contrato de APIs externas (no bloquea deploy).
- [ ] Bruno como cliente API (open source; colecciones versionadas en git, `bru run` en CI).
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
