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

### Fase 3 — Correo ✅ (funcional 2026-08-16)

> Implementada y verificada de punta a punta contra **MailHog real** (testcontainers en los
> tests; contenedor manual para la verificación con datos reales) con las 58 ofertas ya
> analizadas en Fase 2. Gmail real nunca se tocó — la decisión de `requirements.md` §10 sigue
> siendo el runtime de producción, pendiente solo de la App Password real en el server.

- [x] **Selección de ofertas** (`correo/seleccion.py`): no aplicadas (`nueva`+`vista`), con score ≥ 40, sin repetir lo enviado en las últimas 24h (una oferta buena reaparece al día siguiente si sigue sin aplicarse — ni "nunca más" ni spam 4x/día).
- [x] Plantilla HTML en cards (`correo/plantilla.py`): puesto, score, empresa, modalidad, salario (marca "(estimado)" cuando `salario_estimado=True`), `score_razon` + link al portal. CSS inline, sin imágenes externas, aviso si `similar_a` no es nulo.
- [x] Gmail SMTP con App Password (`correo/cliente.py`) + config (`SMTP_*`, `MAIL_FROM`, `PORTAL_BASE_URL`). Reusa `app/common/retry.py`.
- [x] Orquestación y registro de correos enviados (`correo/envio.py` + tabla `correos`). Registrado **después** del envío exitoso — verificado que un SMTP caído no deja fila en `correos`.
- [x] No manda correo si no hay ofertas elegibles (`requirements.md` §4.3) — verificado.
- [x] Enganchado al scheduler (paso 3, después del worker de IA, en `scheduler/jobs.py`).
- [x] Canal real de la alerta de conector roto (`alerts/connector_health.py::fuentes_con_fallas_recurrentes`): fuentes que fallan en 3 corridas seguidas se agregan como sección al pie del correo (no un correo aparte, para no sumar ruido).
- [x] MailHog vía **testcontainers** en los tests (no en `docker-compose.yml` de Infra — es solo para tests, no debe correr en producción; decisión distinta a la de Ollama en Fase 2, que sí necesita estar siempre arriba).
- [x] **Tests:** 24 nuevos (12 unit de plantilla + 8 integración de selección con Postgres real + 4 integración de envío contra MailHog real). Total del proyecto: 89 tests (69 en CI por defecto + 14 de contrato + 6 golden).

**Corrida real verificada:** correo enviado con las 10 mejores ofertas ya analizadas (score 85→45), headers y multipart correctos, registrado en `correos`.

### Fase 4 — Portal (API + Next.js)

> Esqueleto creado (2026-08-16): `app/api/` en el backend, `BreteAI-Frontend/` completo.
> **Dirección de diseño ya decidida:** denso y silencioso, claro + oscuro. El sistema está
> escrito y completo en `BreteAI-Frontend/styles/tokens.css` (no es un TODO).
> Orden de implementación y contexto: **`docs/GUIA-IMPLEMENTACION.md` § Fase 4.**

**Backend — API REST**
- [ ] Hash de password (argon2/bcrypt) + JWT (`api/seguridad.py`). **Empezar por acá.** Decisión abierta: hash en `.env` vs tabla `usuarios` nueva.
- [ ] Contrato de la API (`api/schemas.py`) — invocar la skill `api-design`. Paginación desde el día uno (el histórico crece).
- [ ] Login con cookie `httpOnly` (no `localStorage`), rate limiting, mismo mensaje de error para usuario/password incorrectos.
- [ ] Endpoints de ofertas: listado con filtros, detalle, cambio de estado (**debe escribir en `oferta_historial`**, o el dashboard de Fase 5 nace sin datos), etiquetas, comentarios, adjuntos.
- [ ] Endpoint de feedback → escribe en `feedback_ia` y cierra el ciclo con Fase 2 (hoy la tabla existe pero nada la llena).
- [ ] Vista "último correo" (`routers/correos.py`).
- [ ] Montar routers y CORS en `app/main.py`.

**Frontend — portal Next.js**
- [ ] Scaffold Next.js (App Router + TypeScript). Decidir si entra Tailwind y, si entra, que consuma los tokens existentes.
- [ ] Cliente de API (`lib/api.ts`, único punto de fetch) y tipos (`lib/tipos.ts`, evaluar generarlos desde el OpenAPI).
- [ ] Shell + toggle de tema con script anti-flash + skip link.
- [ ] Tabla de ofertas densa (filas, no cards; score a la izquierda) + filtros en la URL.
- [ ] Detalle tipo ticket: estado, etiquetas, comentarios, adjuntos, feedback a la IA.
- [ ] Login y vista "último correo".
- [ ] Responsivo: PC primero; bajo ~48rem la tabla colapsa a 2 líneas por oferta (sin scroll horizontal).

**Calidad**
- [ ] **Accesibilidad WCAG 2.2 AA**: axe en la suite + recorrido manual con teclado + contraste verificado en los DOS temas.
- [ ] **Tests:** `test_api_auth.py` (incluye el test que recorre todos los endpoints verificando 401) y `test_api_ofertas.py` con Postgres real; Vitest + Testing Library en el portal; colección Bruno de la API propia en CI.

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

**Origen: Fase 3**
- [ ] **Probar envío real por Gmail SMTP** (todo lo verificado hasta ahora fue contra MailHog). Requiere acción manual del usuario:
  1. Activar 2FA en la cuenta de Google.
  2. Generar una **App Password** en myaccount.google.com → Seguridad → Contraseñas de aplicaciones (16 caracteres).
  3. Poner `SMTP_USER`, `SMTP_PASSWORD`, `MAIL_FROM`, `MAIL_TO` en `BreteAI-Backend/.env` (nunca commitear).
  4. Enviar sin pasar `client=` para que use el cliente Gmail por defecto (TLS en 587).
- [ ] **Sincronizar `.env` local cuando se agregue config nueva** — el `.env` no se actualiza solo desde `.env.example` y se desincroniza en silencio (pasó el 2026-08-16: faltaban 12 variables de Fase 2 y 3, y los tests golden fallaban apuntando al puerto de Ollama en vez del de LM Studio). Chequeo rápido:
  ```bash
  comm -23 <(grep -oP '^[A-Z_]+(?==)' .env.example|sort) <(grep -oP '^[A-Z_]+(?==)' .env|sort)
  ```
  Evaluar automatizarlo (validación al arrancar la app, o un test que compare ambos archivos).

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
