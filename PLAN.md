# Mallorca Holística — Plan de desarrollo (MVP → fases posteriores)

Este plan se basa en la documentación de `DOCS/` (alcance del MVP, arquitectura y estructura básica de datos).

## 1) Resumen del producto (MVP)

**Objetivo:** lanzar una primera versión funcional que permita:
- Directorio verificado de profesionales del bienestar en Mallorca.
- Búsqueda “clásica” (terapia / nombre / localización) y visualización en mapa.
- Buscador de orientación con IA (necesidad en lenguaje natural → terapias + profesionales + contenidos).
- Agenda de actividades/eventos publicados por profesionales (con límites por plan).
- Validar modelo de suscripción (asignación manual en MVP; pagos activables más tarde).

**Roles principales:**
- Visitante/usuario: busca, explora, contacta y consulta eventos.
- Profesional: se registra, crea/edita perfil, sube documentación para verificación, publica eventos (según plan).
- Admin: verifica profesionales y documentos, gestiona terapias/keywords/contenidos/eventos, asigna planes.

**No-MVP (según DOCS):**
- Blog completo, reservas internas, pagos para eventos, analítica avanzada (más allá de métricas básicas).

## 2) Propuesta de arquitectura (orientativa)

> Si ya existe una preferencia de stack, este plan se ajusta; la intención es describir fases y entregables.

- **Frontend web:** SPA/SSR (p.ej. Next.js) con SEO para perfiles/terapias, diseño responsive.
- **Backend API:** servicio REST/GraphQL (p.ej. Node/Fastify/Nest) con autenticación y RBAC (professional/admin).
- **Base de datos:** PostgreSQL + PostGIS (mapa y búsquedas geoespaciales).
- **Almacenamiento de ficheros:** documentos de verificación (S3/GCS) + URL firmadas.
- **Búsqueda clásica:** filtros + full-text (Postgres FTS) + geofiltros (distancia) + paginación.
- **IA (orientación):** servicio de recomendación (reglas + embeddings) con trazabilidad y evaluación.
- **Admin:** panel protegido (puede ser app interna o módulo dentro del frontend).
- **Observabilidad:** logs, métricas, alertas; auditoría de acciones admin.
- **Cumplimiento:** GDPR, consentimiento cookies, términos/privacidad, retención de documentos.

## 3) Modelo de datos (MVP)

Basado en la estructura propuesta en `DOCS/5-EstructuraBasicaDeLaBaseDeDatosMVP.docx`:
- **professionals**: datos del perfil + localización (lat/lng) + verificación + plan + fechas.
- **therapies**: catálogo de terapias (nombre, descripción, categoría).
- **professional_therapies**: N↔N profesionales-terapias.
- **keywords**: necesidades/síntomas/temas (estrés, insomnio, ansiedad…).
- **professional_keywords**: N↔N profesionales-keywords.
- **activities/events**: agenda (título, descripción, fecha, lugar, ciudad, organizador, link externo, tipo).
- **therapy_contents**: contenidos informativos asociados a terapias (título, texto, terapia).

Extensiones típicas (MVP o fase siguiente, según prioridad):
- **verification_documents**: ficheros subidos + estado de revisión.
- **plans/plan_limits**: límites configurables (p.ej. eventos/año).
- **reviews**: valoraciones (estrellas, comentario, estado de moderación) + usuario.
- **metrics/events_tracking**: vistas/clics (perfil, contacto, calendario, web).

## 4) Plan por fases (con entregables y criterios de salida)

### Fase 0 — Preparación (1–3 días)
**Objetivo:** dejar el proyecto listo para construir y desplegar.
- Definir stack final, entornos (dev/staging/prod) y convenciones (naming, lint, CI).
- Estructura de repos (frontend/backend/shared/infra) y variables de entorno.
- Pipeline CI (tests + build) y CD básico a staging.

**Salida:** repo scaffold + deploy a staging “hello world”.

### Fase 1 — UX, arquitectura de información y diseño (3–7 días)
**Objetivo:** convertir el alcance en pantallas y flujos claros.
- User journeys: buscar → explorar → contactar / evento.
- Wireframes: Home (doble buscador), Directorio, Perfil, Terapia, Agenda, Registro profesional, Admin.
- Definir taxonomía: categorías de terapias, keywords, filtros, copy y disclaimers (IA no es diagnóstico).

**Salida:** prototipo navegable + especificación de componentes.

### Fase 2 — Base de datos y backend “core” (1–2 semanas)
**Objetivo:** API mínima para soportar directorio, terapias, keywords, mapa y agenda.
- PostgreSQL + migraciones + seed inicial (terapias + keywords mínimas).
- CRUD profesional (perfil, localización, terapias, keywords, enlaces, calendario externo).
- Publicación y lectura de eventos (con límites por plan).
- Endpoints de búsqueda clásica: filtros + paginación + geobúsqueda (radio / bounding box).
- Autenticación y roles (professional/admin); rate limiting y validación de inputs.

**Salida:** API estable para frontend + documentación (OpenAPI) + seeds reproducibles.

### Fase 3 — Verificación y panel de administración (1–2 semanas)
**Objetivo:** habilitar el “directorio verificado”.
- Subida de documentos (seguro RC, carta deontológica, diplomas) + storage seguro.
- Estados de verificación (pendiente/aprobado/rechazado) con trazabilidad.
- Admin: cola de revisión, visor de documentos, acciones (aprobar/rechazar), asignación de plan.
- Admin: gestión de terapias/keywords/contenidos/eventos (CRUD + moderación básica).

**Salida:** flujo completo “registro → revisión → publicación”.

### Fase 4 — Frontend público (1–2 semanas)
**Objetivo:** experiencia usable y SEO-friendly.
- Home con:
  - Buscador clásico (terapia/nombre/localización).
  - Buscador IA (necesidad en lenguaje natural).
  - Contador de comunidad (“Ya somos N…”).
  - Accesos: directorio, agenda, terapias, registro profesionales.
- Directorio: lista + filtros + mapa interactivo.
- Perfil profesional: info, terapias, keywords, enlaces, contacto y link a Calendly/Google Calendar (según plan).
- Páginas de terapia: descripción + profesionales asociados + contenidos relacionados.
- Agenda: próximos eventos + filtros + link externo de inscripción.

**Salida:** MVP navegable end-to-end en staging.

### Fase 5 — Orientación con IA (1–2 semanas, iterativa)
**Objetivo:** recomendación útil, transparente y no sesgada por plan.

**Diseño recomendado (MVP robusto):**
- **Base de conocimiento:** terapias + keywords + contenidos (curados) + relación terapia↔keyword.
- **Comprensión de necesidad (NLU):**
  - Opción A (rápida): embeddings + clasificación por similitud a keywords.
  - Opción B (más flexible): LLM para extraer “necesidades” y mapear a keywords (con guardrails).
- **Ranking de resultados:**
  - Relevancia por solape (keywords/terapias), proximidad geográfica opcional y diversidad de profesionales.
  - Regla explícita: *no boost por plan* (solo límites de funcionalidades, no de ranking).
- **Explicabilidad:** mostrar “por qué” (p.ej. “relacionado con: insomnio, estrés”).
- **Evaluación:** conjunto de consultas de prueba + medición de precisión (manual al inicio) + logging.

**Salida:** buscador IA en producción con fallback a búsqueda clásica y trazas de calidad.

### Fase 6 — Reviews y métricas (MVP+ / v1) (1–2 semanas)
**Objetivo:** aumentar confianza y entregar valor a profesionales.
- Sistema de valoraciones (usuarios registrados) + moderación admin.
- Métricas básicas por profesional (vistas, clics en contacto, clics en calendario, clics en web).
- Panel simple para profesionales (dashboard).

**Salida:** confianza social + “valor Pro” medible.

### Fase 7 — Suscripciones y pagos (fase posterior) (1–2 semanas)
**Objetivo:** automatizar planes.
- Stripe: checkout + webhooks + estado de suscripción.
- Migración desde asignación manual a automática (manteniendo override admin).
- Facturación, emails y gestión de límites por plan (eventos/año, features).

**Salida:** monetización automatizada con auditoría.

### Fase 8 — Hardening, cumplimiento y lanzamiento (continuo; 3–7 días para cierre)
**Objetivo:** estabilidad, seguridad y cumplimiento.
- GDPR: política de privacidad, consentimiento, derechos ARCO, retención/borrado.
- Seguridad: control de acceso, protección de ficheros, auditoría admin, backups y restores.
- Observabilidad: dashboards, alertas, trazas, error reporting.
- QA: tests E2E críticos (registro profesional, verificación, búsqueda, IA, agenda).

**Salida:** checklist de release + go-live.

## 5) Entregables mínimos por “Release” (sugerencia)

**Release MVP (v0):**
- Directorio + perfiles + búsqueda clásica + mapa.
- Registro profesional + verificación por admin.
- Terapias + contenidos básicos.
- Agenda con límites por plan (asignación manual).
- IA orientación (versión 1) con explicabilidad y fallback.

**Release v1 (post-MVP):**
- Reviews moderadas + métricas básicas + dashboard profesional.
- Mejoras IA (calidad + diversidad + evaluación) y contenidos ampliados.

**Release v2:**
- Stripe + automatización de planes.
- Reservas/pagos (si se prioriza) + analítica avanzada (si se prioriza).

## 6) Decisiones tempranas (cerradas) y pendientes

### Cerradas (según confirmación)
- **Mapas:** Mapbox (definir límites de uso, alertas y control de costes).
- **Cuentas de usuario:** no hay cuenta para visitantes/usuarios en el MVP.
- **Verificación (MVP):** documento profesional (ID) + documento nacional (DNI/NIE/pasaporte).
- **Idiomas iniciales:** ES, CA, EN, DE (estructura i18n desde el inicio).

### Pendientes (para que la IA sea útil desde el día 1)
- **Mínimo de contenidos/taxonomía para IA:** definir un “dataset base” suficiente para que el buscador en lenguaje natural tenga buenas respuestas desde el primer lanzamiento, por ejemplo:
  - Lista inicial de terapias (p.ej. 30–60) con categoría.
  - Lista inicial de keywords/necesidades (p.ej. 80–200) normalizadas (sinónimos).
  - Relación terapia ↔ keyword (muchos-a-muchos) para sostener recomendación.
  - Contenido mínimo por terapia (p.ej. 1 ficha corta + 3–5 keywords principales) y 5–15 artículos “pilares” (estrés, ansiedad, insomnio, dolor, duelo, etc.).
  - Conjunto de 30–50 consultas reales de prueba para evaluar calidad (y mejorar iterativamente).

> Nota: si en el futuro se reintroducen reviews, con “sin cuenta de usuario” habría que elegir alternativa (p.ej. reseña verificada por email + moderación, o moverlo a fase posterior con cuentas).
