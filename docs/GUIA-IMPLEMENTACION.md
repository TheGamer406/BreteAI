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
| 1 — Scraping + DB | `BreteAI-Backend`, `BreteAI-Infra` | Infra lista (docker-compose + esquema DB aplicado). **Código de conectores: pendiente.** | `design.md` §1-3, `requirements.md` §4 |
| 2 — IA | `BreteAI-Backend` | Pendiente (bloqueada por Fase 1) | `design.md` §1, §4-B, `requirements.md` §5 |
| 3 — Correo | `BreteAI-Backend` | Pendiente | `design.md` §4-C, `requirements.md` §10 |
| 4 — Portal | `BreteAI-Frontend` + endpoints en `BreteAI-Backend` | Pendiente | `design.md` §4-D, `requirements.md` §7-8 |
| 5 — Dashboard | `BreteAI-Frontend` + queries en `BreteAI-Backend` | Pendiente | `design.md` §4-E, `requirements.md` §9 |

---

## Fase 1 — Scraping + DB

**Objetivo de la fase:** cada conector trae ofertas de su fuente, las guarda tal cual en
`ofertas_raw` (staging), y el scheduler corre esto 4 veces al día. **Todavía NO incluye la IA**
(eso es Fase 2) — en esta fase `ofertas_raw` se llena, pero se queda ahí esperando.

**Ya hecho:** `BreteAI-Infra/docker-compose.yml` + `BreteAI-Infra/db/init/001_schema.sql`
(las 8 tablas ya existen si corrés `docker compose up -d` en `BreteAI-Infra`).

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

## Fase 2 — IA (resumen)

Bloqueada hasta que Fase 1 llene `ofertas_raw` con datos reales. Cuando se implemente:
leer `design.md` §4-B (validación de la salida del LLM con Pydantic, worker con mock de Ollama)
y `requirements.md` §5. El worker toma filas `pendiente` de `ofertas_raw` y escribe en `ofertas`.

## Fase 3 — Correo (resumen)

Bloqueada hasta Fase 2 (necesita `score` para armar el top 5-10). Ver `design.md` §4-C.

## Fase 4 — Portal (resumen)

Bloqueada hasta tener datos reales en `ofertas`. Ver `design.md` §4-D — el contrato de API
se define con una colección Bruno antes de tocar el frontend.

## Fase 5 — Dashboard (resumen)

Bloqueada hasta Fase 4 (reusa auth y estructura del portal). Ver `design.md` §4-E.

---

## Estructura de módulos — Fase 1 (árbol completo)

```
BreteAI-Backend/
├── app/
│   ├── config.py
│   ├── db/
│   │   ├── models.py
│   │   └── session.py
│   ├── connectors/
│   │   ├── canonical.py       ← modelo canónico (contrato)
│   │   ├── base.py            ← lógica compartida (DRY)
│   │   ├── remotive.py
│   │   ├── remoteok.py
│   │   ├── arbeitnow.py
│   │   ├── jobicy.py
│   │   ├── himalayas.py
│   │   ├── adzuna.py
│   │   ├── greenhouse.py
│   │   ├── lever.py
│   │   └── ashby.py
│   ├── pipeline/
│   │   ├── staging.py
│   │   └── runner.py
│   ├── scheduler/
│   │   └── jobs.py
│   └── alerts/
│       └── connector_health.py
├── tests/
│   ├── fixtures/
│   ├── unit/
│   ├── integration/
│   └── contract/
└── requirements.txt
```

Cada archivo de esa lista ya existe como esqueleto con un docstring — abrilo antes de escribir,
el comentario de arriba tiene el detalle específico de qué va ahí.

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
