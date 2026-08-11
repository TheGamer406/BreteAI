# Requerimientos - BreteAI

> **BreteAI** — Buscador de trabajos automatizado con integración de IA.
> Definido a partir de `idea.md`, `inspiration.md` y las respuestas de `preguntas.md` (63 + 64-76).
> Estado: **v1 — decisiones de arquitectura CERRADAS.** Perfil del candidato en `perfil.toon`.
> Todo corre **local** (server propio del usuario), gratis/open source, tope ~$10/mes.
> **Diseño técnico detallado** (pipeline por etapas, modelo canónico, DDL, tests): ver `design.md`.

---

## 1. Visión

Sistema automatizado y **open source** que busca ofertas de trabajo en múltiples fuentes varias veces al día, las analiza con IA (resumen + score de match 0-100 contra el perfil del usuario), las guarda en base de datos y notifica por correo. Un portal web permite revisar, filtrar y gestionar las ofertas como tickets, más un dashboard de analítica.

**Doble objetivo:**
1. **Uso real:** encontrar un mejor trabajo (meta salarial ↓ sección 2).
2. **Portafolio:** proyecto demostrable de "resolver un problema real con IA + automatización + buenas prácticas", publicable en GitHub y LinkedIn.

**Configurable para cualquiera:** el perfil (rol, salario, ubicación, skills) vive en un archivo de configuración; otra persona clona el repo, cambia su perfil y funciona.

---

## 2. Perfil del Usuario (contexto para la IA)

- Estudiante de Ingeniería en Desarrollo de Software (UCenfotec), nivel **Junior**.
- Intereses: **Backend, Data Science, Data Analyst** (y afines).
- Ubicación: **Costa Rica**. Abierto a **presencial, virtual (remoto) y mixto**, incluyendo **reubicación internacional**.
- Salario objetivo y actual: **configurables en `perfil.toon`** (privado). El sistema filtra/prioriza según la meta salarial definida ahí.
- CV en inglés y español (PDF en la raíz) → **ya convertido a `perfil.toon`** (formato TOON, ligero), que consume la IA. Editable para adaptarlo a otra persona.

---

## 3. Alcance del MVP y Roadmap

Orden acordado de construcción (cada fase entrega algo funcional):

1. **Fase 1 — Scraping + Base de Datos.** Obtener ofertas de fuentes legales y persistirlas. Con esto se entiende qué datos hay disponibles.
2. **Fase 2 — IA (clasificación/resumen/score).** Parametrizar el modelo con el perfil e instrucciones.
3. **Fase 3 — Correo.** Notificación con top de ofertas.
4. **Fase 4 — Portal web.** Visor + gestión tipo ticket + login.
5. **Fase 5 — Dashboard.** Analítica.

**Dedicación:** ~15-20 h esta semana, luego ~10 h/semana por 3-5 semanas.

---

## 4. Obtención de Ofertas (Scraping)

**Principio rector: solo métodos legales.** Priorizar APIs oficiales/públicas y feeds abiertos; el scraping HTML directo de LinkedIn/Indeed queda descartado por sus términos de servicio.

### 4.1 Fuentes recomendadas (investigadas, gratis y legales)

**Feeds/APIs abiertas de empleo remoto (sin login, sin token):**
- Remotive, RemoteOK, Arbeitnow, Jobicy, Himalayas, Working Nomads, WeWorkRemotely.
- **Adzuna API** — 1.000 llamadas/mes gratis, buena para agregados y datos de salario/tendencias (útil para tu análisis de data).

**Empresas grandes (IBM, Amazon, Microsoft, etc.) vía sus ATS con API pública JSON:**
- **Greenhouse** — `GET https://boards-api.greenhouse.io/v1/boards/{board_token}/jobs?content=true` (la más limpia).
- **Lever** — API pública oficial.
- **Ashby** — API pública.
- **Workday** — sin API pública; requiere render JS (dejar para después / opcional).

### 4.2 Fuentes Costa Rica

- Computrabajo CR, elempleo, Brete.cr: **no exponen API pública** → requieren scraping HTML (mayor riesgo legal y de mantenimiento). **DECISIÓN: fase posterior** (no en v1). La v1 arranca con APIs oficiales/remotas. Ver `ROADMAP.md`.

### 4.3 Programación

- **4 corridas diarias**: 05:00, 11:00, 16:00, 22:00 (hora CR). Si hay resultados nuevos → correo.

### 4.4 Filtros base

- Nivel: Junior. Ubicación: Costa Rica + remoto internacional + reubicación.
- Modalidad: presencial / virtual / mixto.
- Salario objetivo: según `perfil.toon`. **DECISIÓN:** si la oferta no publica salario, buscar referencia web; si no hay, guardar `null` / "no especificado".
- Palabras clave derivadas del perfil (backend, data, junior, etc.).

### 4.5 Anti-bloqueo / robustez

- Al usar APIs oficiales se evitan captchas y rate limits en su mayoría.
- Respetar rate limits de cada API, con reintentos y backoff.
- **Alerta si un scraper se rompe** (cambio de HTML/API) → notificación. Registrar en logs.

### 4.6 Deduplicación

- Detectar oferta repetida entre fuentes por **empresa + descripción/puesto** (comparación asistida por IA/embeddings). En el portal marcar "esta oferta se parece a tal otra".

### 4.7 Histórico

- **Guardar todo** (incluyendo ofertas viejas) para análisis de datos posterior.

### 4.8 Pipeline por etapas (staging)

- **DECISIÓN:** flujo `obtener → guardar RAW tal cual → (fin de corrida) → cola → IA normaliza a estructura fija → tabla ofertas`.
- El JSON crudo se guarda en `ofertas_raw` (JSONB) **antes** de cualquier procesamiento; la propia tabla funciona como cola de la IA (sin Redis/RabbitMQ en v1).
- Beneficios: reprocesable sin volver a llamar APIs, idempotente (`UNIQUE fuente + id_externo`), y cada etapa se prueba por separado. Detalle en `design.md` §1.

---

## 5. Capa de IA

### 5.1 Tareas

- **Resumir** descripciones largas a datos clave (requisitos, beneficios).
- **Extraer** campos estructurados (modalidad, salario si existe, ubicación, seniority).
- **Enriquecer / info de segunda mano** (ej: detectar "no es IBM directo, es un proveedor de servicios de IBM").
- **Score de match 0-100** contra el perfil (probabilidad de destacar como aplicante → prioridad).
- **Deduplicación semántica** (sección 4.6).
- **Corrección manual (feedback simple):** el usuario corrige una clasificación y se guarda como **ejemplo/ajuste de criterios del prompt** (NO re-entrenamiento del modelo). Se mantiene simple.

### 5.2 Modelo

- **DECISIÓN: 100% local con Ollama (gratis).** Sin API de pago (el usuario prefiere costo cero garantizado).
- El usuario ya tiene **Llama 3.2**. Recomendación investigada para RTX 1660 Super 8GB: **Qwen 2.5 7B (Q4_K_M)** — buen multilingüe (ES/EN) y cabe en 8GB. Alternativas: Llama 3.1 8B, Mistral Small 3, Phi-4 Mini (más rápido/liviano).
- **Latencia aceptada** (no necesita ser el más rápido, pero no días).
- **Corre en el server con GPU** (siempre encendido).
- *Nota de referencia:* la API de Claude/Haiku hubiera costado centavos/día para este volumen, pero **Claude Pro ($20/mes) NO incluye API** (se paga aparte). Se descartó para mantener costo $0.

### 5.3 Monitoreo del server GPU

- Script (AppScript u otro) que hace **ping ligero** al server y **notifica cuando no está disponible**. → Detalle en `alertServer.md` (crear después).

---

## 6. Base de Datos

- **DECISIÓN: PostgreSQL** (mejor para JSON crudo de ofertas + analítica del dashboard).
- Corre en el **mismo server** que la GPU.
- **Backups automáticos diarios ~17:00.**
- **Modelo canónico de oferta:** formato único al que todo conector mapea su fuente (ver `design.md` §2); la DB, la IA, el correo y el portal solo hablan ese formato.
- Tablas principales: `corridas` (registro de cada corrida), `ofertas_raw` (staging/cola, JSON crudo), `ofertas` (estructura fija), más gestión (`comentarios`, `adjuntos`, `oferta_historial`, `feedback_ia`, `correos`). DDL borrador en `design.md` §3.
- Campos por oferta: título, empresa, ubicación, salario, modalidad, link, descripción, fecha, **score**, **estado**, comentarios, fuente, JSON crudo original.
- **Estados:** `nueva`, `vista`, `aplicada`, `enProceso`, `enEspera` (respondieron / avanzando), `respondida`, `rechazada`.
- **Optimización de consultas:** se buscará rendimiento con índices bien pensados; Stored Procedures opcionales para las consultas clave (aplicadas / no aplicadas). *Nota: con un solo server el impacto es menor; un ORM con buenos índices suele ser suficiente y más mantenible — a validar en implementación.*

---

## 7. Backend

- **DECISIÓN: Python + FastAPI.** El corazón del proyecto es scraping + IA + análisis de datos (mejor ecosistema y alineado con Data Science).
- Expone **API REST** para el frontend.
- **Autenticación:** login simple de **un solo usuario**, con **contraseña hasheada (bcrypt/argon2)** y proceso seguro (tokens JWT o sesión). No texto plano.
- **Scheduler:** dentro del server (cron del contenedor / APScheduler / systemd timer) — **no** GitHub Actions (no debe correr el pipeline). 
- **Hosting:** server propio del usuario, accesible vía **Tailscale** (tunnel/HTTPS). Tiene IP pública y un dominio (dominio no considerado necesario por ahora).

---

## 8. Frontend / Portal Web

- **DECISIÓN: Next.js** — se necesita interactividad (tickets, filtros, estados); Astro descartado.
- **Login requerido** (mismo esquema seguro del backend).
- **Diseño:** pensado para **PC pero responsivo**; estética **fresca, fácil de usar a diario, sin sobrecargar, que no parezca hecho por IA**.
- **Vistas v1:**
  - Visor general de ofertas.
  - Ofertas **no aplicadas**.
  - Ofertas **aplicadas**.
  - Ofertas del **último correo**.
  - **Dashboard** (sección 9).
- **Gestión tipo ticket** (NO tablero Jira completo): cada oferta se abre como ticket con **etiquetas, comentarios, archivos adjuntos, estado**.
- Filtros: empresa, modalidad (presencial/virtual/mixto), ubicación (CR / fuera y dónde), estado, score.
- **DECISIÓN: hospedar el frontend en el mismo server** (dentro de la red Docker, junto al backend). Más simple, todo junto. Acceso externo vía Tailscale.

---

## 9. Dashboard (Analítica)

Métricas:
- Ofertas aplicadas vs. no aplicadas.
- Ofertas con respuesta / sin respuesta / en proceso (por estado).
- Tasa de respuesta.
- Cantidad por modalidad (presencial / virtual / mixto).
- Ofertas dentro de CR vs. fuera y en qué países (lista).
- Navegación y filtros interactivos.

---

## 10. Notificaciones / Correo

- **Un solo canal: correo** (no Telegram/WhatsApp/Notion). Es solo el disparador para ir al portal.
- Destinatario: `jmurillochevez@gmail.com`.
- **Contenido:** top 5-10 ofertas (de la última corrida o de las **no aplicadas** aún), en formato **card**: puesto, score, empresa, modalidad, salario (si se obtuvo), match, y **link directo al portal a esa oferta**.
- **DECISIÓN: Gmail SMTP con App Password** desde el backend (gratis, mantiene el stack simple).

---

## 11. Infraestructura, Docker y Despliegue

- **Servicios separados**, comunicados por una **red Docker** dedicada (aún no creada).
- **Repos separados** (multi-repo): backend, frontend, base de datos/infra.
- **Volúmenes persistentes:** DB, correos, modelos de IA, logs.
- **DECISIÓN CI/CD:** GitHub Actions con **runners gratuitos de GitHub** para build/test → **auto-deploy por SSH sobre Tailscale** al server (`docker compose pull && up -d`). Es el flujo DevOps estándar (CI en la nube, CD al server) que el usuario quiere aprender.
- **Secretos:** `.env` en el server; **GitHub Secrets** para el pipeline (llave SSH, etc.).
- **DECISIÓN nombre del proyecto: `BreteAI`** ("brete" = trabajo en CR). Se usa para repos, red Docker, etc.
  - Repos sugeridos: `breteai-backend`, `breteai-frontend`, `breteai-infra` (docker/DB).
  - Red Docker: `breteai-net`.

---

## 12. Graphify (requisito esencial)

- Mapear relaciones **entre los repos** y **dentro de cada repo**.
- **Parte del pipeline** (para iteraciones con IA y comprensión del usuario).
- Salidas esperadas: **documentación, dependencias, onboarding futuro** + iteraciones con IA.

---

## 13. Legal y Riesgos

- **Todo dentro de lo legal**: priorizar APIs oficiales/públicas; evitar scraping prohibido.
- Privacidad: no es preocupación mayor (buscar trabajo ya expone datos).
- **Alerta cuando un scraper se rompa** por cambios de la fuente.
- **Apify (referencia del video):** plan gratis = **$0/mes con $5 de crédito mensual** (no acumulable, 25 runs concurrentes, 8GB RAM). Útil como respaldo puntual, pero el diseño base es sin Apify para mantener costo $0. *(Anotado al final como pediste.)*

---

## 14. Estrategia de Pruebas

- **DECISIÓN cliente API: Bruno** (open source, offline; colecciones en texto plano versionadas en git, CLI `bru run` para CI). Alternativa a Postman.
- **Principio:** cada flujo se prueba **aislado** gracias al pipeline por etapas (cada etapa lee de un lado y escribe en otro). Detalle completo por flujo en `design.md` §4.
  - **Conectores:** fixtures JSON reales → unitarios de mapeo sin red; tests de **contrato** contra la API real en workflow scheduled = alerta de conector roto.
  - **IA:** validación Pydantic de la salida del LLM (bien formada, malformada, truncada, fuera de rango); worker con mock de Ollama; golden set de scores contra Ollama real solo en el server.
  - **Correo:** render del template unitario; envío contra MailHog (SMTP falso en Docker). Gmail real nunca en tests.
  - **API/Portal:** TestClient de FastAPI + Postgres de prueba; colección Bruno de la API propia como contrato para el frontend.
  - **Dashboard:** agregaciones con seed conocido → números exactos.
- **Herramientas:** pytest, testcontainers, Bruno, MailHog, ruff, Vitest/Testing Library, Playwright (opcional).
- **CI:** lint → unit tests → build → smoke test (compose + healthchecks) → deploy. Contratos externos en workflow scheduled aparte (no bloquean deploy).

---

## 15. Presupuesto

- **Gratis / open source**, tope duro ~$10/mes. (El usuario ya paga Claude $20 aparte; ese pago **no** cubre API.)

---

## Fuentes de la investigación

- [Free Jobs API: 8 opciones (JobsPipe)](https://jobspipe.dev/free-jobs-api)
- [Remote Jobs Aggregator — Arbeitnow/Jobicy/RemoteOK (Apify)](https://apify.com/benthepythondev/remote-jobs-aggregator)
- [Adzuna Jobs API](https://apify.com/nabeelbaghoor/jobflow-adzuna)
- [How to scrape jobs from any career page — Greenhouse/Lever/Ashby/Workday (DEV)](https://dev.to/get_anything/how-to-scrape-jobs-from-any-company-career-page-greenhouse-lever-ashby-workday-46f1)
- [Scrape Lever & Greenhouse programmatically 2026](https://dataresearchtools.com/how-to-scrape-lever-and-greenhouse-job-boards-programmatically-2026/)
- [Apify Free Tier ($5 credits)](https://use-apify.com/docs/what-is-apify/apify-free-plan)
- [Best Local LLMs for 8GB VRAM 2026](https://www.mayhemcode.com/2026/06/best-local-llms-for-4gb-6gb-and-8gb.html)
