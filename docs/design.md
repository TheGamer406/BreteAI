# Diseño técnico - BreteAI

> Detalle de diseño de la v1. Complementa `requirements.md` (fuente de verdad de decisiones)
> y `ROADMAP.md` (control de alcance). Aquí vive el **cómo**: pipeline por etapas,
> modelo canónico, esquema de DB y estrategia de pruebas por flujo.

---

## 1. Pipeline por etapas (staging)

Principio: **cada etapa lee de un lado y escribe en otro**. Eso permite reprocesar sin
volver a llamar APIs y probar cada etapa por separado.

```
[Scheduler 4x/día: 05, 11, 16, 22 CR]
        │
        ▼
┌─────────────────┐    guarda TAL CUAL     ┌──────────────┐
│ Conectores       │ ─────────────────────► │ ofertas_raw  │  (JSONB crudo + metadatos)
│ (uno por fuente) │                        │  = la cola   │
└─────────────────┘                        └──────┬───────┘
                                                  │ fin de corrida:
                                                  │ worker IA toma las `pendiente`
                                                  ▼
                                        ┌───────────────────┐
                                        │ Worker IA (Ollama) │  normaliza al modelo canónico
                                        │ resumen + score +  │  + valida JSON de salida (Pydantic)
                                        │ extracción campos  │
                                        └────────┬──────────┘
                                                 ▼
                                          ┌────────────┐
                                          │  ofertas    │  (estructura fija)
                                          └─────┬──────┘
                              ┌─────────────────┼──────────────────┐
                              ▼                 ▼                  ▼
                        correo top 5-10    portal (tickets)    dashboard
```

Reglas del pipeline:

- **Guardar raw siempre**: si la IA falla o cambia el prompt, se reprocesa desde `ofertas_raw`
  sin tocar las APIs (histórico completo, §4.7 de requirements).
- **La cola es la propia tabla**: filas de `ofertas_raw` con `estado_proc = 'pendiente'`.
  Sin Redis/RabbitMQ en v1.
- **Idempotencia**: `UNIQUE (fuente, id_externo)` en raw → si una corrida se relanza,
  no se duplica nada (upsert / ignore).
- **Fallos aislados**: una raw que falla queda `estado_proc = 'error'` con mensaje y contador
  de intentos; es reintentable y nunca bloquea al resto.
- Cada corrida se registra en la tabla `corridas` (alimenta la alerta de conector roto y el dashboard).

## 2. Modelo canónico de oferta

Formato único al que cada conector mapea el JSON de su fuente. Todo lo demás
(DB, IA, correo, portal) habla solo este idioma.

| Campo | Tipo | Notas |
|---|---|---|
| `fuente` | string | `remotive`, `remoteok`, `arbeitnow`, `jobicy`, `himalayas`, `adzuna`, `greenhouse`, `lever`, `ashby` |
| `id_externo` | string | id en la fuente (para dedup/idempotencia) |
| `titulo` | string | |
| `empresa` | string | |
| `ubicacion` | string \| null | texto normalizado |
| `modalidad` | enum \| null | `presencial` / `remoto` / `mixto` (nota: "virtual" en requirements = `remoto`) |
| `salario` | objeto \| null | `{min, max, moneda}`; `null` = "no especificado" |
| `descripcion` | text | limpia (varios ATS devuelven HTML) |
| `url` | string | link directo a la oferta |
| `fecha_publicacion` | date \| null | |

Campos que **NO** pone el conector (los agrega la capa IA / el portal):
`resumen`, `score`, `seniority`, `empresa_real` (enriquecimiento), `estado`, etiquetas, comentarios.

## 3. Esquema de DB (borrador DDL)

PostgreSQL. Borrador para iterar en Fase 1; el ORM (SQLAlchemy) será la fuente final.

```sql
-- registro de cada corrida del scheduler (observabilidad + alerta de conector roto)
CREATE TABLE corridas (
  id            BIGSERIAL PRIMARY KEY,
  inicio        TIMESTAMPTZ NOT NULL DEFAULT now(),
  fin           TIMESTAMPTZ,
  estado        TEXT NOT NULL DEFAULT 'enCurso',   -- enCurso | ok | parcial | error
  fuentes_ok    TEXT[] NOT NULL DEFAULT '{}',
  fuentes_error TEXT[] NOT NULL DEFAULT '{}',
  notas         TEXT
);

-- staging: la oferta TAL CUAL vino de la fuente. Esta tabla ES la cola de la IA.
CREATE TABLE ofertas_raw (
  id           BIGSERIAL PRIMARY KEY,
  corrida_id   BIGINT REFERENCES corridas(id),
  fuente       TEXT NOT NULL,
  id_externo   TEXT NOT NULL,
  payload      JSONB NOT NULL,                     -- JSON crudo original, sin tocar
  obtenida_en  TIMESTAMPTZ NOT NULL DEFAULT now(),
  estado_proc  TEXT NOT NULL DEFAULT 'pendiente',  -- pendiente | procesada | error | descartada
  error_msg    TEXT,
  intentos     INT NOT NULL DEFAULT 0,
  UNIQUE (fuente, id_externo)                      -- idempotencia entre corridas
);
CREATE INDEX idx_raw_cola ON ofertas_raw(estado_proc) WHERE estado_proc = 'pendiente';

-- estructura fija: modelo canónico + campos de IA + gestión
CREATE TABLE ofertas (
  id                BIGSERIAL PRIMARY KEY,
  raw_id            BIGINT NOT NULL REFERENCES ofertas_raw(id),
  fuente            TEXT NOT NULL,
  id_externo        TEXT NOT NULL,
  -- canónico
  titulo            TEXT NOT NULL,
  empresa           TEXT,
  ubicacion         TEXT,
  pais              TEXT,                          -- para dashboard CR vs fuera
  modalidad         TEXT,                          -- presencial | remoto | mixto | NULL
  salario_min       NUMERIC,
  salario_max       NUMERIC,
  salario_moneda    TEXT,
  salario_estimado  BOOLEAN NOT NULL DEFAULT false, -- true si vino de referencia web
  descripcion       TEXT,
  url               TEXT NOT NULL,
  fecha_publicacion DATE,
  -- IA
  resumen           TEXT,
  requisitos        JSONB,
  beneficios        JSONB,
  seniority         TEXT,
  empresa_real      TEXT,                          -- "segunda mano": proveedor vs empresa real
  score             INT CHECK (score BETWEEN 0 AND 100),
  score_razon       TEXT,
  -- gestión (portal)
  estado            TEXT NOT NULL DEFAULT 'nueva', -- nueva|vista|aplicada|enProceso|enEspera|respondida|rechazada
  etiquetas         TEXT[] NOT NULL DEFAULT '{}',
  similar_a         BIGINT REFERENCES ofertas(id), -- dedup: "se parece a tal otra"
  creada_en         TIMESTAMPTZ NOT NULL DEFAULT now(),
  actualizada_en    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (fuente, id_externo)
);
CREATE INDEX idx_ofertas_estado    ON ofertas(estado);
CREATE INDEX idx_ofertas_score     ON ofertas(score DESC);
CREATE INDEX idx_ofertas_empresa   ON ofertas(empresa);
CREATE INDEX idx_ofertas_modalidad ON ofertas(modalidad);

-- historial de cambios de estado (tasa de respuesta y tiempos en dashboard)
CREATE TABLE oferta_historial (
  id        BIGSERIAL PRIMARY KEY,
  oferta_id BIGINT NOT NULL REFERENCES ofertas(id),
  de_estado TEXT,
  a_estado  TEXT NOT NULL,
  en        TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE comentarios (
  id        BIGSERIAL PRIMARY KEY,
  oferta_id BIGINT NOT NULL REFERENCES ofertas(id),
  texto     TEXT NOT NULL,
  creado_en TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE adjuntos (
  id        BIGSERIAL PRIMARY KEY,
  oferta_id BIGINT NOT NULL REFERENCES ofertas(id),
  nombre    TEXT NOT NULL,
  ruta      TEXT NOT NULL,                          -- archivo en volumen Docker, no en DB
  subido_en TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- correcciones del usuario → ajustan criterios del prompt (NO re-entrenamiento)
CREATE TABLE feedback_ia (
  id             BIGSERIAL PRIMARY KEY,
  oferta_id      BIGINT NOT NULL REFERENCES ofertas(id),
  campo          TEXT NOT NULL,                     -- ej: score, modalidad, seniority
  valor_ia       TEXT,
  valor_correcto TEXT,
  nota           TEXT,
  creado_en      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- correos enviados (vista "último correo" del portal)
CREATE TABLE correos (
  id         BIGSERIAL PRIMARY KEY,
  enviado_en TIMESTAMPTZ NOT NULL DEFAULT now(),
  oferta_ids BIGINT[] NOT NULL
);
```

## 4. Estrategia de pruebas por flujo

Herramientas: **pytest** + **testcontainers** (Postgres real en Docker para integración),
**Bruno** (cliente API open source; colecciones en texto plano versionadas en git, CLI `bru run`
para CI), **MailHog** (SMTP falso en contenedor), **ruff** (lint), **Vitest/Testing Library**
(frontend), **Playwright** (E2E opcional).

### Flujo A — Conectores → `ofertas_raw`
| Nivel | Qué prueba | Cómo |
|---|---|---|
| Unitario | Mapeo fuente → canónico | Fixture JSON real de cada fuente en el repo → función de mapeo → assert. **Sin red.** |
| Contrato | La fuente no cambió su formato | Colección Bruno por fuente + `pytest -m contract` contra la API real. Corre **scheduled** (no bloquea deploy). Esto ES la "alerta de conector roto". |
| Integración | Inserción + idempotencia | Conector con respuesta mockeada → Postgres testcontainers → assert filas en raw; correr 2 veces → no duplica. |

### Flujo B — Cola raw → IA → `ofertas`
| Nivel | Qué prueba | Cómo |
|---|---|---|
| Unitario | Armado del prompt con `perfil.toon`; **validación Pydantic de la salida del LLM** | Casos: JSON válido, malformado, truncado, score fuera de rango → rechazar sin crashear. |
| Integración (sin GPU) | Worker completo | Mock de Ollama (respuesta enlatada) → assert fila en `ofertas` + raw `procesada`; caso fallo → raw `error` reintentable. |
| Golden set | Regresiones de score | 5-10 ofertas reales con rango esperado (ej: "senior en Alemania sin remoto" < 40). Contra Ollama real, **solo en el server**, no en CI. |

### Flujo C — Correo
| Nivel | Qué prueba | Cómo |
|---|---|---|
| Unitario | Render del template | 5 ofertas → assert HTML: links al portal, top N, solo no-aplicadas. |
| Integración | Envío SMTP | Contenedor MailHog → assert destinatario/asunto/cuerpo. **Gmail real nunca en tests.** |

### Flujo D — API REST + Portal
| Nivel | Qué prueba | Cómo |
|---|---|---|
| Unitario/Integración | Endpoints (login, listar, filtrar, cambiar estado) | `TestClient` de FastAPI + Postgres de prueba. Auth: password mala → 401, token expirado → 401, hash bcrypt/argon2 (nunca texto plano). |
| Contrato | API propia estable para el frontend | Colección Bruno de la API BreteAI; `bru run` en CI detecta rupturas de contrato. |
| Frontend | Componentes (card, filtros) | Vitest + Testing Library. Playwright para login→ver→cambiar estado (opcional, no bloqueante en v1). |

### Flujo E — Dashboard
| Nivel | Qué prueba | Cómo |
|---|---|---|
| Integración | Queries de agregación | Seed conocido → assert números exactos (tasa de respuesta, conteos por modalidad/país). |

### Transversal — Infra/CI
- **Smoke test**: `docker compose up` → healthchecks (`/health` FastAPI, `pg_isready`, `ollama list`).
- **Pipeline CI** (GitHub Actions): `ruff` → unit tests → build imágenes → smoke → deploy SSH/Tailscale.
- Tests de contrato contra APIs externas: workflow **scheduled** aparte; si falla → notificación (conector roto).

## 5. Riesgos de diseño (orden por criticidad)

1. **Salida del LLM** — un 7B local devolverá JSON malformado con frecuencia. Mitigación: schema Pydantic estricto + reintentos + `estado_proc='error'` reprocesable. Diseñar/probar primero.
2. **Idempotencia** — corrida relanzada no duplica (`UNIQUE (fuente, id_externo)`).
3. **Contrato backend↔frontend** — definirlo (colección Bruno) antes de Fase 4.
4. **Secretos** — App Password Gmail, Adzuna key, password portal: solo `.env` + GitHub Secrets; nunca dentro de imágenes Docker.
5. **Scheduler** — APScheduler en el contenedor backend; tabla `corridas` registra qué pasó si hubo reinicio.
6. **Backup 17:00** — `pg_dump` a volumen; probar restore al menos una vez.
