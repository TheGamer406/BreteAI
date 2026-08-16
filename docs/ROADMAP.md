# ROADMAP - BreteAI

Control de alcance: qué entra en la **v1** y qué queda para **fases posteriores**.
Marcar `[x]` al completar.

---

## v1 (MVP funcional)

### Fase 1 — Scraping + DB ✅ (funcional 2026-08-12)

> Cerrada: el pipeline trae ofertas reales y las persiste en `ofertas_raw`.
> Lo que quedó pendiente NO bloquea Fase 2 → movido a **Fase Fix** (abajo).

- [x] Repo `breteai-infra`: docker-compose con PostgreSQL + red `breteai-net` + volúmenes.
- [x] Esquema de DB según `design.md` §3: `corridas`, `ofertas_raw` (staging/cola), `ofertas` + tablas de gestión. Aplicado y verificado en DBeaver (2026-08-11).
- [x] Conectores de fuentes legales — implementados y verificados contra las APIs reales (2026-08-12):
  - [x] Remotive
  - [x] RemoteOK
  - [x] Arbeitnow
  - [x] Jobicy
  - [x] Himalayas
  - [x] Adzuna (código listo; probar con API key real → Fase Fix)
  - [x] ATS Greenhouse (verificado en vivo contra board público `gitlab`)
  - [x] ATS Lever (código listo; probar con board activo → Fase Fix)
  - [x] ATS Ashby (verificado en vivo contra board público `notion`)
- [x] Modelo canónico de oferta (mapeo por conector, `design.md` §2).
- [x] Scheduler 4x/día (05:00, 11:00, 16:00, 22:00 CR) + registro en tabla `corridas` — código y config verificados; confirmar disparo real → Fase Fix.
- [x] Idempotencia (`fuente + id_externo`) — verificada con test de integración (corrida duplicada no duplica filas).
- [x] Alerta si un conector se rompe: `registrar_fallo()` + tests de contrato (`pytest -m contract`) como detección; canal de notificación real = Fase 3 (correo).
- [x] **Tests:** 35 tests (21 unit+integration en CI, 14 de contrato aparte). Fixtures reales de 7/9 fuentes (`tests/fixtures/`), Postgres real vía testcontainers.

**Corrida real verificada:** 411 ofertas de 5 fuentes guardadas en `ofertas_raw`, sin duplicados entre corridas.

### Fase 2 — IA ✅ (funcional 2026-08-15)

> Implementada y verificada contra un LLM real: **Qwen2.5-7B-Instruct (Q4_K_M)** corriendo en
> LM Studio en esta PC de desarrollo (`localhost:1234`, API OpenAI-compatible). El cliente
> (`app/ai/client.py`) habla esa misma API que expone Ollama, así que sirve sin cambios para el
> runtime de producción documentado en `requirements.md` §5.2 — solo cambia `OLLAMA_HOST`/`OLLAMA_MODEL`.

- [x] **Validación estricta de la salida del LLM** (`ai/schemas.py`) + reintentos + raw en `error` reprocesable — Riesgo #1 de `design.md` §5.
- [x] Servicio de IA: agregado al `docker-compose` de Infra (`ollama/ollama` + GPU passthrough) para el server. En esta PC se usó LM Studio en vez de instalar Ollama (mismo protocolo, ya tenía el runtime).
- [x] Cliente contra API OpenAI-compatible (`ai/client.py`) + config (`OLLAMA_HOST`, `OLLAMA_MODEL`, `OLLAMA_MODELO_EMBEDDINGS`, timeout largo).
- [x] Carga de `perfil.toon` (`ai/perfil.py`, parser TOON propio) y armado de prompts (`ai/prompts.py`, limpia HTML y trunca descripciones largas).
- [x] Worker de la cola (`pipeline/worker.py`): `ofertas_raw` pendiente → IA → `ofertas`. Re-mapea el payload crudo a canónico vía `connectors/registro.py` (nuevo, extraído de `runner.py` para no duplicar la lista de conectores).
- [x] Pipeline de análisis: resumen + extracción de campos + score 0-100 contra el perfil — verificado discriminando correctamente (junior backend match=85-100, senior/stack ajeno=0-10, no técnico=15-40).
- [x] Enriquecimiento "info de segunda mano" (`empresa_real`) — sale del mismo análisis.
- [x] Estimación de salario cuando no viene (`ai/salario.py`) — promedia ofertas ya guardadas con salario real (no LLM, evita alucinación); `salario_estimado=True` cuando aplica.
- [x] Dedup semántico con embeddings (`ai/embeddings.py`) → llena `ofertas.similar_a`. Requirió agregar columna `embedding JSONB` a `ofertas` (DDL + ORM). Verificado: mismo título+empresa en fuente distinta → detecta; título similar pero empresa distinta → no falso-positivo.
- [x] Feedback simple: correcciones ajustan criterios del prompt (`feedback/criterios.py` + tabla `feedback_ia`), enganchado al worker.
- [x] Worker enganchado al scheduler (`scheduler/jobs.py`: scraping → reprocesar errores → worker, cada corrida).
- [x] **Tests:** 65 tests totales — 11 unit de schemas (fixtures reales del LLM) + 7 unit de prompts + 6 integración de worker (Postgres real, LLM mockeado) en el run default; 6 golden calibrados contra el LLM real (marker `golden`); + los 14 de contrato de Fase 1.

**Corrida real verificada:** 58 ofertas analizadas de punta a punta (411 en cola), 0 errores, ~4s/oferta, scores coherentes en todo el rango.

**Refactor de Fase 1 durante esta fase:** lógica de reintentos extraída a `app/common/retry.py` (compartida entre `connectors/base.py` y `ai/client.py`, DRY); tipo de `Oferta.requisitos`/`beneficios` corregido a `list[str]` (antes `List[dict]`, inconsistente con el schema real).

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

### Fase Fix — deuda técnica acumulada

> **Qué es:** pendientes que quedaron de fases ya cerradas y que **no bloquean** avanzar.
> Se visitan de forma agrupada dentro de un par de fases, no ahora.
> Al cerrar cada fase, lo que quede sin terminar y no bloquee → se mueve acá con su origen.

**Origen: Fase 1**
- [ ] Backup diario automático ~17:00 (`pg_dump` en contenedor + volumen) + **probar restore al menos una vez**.
- [ ] Dedup **cross-fuente** (misma vacante publicada en 2 fuentes distintas → marcar "similar a"). Ojo: distinto de la idempotencia `fuente+id_externo` que ya funciona, y distinto del dedup semántico con embeddings de Fase 2.
- [ ] Probar Adzuna con API key real (crear cuenta gratis, 1k llamadas/mes) y definir la lista de países a consultar según `perfil.toon` (hoy hardcodeada a `["us","gb"]` en `adzuna.py`).
- [ ] Probar Lever contra un board con postings activos (los que se probaron estaban vacíos o dan 404) y reemplazar `tests/fixtures/lever.json` por una captura real.
- [ ] Confirmar disparo real del scheduler (esperar un horario o forzar un trigger de prueba) — hoy solo está verificada la config.
- [ ] Mapear board tokens/slugs de ATS a nombres legibles de empresa (hoy `empresa` guarda el token, ver TODOs en `greenhouse.py`/`lever.py`/`ashby.py`).
- [ ] Colección **Bruno** por fuente (cliente API open source, colecciones versionadas en git).

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
