# Guía de Implementación - BreteAI

> **Para agentes de IA que van a escribir código en este repo.**
> Este documento es el punto de entrada: decí "implementá la Fase N" y este archivo te dice
> qué leer, qué archivos tocar y en qué orden. No necesitás releer todo `requirements.md` o
> `design.md` de cero cada vez — las referencias puntuales están abajo, por archivo.

---

## Cómo usar esta guía

1. Identificá la fase a implementar (tabla de abajo).
2. Leé **solo** las secciones de `docs/design.md` y `docs/requirements.md` referenciadas para esa fase.
3. Andá a los archivos listados, en el orden dado. Cada archivo tiene un **docstring al inicio**
   que dice qué implementar ahí y qué NO duplicar (mirá el módulo base referenciado).
4. No crees archivos ni carpetas nuevas fuera de la estructura ya planteada sin necesidad real.
5. Reglas de todo el proyecto (no repetir en cada fase): ver `CLAUDE.md` y "Principios transversales" al final de este documento.

| Fase | Repo | Estado | Doc de referencia |
|---|---|---|---|
| 1 — Scraping + DB | `BreteAI-Backend`, `BreteAI-Infra` | ✅ **Funcional** (2026-08-12). 411 ofertas reales en `ofertas_raw`, 35 tests. | `design.md` §1-3, `requirements.md` §4 |
| 2 — IA | `BreteAI-Backend`, `BreteAI-Infra` (servicio Ollama) | ✅ **Funcional** (2026-08-15). Verificado contra LLM real (LM Studio en dev; Ollama es el runtime documentado de producción — mismo cliente, API OpenAI-compatible). | `design.md` §1, §4-B, §5, `requirements.md` §5 |
| 3 — Correo | `BreteAI-Backend` | Desbloqueada: ya hay ofertas con `score` en `ofertas`. | `design.md` §4-C, `requirements.md` §10 |
| 4 — Portal | `BreteAI-Frontend` + endpoints en `BreteAI-Backend` | Pendiente | `design.md` §4-D, `requirements.md` §7-8 |
| 5 — Dashboard | `BreteAI-Frontend` + queries en `BreteAI-Backend` | Pendiente | `design.md` §4-E, `requirements.md` §9 |
| Fix — deuda técnica | varios | Pendientes de fases cerradas que no bloquean. Ver `ROADMAP.md` § Fase Fix. | — |

---

## Fase 1 — Scraping + DB ✅ IMPLEMENTADA

**Objetivo de la fase:** cada conector trae ofertas de su fuente, las guarda tal cual en
`ofertas_raw` (staging), y el scheduler corre esto 4 veces al día. **No incluye la IA**
(eso es Fase 2) — en esta fase `ofertas_raw` se llena y se queda ahí esperando.

**Estado:** funcional y verificada con datos reales. Lo que quedó pendiente está en
`ROADMAP.md` § Fase Fix. La sección de abajo queda como referencia de cómo está armada
(útil para entender el patrón antes de escribir Fase 2, que lo replica).

**Contexto necesario antes de tocar código:**
- `docs/design.md` §1 (diagrama del pipeline por etapas) y §2 (modelo canónico de oferta).
- `docs/requirements.md` §4.1 (lista de fuentes) y §4.5 (anti-bloqueo/alerta de conector roto).

### Orden de implementación (`BreteAI-Backend/app/`)

1. **`config.py`** — variables de entorno (conexión DB, API keys de Adzuna, board tokens de ATS).
2. **`db/models.py`** — modelos ORM que mapean 1:1 el DDL de `db/init/001_schema.sql` (Infra). No inventes columnas nuevas; si falta algo, es una decisión de diseño — pará y preguntá antes de improvisar el schema.
3. **`db/session.py`** — engine/sesión de SQLAlchemy.
4. **`connectors/canonical.py`** — el modelo canónico (Pydantic) de `design.md` §2. **Este es el contrato que todos los conectores deben cumplir.**
5. **`connectors/base.py`** — la clase base abstracta. Acá vive TODA la lógica compartida (fetch con reintentos/backoff, guardar en `ofertas_raw`, manejo de errores, idempotencia). **DRY: ningún conector individual debe reimplementar esto.**
6. **`connectors/<fuente>.py`** (uno por archivo, ver lista en el árbol de abajo) — cada uno implementa SOLO `_fetch()` (llamada HTTP a su API) y `_map()` (su JSON → modelo canónico). Nada más.
7. **`pipeline/staging.py`** — inserta en `ofertas_raw` con `UNIQUE(fuente, id_externo)` (idempotencia real, no solo de intención).
8. **`pipeline/runner.py`** — orquesta una corrida completa: crea fila en `corridas`, corre todos los conectores, cierra la fila con el resultado (`ok`/`parcial`/`error`).
9. **`scheduler/jobs.py`** — APScheduler con los 4 triggers diarios (05:00, 11:00, 16:00, 22:00 CR), llama a `pipeline/runner.py`.
10. **`alerts/connector_health.py`** — si un conector falla repetidamente, generar la alerta (por ahora: log estructurado; el canal real de notificación es Fase 3, correo).

### Tests de esta fase (ver `design.md` §4-A)

- `tests/fixtures/<fuente>.json` — una respuesta real guardada de cada API (para tests sin red).
- `tests/unit/test_canonical_mapping.py` — cada `_map()` contra su fixture.
- `tests/integration/test_staging_pipeline.py` — conector con fixture → Postgres de test → verifica fila + idempotencia (correr 2 veces, no duplica).
- `tests/contract/test_connectors_contract.py` — pega a la API real, marcado `@pytest.mark.contract` (no corre en cada push).

### No hacer en esta fase (para no salirse de alcance)

- Nada de score/resumen/IA — esos campos quedan `NULL` en `ofertas` hasta Fase 2.
- Nada de dedup semántico (embeddings) — solo `UNIQUE(fuente, id_externo)` en raw. El dedup empresa+puesto es Fase 1 pero puede ir al final, no bloquea el resto.
- No tocar `BreteAI-Frontend` ni endpoints de API pública todavía.

---

## Fase 2 — IA ✅ IMPLEMENTADA

**Objetivo de la fase:** el worker toma las filas `pendiente` de `ofertas_raw`, se las manda
a un LLM local junto con el perfil del candidato, y escribe en `ofertas` la versión
enriquecida: resumen, campos extraídos y **score de match 0-100**. Con esto el sistema pasa de
"junté ofertas" a "sé cuáles te sirven".

**Estado:** funcional y verificada contra un LLM real (Qwen2.5-7B-Instruct, Q4_K_M) corriendo
en LM Studio en esta máquina de desarrollo (`http://localhost:1234`, API OpenAI-compatible).
**Nota de runtime:** `requirements.md` §5.2 define Ollama como el runtime de producción — el
cliente (`app/ai/client.py`) habla la API OpenAI-compatible (`/v1/chat/completions`,
`/v1/embeddings`, `/v1/models`) que exponen **ambos** de forma casi idéntica, así que el mismo
código sirve para los dos; solo cambia `OLLAMA_HOST`/`OLLAMA_MODEL` en `.env`. En el server con
GPU, levantar el servicio `ollama` de `BreteAI-Infra/docker-compose.yml` y usar esas variables.

La sección de abajo queda como referencia de cómo está armada (útil para Fase 3, que reusa el
mismo patrón de template method + aislamiento de fallos).

**Contexto necesario antes de tocar código (leer SOLO esto):**
- `docs/design.md` §1 (el diagrama del pipeline: dónde encaja el worker) y **§5 Riesgo #1**
  (por qué la validación de la salida del LLM es lo primero).
- `docs/design.md` §4-B (la tabla de qué se testea en este flujo).
- `docs/requirements.md` §5 (tareas de la IA, modelo elegido y por qué).
- `config/perfil.example.toon` (estructura del perfil que consume el prompt).

### Orden de implementación

> Está pensado para que **cada paso se pueda probar antes de seguir al siguiente**.
> Los pasos 1-3 no necesitan GPU ni Ollama corriendo.

1. **`app/ai/schemas.py`** — el schema Pydantic de la salida del LLM + `parsear_respuesta_llm()`.
   **Empezar por acá aunque parezca al revés:** es la mitigación del Riesgo #1 y define el
   contrato que todo lo demás asume. Se testea entero sin GPU.
2. **`app/ai/perfil.py`** — cargar y parsear `resources/perfil.toon`.
3. **`app/ai/prompts.py`** — armar el prompt (perfil + criterios + oferta + schema pedido).
4. **Infra: servicio Ollama** — agregar al `docker-compose.yml` de `BreteAI-Infra` (imagen
   `ollama/ollama`, volumen persistente para los modelos, en la red `breteai-net`), y bajar el
   modelo. Recién acá hace falta la GPU.
5. **`app/ai/client.py`** — cliente HTTP de Ollama + config nueva en `app/config.py` y en los
   dos `.env.example`.
6. **`app/ai/analyzer.py`** — orquesta prompt → LLM → validación → `AnalisisIA`, con reintentos.
7. **`app/pipeline/worker.py`** — consume la cola y persiste en `ofertas`. Acá está la parte
   con más detalle fino (re-mapeo del raw, estados, aislamiento de fallos): leer su docstring
   completo antes de escribir.
8. **`app/scheduler/jobs.py`** (modificar) — agregar el paso 2: después de la corrida de
   scraping, correr el worker.
9. **`app/ai/salario.py`** — estimación cuando la oferta no publica salario.
10. **`app/ai/embeddings.py`** — dedup semántico → `ofertas.similar_a`. *Puede quedar al final.*
11. **`app/feedback/criterios.py`** — correcciones del usuario → criterios extra del prompt.
    *Puede quedar al final* (sin feedback acumulado el sistema funciona igual).

### Tests de esta fase (ver `design.md` §4-B)

- `tests/unit/test_ai_schemas.py` — **el más importante**: JSON válido, con fences, con
  preámbulo, truncado, score fuera de rango. Sin GPU.
- `tests/unit/test_prompts.py` — el prompt contiene perfil + criterios; trunca descripciones largas.
- `tests/integration/test_ai_worker.py` — worker completo, Postgres real (testcontainers) +
  **Ollama mockeado**. Corre en CI. Reusar las fixtures de `tests/conftest.py`.
- `tests/golden/test_golden_scores.py` — contra Ollama REAL, marker `golden`, solo en el server.
- `tests/fixtures/ollama/` — respuestas del LLM capturadas de corridas reales (ver su README).

**Patrón a copiar:** Fase 1 ya resolvió los mismos problemas (template method compartido,
aislamiento de fallos, mock del punto de entrada externo). Mirar `app/connectors/base.py` y
`tests/integration/test_staging_pipeline.py` antes de inventar un patrón nuevo.

### No hacer en esta fase (para no salirse de alcance)

- **No re-entrenar ni fine-tunear** el modelo. Decisión cerrada (`requirements.md` §5.1): el
  feedback solo ajusta criterios del prompt.
- **No inventar valores.** Si el LLM no da un dato confiable (salario, seniority), va `NULL`.
  Un dato alucinado contamina el score, el correo y el dashboard.
- **No cambiar los campos canónicos** que ya trajo Fase 1: si el LLM dice un título distinto
  al de la fuente, manda el de la fuente.
- No tocar el correo (Fase 3), ni endpoints/portal (Fase 4).

### Decisiones que quedaron abiertas a propósito

Están anotadas en los docstrings de cada archivo. Al resolverlas, **dejar la decisión escrita
en el código** (no solo implementarla):
- Dónde guardar los vectores de embeddings (columna en `ofertas` vs `pgvector`) → `embeddings.py`.
- De dónde sale la referencia de salario → `salario.py`.
- Si el worker re-analiza o saltea una oferta ya procesada → `worker.py`.
- Análisis en lote vs de a una oferta → `analyzer.py`.

Si una decisión no está en `design.md`/`requirements.md` **y** no está listada acá, es una
decisión de arquitectura nueva: pará y preguntá antes de asumir.

## Fase 3 — Correo (resumen)

Bloqueada hasta Fase 2 (necesita `score` para armar el top 5-10). Ver `design.md` §4-C.

## Fase 4 — Portal (resumen)

Bloqueada hasta tener datos reales en `ofertas`. Ver `design.md` §4-D — el contrato de API
se define con una colección Bruno antes de tocar el frontend.

## Fase 5 — Dashboard (resumen)

Bloqueada hasta Fase 4 (reusa auth y estructura del portal). Ver `design.md` §4-E.

---

## Estructura de módulos (árbol completo)

`✅` = implementado y verificado · `🔨` = esqueleto con docstring, a implementar

```
BreteAI-Backend/
├── app/
│   ├── config.py                      ✅ + vars de Ollama/IA (Fase 2)
│   ├── main.py                        ✅
│   ├── common/
│   │   └── retry.py                   ✅ backoff compartido (connectors/base.py + ai/client.py)
│   ├── db/
│   │   ├── models.py                  ✅ + columna `embedding` (dedup semántico)
│   │   └── session.py                 ✅
│   ├── connectors/                    ← Fase 1
│   │   ├── canonical.py               ✅ modelo canónico (contrato)
│   │   ├── base.py                    ✅ lógica compartida (DRY)
│   │   ├── registro.py                ✅ fuente → clase (runner.py Y worker.py lo usan)
│   │   ├── remotive.py                ✅
│   │   ├── remoteok.py                ✅
│   │   ├── arbeitnow.py               ✅
│   │   ├── jobicy.py                  ✅
│   │   ├── himalayas.py               ✅
│   │   ├── adzuna.py                  ✅
│   │   ├── greenhouse.py              ✅
│   │   ├── lever.py                   ✅
│   │   └── ashby.py                   ✅
│   ├── ai/                            ← Fase 2
│   │   ├── schemas.py                 ✅ salida del LLM validada (Riesgo #1)
│   │   ├── client.py                  ✅ cliente API OpenAI-compatible (Ollama/LM Studio)
│   │   ├── perfil.py                  ✅ carga + parser TOON de perfil.toon
│   │   ├── prompts.py                 ✅ armado de prompts
│   │   ├── analyzer.py                ✅ orquesta 1 oferta: prompt → LLM → validación
│   │   ├── salario.py                 ✅ estimación por promedio de ofertas ya guardadas
│   │   └── embeddings.py              ✅ dedup semántico → ofertas.similar_a
│   ├── feedback/                      ← Fase 2
│   │   └── criterios.py               ✅ feedback_ia → criterios extra del prompt
│   ├── pipeline/
│   │   ├── staging.py                 ✅ escribe en ofertas_raw
│   │   ├── runner.py                  ✅ orquesta una corrida de scraping
│   │   └── worker.py                  ✅ consume la cola: ofertas_raw → IA → ofertas
│   ├── scheduler/
│   │   └── jobs.py                    ✅ scraping + worker en cada corrida
│   └── alerts/
│       └── connector_health.py        ✅
├── tests/
│   ├── conftest.py                    ✅ Postgres real (testcontainers) + oferta_raw_factory
│   ├── fixtures/
│   │   ├── <fuente>.json              ✅ respuestas reales de las 9 fuentes
│   │   └── ollama/                    ✅ 6 respuestas reales capturadas (ver su README)
│   ├── unit/
│   │   ├── test_canonical_mapping.py  ✅ 18 tests
│   │   ├── test_ai_schemas.py         ✅ 11 tests — el más importante de Fase 2
│   │   └── test_prompts.py            ✅ 7 tests
│   ├── integration/
│   │   ├── test_staging_pipeline.py   ✅ 3 tests
│   │   └── test_ai_worker.py          ✅ 6 tests, LLM mockeado
│   ├── contract/
│   │   └── test_connectors_contract.py ✅ 14 tests (marker `contract`)
│   └── golden/
│       └── test_golden_scores.py      ✅ contra LLM real (marker `golden`, solo server/dev con GPU)
├── pytest.ini                         ✅ markers `contract` y `golden`
└── requirements.txt                   ✅ versiones pineadas
```

Todos los archivos ya existen y están implementados — abrilos para ver el detalle de cada
decisión (quedan documentadas en los docstrings, no solo en este índice).

---

## Principios transversales (aplican a TODAS las fases)

- **DRY:** si dos conectores (o dos endpoints, o dos tests) están por repetir la misma lógica,
  esa lógica va a un módulo base/compartido, no se copia y pega.
- **Modularidad:** cada archivo tiene una sola responsabilidad. Un conector no sabe de scheduler;
  el scheduler no sabe de HTTP; `staging.py` no sabe de qué fuente vino la oferta.
- **El staging (`ofertas_raw`) es la frontera de testeo:** cualquier etapa se puede probar
  sembrando datos ahí directamente, sin correr las etapas anteriores.
- **Nunca inventar decisiones de arquitectura** que no estén en `design.md`/`requirements.md`.
  Si falta una decisión, es una señal para preguntar, no para asumir.
- **Secretos:** todo en `.env` (gitignored), nunca hardcodeado, nunca en el commit.
