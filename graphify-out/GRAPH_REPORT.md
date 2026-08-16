# Graph Report - .  (2026-08-16)

## Corpus Check
- 86 files · ~51,314 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 497 nodes · 1041 edges · 45 communities (37 shown, 8 thin omitted)
- Extraction: 95% EXTRACTED · 5% INFERRED · 0% AMBIGUOUS · INFERRED: 51 edges (avg confidence: 0.66)
- Token cost: 81,651 input · 0 output

## Community Hubs (Navigation)
- Conectores de fuentes (Fase 1)
- Modelos ORM y alertas de conectores
- Perfil TOON y parser
- Cliente LLM y dedup semántico
- Analyzer y schemas de IA
- Cliente SMTP y envío de correo
- Plantilla HTML del correo
- Cliente IA, retry compartido y config
- Guía de implementación
- Registro de conectores y sesión DB
- Docker Compose: servicio Ollama
- Entrypoint FastAPI y scheduler
- Docker Compose: servicio Postgres + DBeaver
- Modelo canónico y tablas de gestión
- Settings de ATS (boards/tokens)
- Cliente IA: embeddings y reintentos
- Reglas de negocio y decisiones de correo
- README del Backend
- Tests de contrato de conectores
- Fixtures de respuestas de Ollama
- Requirements: stack e IA
- Submódulos del monorepo
- Dependency de sesión FastAPI
- Paquete de conectores
- Graphify en el proyecto
- Backup diario (pendiente)
- DDL borrador de Infra
- Estrategia de pruebas (design.md)
- MailHog para tests
- Pipeline por etapas (diseño)
- Testcontainers en tests

## God Nodes (most connected - your core abstractions)
1. `OfertaCanonica` - 48 edges
2. `BaseConnector` - 28 edges
3. `get_settings()` - 21 edges
4. `Modalidad` - 21 edges
5. `render_correo()` - 19 edges
6. `parsear_respuesta_llm()` - 18 edges
7. `enviar_correo_ofertas()` - 18 edges
8. `Oferta` - 18 edges
9. `construir_prompt_analisis()` - 17 edges
10. `Perfil` - 15 edges

## Surprising Connections (you probably didn't know these)
- `Dependencias Fase 1 pineadas (requirements.txt)` --conceptually_related_to--> `Fase 1 — Scraping + DB ✅ IMPLEMENTADA`  [INFERRED]
  BreteAI-Backend/requirements.txt → docs/GUIA-IMPLEMENTACION.md
- `BreteAI-Backend README` --references--> `Fase 1 — Scraping + DB ✅ IMPLEMENTADA`  [EXTRACTED]
  BreteAI-Backend/README.md → docs/GUIA-IMPLEMENTACION.md
- `Fixtures de conectores (una respuesta real por fuente)` --references--> `Fase 1 — Scraping + DB ✅ IMPLEMENTADA`  [EXTRACTED]
  BreteAI-Backend/tests/fixtures/README.md → docs/GUIA-IMPLEMENTACION.md
- `Fase 1 — Scraping + DB` --implements--> `Volumen db_data`  [EXTRACTED]
  docs/ROADMAP.md → BreteAI-Infra/docker-compose.yml
- `Stack CERRADO (v1)` --references--> `red breteai-net`  [EXTRACTED]
  CLAUDE.md → BreteAI-Infra/docker-compose.yml

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **Cliente API OpenAI-compatible compartido entre Ollama (server) y LM Studio (dev)** — breteai_infra_docker_compose_servicio_ollama, claude_lm_studio_headless, docs_guia_implementacion_fase_2_ia [INFERRED 0.85]
- **Decisiones que definen qué entra en el correo (no spam, no repetido, score mínimo)** — docs_guia_implementacion_fase_3_correo_decision_reenvio_24h, docs_guia_implementacion_fase_3_correo_decision_estados_no_aplicada, docs_guia_implementacion_fase_3_correo_decision_score_minimo [EXTRACTED 1.00]
- **Suite de tests acumulada del proyecto (89 tests: Fase 1 + Fase 2 + Fase 3)** — docs_roadmap_fase_1_scraping_db, docs_roadmap_fase_2_ia, docs_roadmap_fase_3_correo [EXTRACTED 1.00]
- **Flujo del pipeline por etapas: scheduler → raw → worker IA → ofertas** — docs_design_scheduler, docs_design_tabla_corridas, docs_design_tabla_ofertas_raw, docs_design_worker_ia, docs_design_tabla_ofertas [EXTRACTED 1.00]
- **Herramientas de la estrategia de pruebas** — docs_design_estrategia_de_pruebas, docs_design_mailhog, docs_design_testcontainers [EXTRACTED 1.00]
- **Componentes del proyecto (submódulos)** — breteai_frontend_readme_frontend, breteai_infra_readme_infra [EXTRACTED 0.90]

## Communities (45 total, 8 thin omitted)

### Community 0 - "Conectores de fuentes (Fase 1)"
Cohesion: 0.07
Nodes (44): ABC, AdzunaConnector, Conector Adzuna — requiere credenciales (gratis, 1k llamadas/mes). Endpoint:…, ArbeitnowConnector, Conector Arbeitnow — API pública, sin autenticación., AshbyConnector, Conector Ashby (ATS) — API pública, sin autenticación. Endpoint: GET…, BaseConnector (+36 more)

### Community 1 - "Modelos ORM y alertas de conectores"
Cohesion: 0.06
Nodes (47): Any, fuentes_con_fallas_recurrentes(), Session, Alerta cuando un conector falla (docs/requirements.md §4.5).…, Fuentes presentes en `fuentes_error` de las últimas `corridas_para_alerta`…, Adjunto, Base, Comentario (+39 more)

### Community 2 - "Perfil TOON y parser"
Cohesion: 0.11
Nodes (42): Candidato, cargar_perfil(), _coerce_scalar(), CriteriosMatch, Experiencia, Formacion, Idioma, _parse_line() (+34 more)

### Community 3 - "Cliente LLM y dedup semántico"
Cohesion: 0.08
Nodes (36): OllamaClient, Ping liviano para el healthcheck del smoke test de CI (docs/design.md §4,…, Cliente contra un servidor local con API OpenAI-compatible (Ollama o LM Studio,…, buscar_similar(), calcular_embedding(), Session, Dedup semántico con embeddings (requirements.md §4.6 y §5.1). Detecta la misma…, Embedding de un texto CORTO y estable -- NO la descripción completa (cara de… (+28 more)

### Community 4 - "Analyzer y schemas de IA"
Cohesion: 0.09
Nodes (34): analizar_oferta(), Orquesta el análisis de UNA oferta: prompt -> LLM -> validación. Template…, Analiza una oferta contra el perfil y devuelve un AnalisisIA validado. Si la…, AnalisisIA, _extraer_json_candidato(), parsear_respuesta_llm(), BaseModel, Exception (+26 more)

### Community 5 - "Cliente SMTP y envío de correo"
Cohesion: 0.11
Nodes (31): ClienteSMTP, Ping liviano para el healthcheck del smoke test de CI. No manda nada, no usa…, enviar_correo_ofertas(), Session, Selecciona ofertas, renderiza, envía y registra -- en ese orden. El registro en…, _ids_recientemente_enviados(), marcar_como_enviadas(), Session (+23 more)

### Community 6 - "Plantilla HTML del correo"
Cohesion: 0.21
Nodes (22): _card_html(), _formatear_salario(), Render del HTML/texto del correo. Único lugar del proyecto con markup de correo…, Devuelve (asunto, cuerpo_html). Las cards se renderizan en el orden recibido --…, render_correo(), render_texto_plano(), _seccion_alertas_html(), Oferta (+14 more)

### Community 7 - "Cliente IA, retry compartido y config"
Cohesion: 0.18
Nodes (6): Cliente HTTP hacia el LLM local. Única puerta de salida al modelo — ningún otro…, Reintentos con backoff exponencial, compartido entre los conectores de Fase 1…, get_settings(), Singleton cacheado de Settings — se lee .env una sola vez., Cliente SMTP. Única puerta de salida hacia el servidor de correo -- ningún otro…, Orquesta el envío. Etapa 3 del pipeline (design.md §1: `ofertas` -> correo top…

### Community 8 - "Guía de implementación"
Cohesion: 0.15
Nodes (14): Cómo usar esta guía, Decisiones que quedaron abiertas a propósito, Decisiones tomadas (quedaron escritas en el código, ver docstrings para el detalle), Estructura de módulos (árbol completo), Fase 2 — IA ✅ IMPLEMENTADA, Fase 3 — Correo ✅ IMPLEMENTADA, Fase 4 — Portal (resumen), Fase 5 — Dashboard (resumen) (+6 more)

### Community 9 - "Registro de conectores y sesión DB"
Cohesion: 0.24
Nodes (11): Exception, Registra que un conector falló en una corrida. No confundir con…, registrar_fallo(), conectores_para_corrida(), Instancia todos los conectores registrados, en orden estable., db_session(), Context manager para usar fuera de FastAPI (ej: scheduler, scripts)., ejecutar_corrida() (+3 more)

### Community 10 - "Docker Compose: servicio Ollama"
Cohesion: 0.21
Nodes (14): BreteAI-Infra docker-compose.yml, En dev sin GPU dedicada a Docker: correr Ollama/LM Studio nativo en el host y apuntar OLLAMA_HOST, GPU passthrough nvidia (deploy.resources.reservations.devices, nvidia-container-toolkit), servicio ollama (ollama/ollama, Fase 2, requirements.md §5.2), volumen ollama_data (persistencia de modelos descargados), IA local Fase 2: LM Studio en dev vs Ollama en server (mismo cliente API OpenAI-compatible), Levantar LM Studio headless (dev, sustituto de Ollama), Fase 2 — IA ✅ IMPLEMENTADA (verificada contra LM Studio, LLM real) (+6 more)

### Community 11 - "Entrypoint FastAPI y scheduler"
Cohesion: 0.19
Nodes (12): Main application entrypoint for BreteAI. This is where we initialize and run…, Initialize the scheduler when the app starts., Stop the scheduler when the app shuts down., root(), shutdown_event(), startup_event(), detener_scheduler(), iniciar_scheduler() (+4 more)

### Community 12 - "Docker Compose: servicio Postgres + DBeaver"
Cohesion: 0.26
Nodes (12): Healthcheck del servicio db (pg_isready), Montaje ./db/init → /docker-entrypoint-initdb.d, red breteai-net, Servicio db (Postgres 16-alpine), Volumen db_data, DBeaver Community (cliente DB nativo por pacman), DBeaver requiere Java 21+, Activación de Docker (systemctl + grupo docker) (+4 more)

### Community 13 - "Modelo canónico y tablas de gestión"
Cohesion: 0.20
Nodes (11): Idempotencia (UNIQUE fuente + id_externo), Modelo canónico de oferta, Riesgo #1: salida del LLM malformada, Scheduler (APScheduler, 4x/día), Tabla correos, Tabla corridas, Tabla feedback_ia, Tabla oferta_historial (+3 more)

### Community 14 - "Settings de ATS (boards/tokens)"
Cohesion: 0.22
Nodes (6): BaseSettings, Retorna lista de board tokens de Greenhouse., Configuración central del proyecto vía variables de entorno., Retorna lista de slugs de Lever., Retorna lista de board names de Ashby., Settings

### Community 15 - "Cliente IA: embeddings y reintentos"
Cohesion: 0.22
Nodes (6): Vector de embedding de un texto corto (ver app/ai/embeddings.py para el dedup…, Manda el prompt al modelo y devuelve el texto crudo de la respuesta. NO parsea…, Ejecuta `func()` reintentando ante cualquier excepción, con backoff exponencial…, reintentar_con_backoff(), Logger, T

### Community 16 - "Reglas de negocio y decisiones de correo"
Cohesion: 0.22
Nodes (9): Reglas de negocio clave, Fase 3 — Correo ✅ IMPLEMENTADA (verificada contra MailHog real), Decisión: alerta de conector roto al pie del correo tras 3 corridas seguidas con falla, Decisión: estados 'nueva' y 'vista' cuentan como no aplicada (ESTADOS_NO_APLICADA), Decisión: no reenviar ofertas ya enviadas en las últimas 24h (HORAS_EXCLUSION_REENVIO), Decisión: score mínimo 40 por defecto (SCORE_MINIMO_DEFAULT), Fase 4 — Portal (desbloqueada), Fase 5 — Dashboard (pendiente) (+1 more)

### Community 17 - "README del Backend"
Cohesion: 0.25
Nodes (8): BreteAI-Backend README, Stack de BreteAI-Backend (Python, FastAPI, Ollama, SQLAlchemy/psycopg, APScheduler), Dependencias Fase 1 pineadas (requirements.txt), lever.json/adzuna.json armados a mano (sin token/board real en dev), Fixtures de conectores (una respuesta real por fuente), Fase 1 — Scraping + DB ✅ IMPLEMENTADA, Orden de implementación (`BreteAI-Backend/app/`), Fase Fix — deuda técnica acumulada

### Community 18 - "Tests de contrato de conectores"
Cohesion: 0.29
Nodes (7): parametrize, La fuente responde y trae al menos un item -- si esto falla, la API cambió de…, El pipeline completo _fetch() -> _map() sin mock produce al menos una…, Sin config en .env, _fetch() debe devolver [] (con warning en logs), nunca…, test_fetch_no_crashea_sin_credenciales(), test_fetch_trae_items_reales(), test_fetch_y_map_producen_ofertas_validas()

### Community 19 - "Fixtures de respuestas de Ollama"
Cohesion: 0.33
Nodes (6): Fixtures deben salir de corridas reales contra el modelo, no inventadas a mano, Fixtures de respuestas de Ollama (Fase 2), Fixture: respuesta_con_fences.txt (JSON envuelto en ```json), Fixture: respuesta_con_preambulo.txt (prosa + JSON), Fixture: respuesta_sin_json.txt (solo prosa, sin JSON), Fixture: respuesta_truncada.txt (JSON cortado por límite de tokens)

### Community 20 - "Requirements: stack e IA"
Cohesion: 0.40
Nodes (6): Stack CERRADO (v1), Principios transversales (DRY, modularidad, staging como frontera de testeo, no inventar decisiones), §5 Capa de IA (tareas: resumen, extracción, score, dedup semántico, feedback), §14 Estrategia de Pruebas (Bruno, pytest, testcontainers, MailHog), §5.2 Modelo: 100% local con Ollama, Qwen 2.5 7B Q4_K_M, §10 Notificaciones / Correo (Gmail SMTP con App Password)

### Community 21 - "Submódulos del monorepo"
Cohesion: 0.50
Nodes (4): BreteAI-Frontend, BreteAI-Infra, Submódulos git, BreteAI (proyecto)

### Community 22 - "Dependency de sesión FastAPI"
Cohesion: 0.67
Nodes (3): get_db(), Session, Dependency de FastAPI para inyectar sesión en endpoints.

## Knowledge Gaps
- **36 isolated node(s):** `Cómo usar esta guía`, `Orden de implementación (`BreteAI-Backend/app/`)`, `Decisiones que quedaron abiertas a propósito`, `No hacer en esta fase`, `Decisiones tomadas (quedaron escritas en el código, ver docstrings para el detalle)` (+31 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **8 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `OfertaCanonica` connect `Conectores de fuentes (Fase 1)` to `Perfil TOON y parser`, `Cliente LLM y dedup semántico`, `Analyzer y schemas de IA`?**
  _High betweenness centrality (0.103) - this node is a cross-community bridge._
- **Why does `get_settings()` connect `Cliente IA, retry compartido y config` to `Conectores de fuentes (Fase 1)`, `Perfil TOON y parser`, `Cliente LLM y dedup semántico`, `Cliente SMTP y envío de correo`, `Registro de conectores y sesión DB`, `Entrypoint FastAPI y scheduler`, `Settings de ATS (boards/tokens)`?**
  _High betweenness centrality (0.070) - this node is a cross-community bridge._
- **Why does `Oferta` connect `Plantilla HTML del correo` to `Modelos ORM y alertas de conectores`, `Perfil TOON y parser`, `Cliente LLM y dedup semántico`, `Cliente SMTP y envío de correo`?**
  _High betweenness centrality (0.059) - this node is a cross-community bridge._
- **Are the 10 inferred relationships involving `OfertaCanonica` (e.g. with `AdzunaConnector` and `ArbeitnowConnector`) actually correct?**
  _`OfertaCanonica` has 10 INFERRED edges - model-reasoned connections that need verification._
- **Are the 10 inferred relationships involving `BaseConnector` (e.g. with `AdzunaConnector` and `ArbeitnowConnector`) actually correct?**
  _`BaseConnector` has 10 INFERRED edges - model-reasoned connections that need verification._
- **Are the 7 inferred relationships involving `Modalidad` (e.g. with `ArbeitnowConnector` and `AshbyConnector`) actually correct?**
  _`Modalidad` has 7 INFERRED edges - model-reasoned connections that need verification._
- **What connects `Cómo usar esta guía`, `Orden de implementación (`BreteAI-Backend/app/`)`, `Decisiones que quedaron abiertas a propósito` to the rest of the system?**
  _36 weakly-connected nodes found - possible documentation gaps or missing edges._