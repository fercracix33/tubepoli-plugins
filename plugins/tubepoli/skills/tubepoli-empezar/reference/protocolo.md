# Protocolo de las herramientas de tubepoli

## Envelope

Éxito: `{ ok: true, operation: {name, operationId?, replayed?}, status, data, revision, warnings, provenance, coverage, nextCursor, jobId }`.

- `status` (solo mutaciones): `changed` (hubo cambio y nueva revisión), `noop` (nada que cambiar; no es un error), `accepted` (se aceptó y sigue en segundo plano: mira `jobId`), `partial` (lote con resultados mezclados: revisa cada elemento).
- `revision`: la revisión nueva. Guárdala para la siguiente escritura del mismo objeto.
- `warnings`: avisos que debes contar al usuario si afectan a la decisión (hechos por comprobar, permisos pendientes, muestra pequeña…).
- `coverage`: `complete | partial | unknown | pending | unsupported | not_authorized | unavailable`. Todo lo que no sea `complete` se comunica como «incompleto» o «desconocido», nunca como cero.
- `nextCursor`: hay más páginas; pásalo como `cursor` si lo necesitas.
- `operation.replayed: true`: la respuesta viene del registro de idempotencia (el cambio ya estaba aplicado).

Error: `{ ok: false, error: {code, reason?, message, issues?, details?, retryAfterSeconds?} }`.

## Qué hacer con cada error

| Código | Qué significa | Qué hacer |
| --- | --- | --- |
| `VALIDATION_ERROR` | Entrada inválida (`issues[{field, message, limit}]`) o regla del método (`reason`). | Corrige el campo indicado. Si es una regla (p. ej. `selection_requirements_missing`, `promise_required`, `active_plan_exists`), explica la regla al usuario. Si choca con otro objeto (`duplicate`), `details` trae su id (p. ej. `details.conflict` con `planId` y `slotId`) y `resolveWith` las operaciones para resolverlo: propónselo al usuario y hazlo tú. |
| `REVISION_CONFLICT` | Alguien guardó antes (`details.currentRevision`). | Relee, muestra la diferencia y replantea. No sobrescribas. |
| `IDEMPOTENCY_CONFLICT` | Reusaste un `operationId` con otra entrada. | Usa un UUID nuevo para el cambio nuevo. |
| `NOT_FOUND_OR_FORBIDDEN` | No existe o no tienes acceso (`missing_capability`). | Dile al usuario qué permiso falta; se amplía al reconectar la IA en tubepoli. |
| `DEPENDENCY_NOT_READY` | Falta un paso previo (`details.missing`). | Haz el paso que falta. Solo si el usuario lo decide, usa la excepción documentada (`acceptMissing` + `exceptionReason` en `pipeline_move`). |
| `CAPABILITY_UNAVAILABLE` | El servidor no tiene ese proveedor (`reason`: `provider_not_configured`, `"media_too_large"`…). | Ofrece la vía manual (Studio, enlace externo, dato declarado con `analytics_import`). |
| `PROVIDER_AUTH_REQUIRED` | Falta conectar o ampliar permisos de Google (`scope_missing`). | La persona titular conecta YouTube desde la web de tubepoli (o con la URL de la operación indicada, si la tienes). Tú nunca recibes tokens. |
| `QUOTA_EXCEEDED` | Cuota de YouTube agotada (`retryAfterSeconds`). | Espera o trabaja con lo ya guardado (inventario sincronizado, referencias). |
| `RIGHTS_REQUIRED` | Material restringido o sin consentimiento. | No lo uses públicamente; pide el permiso o excluye el material. |
| `PROVIDER_OUTCOME_UNKNOWN` | Un envío externo puede haberse hecho o no. | No repitas a ciegas: pide comprobarlo en Studio y, si ya está publicado, vincúlalo con `publication_link_existing`. |

## Trabajos en segundo plano

Si la respuesta trae `jobId` (p. ej. `research_channels_sync`), el trabajo sigue en segundo plano. No esperes en bucle: di al usuario que está en marcha y vuelve a leer el objeto cuando haga falta (p. ej. `research_channels_get`).

## Lecturas útiles

- `capabilities_get` (con `domain`): qué operaciones existen en este conector y si te están permitidas.
- `operations_search` → `operations_describe` → `operations_call`: cualquier operación del catálogo completo desde este mismo conector (mismas reglas y respuesta que una herramienta directa).
- `context_get`: arranque compacto (≤16 KiB), con los planes abiertos (en curso, borradores y pausados con su id). `channels_context_get` con `format: "markdown"`: perfil completo.
- `strategy_get`: nivel, foco, fase, colchón y progreso del plan.
- `pipeline_list`: fichas por etapa (con su id y `revision`; filtra con `stage`, `q`, `blocked`…).
- `guides_list` / `guides_get`: estas guías.

## Efectos públicos (requieren confirmación explícita)

Herramientas marcadas «EFECTO PÚBLICO» en su descripción. En el núcleo: `business_forms_publish`. Antes de llamarla, enseña al usuario el cambio exacto y espera su «sí». Las demás (responder o moderar comentarios, pausar o archivar formularios, reintentar entregas, subir, programar o cambiar metadatos en YouTube) se usan vía `operations_call` y siguen la misma regla (`operations_describe` dice si una operación tiene efecto público).

## Avanzado (vía operations_call)

- Trabajos: `jobs_get` (`queued → running → succeeded|partial|failed`) y `jobs_cancel` (pide cancelar; no revierte un efecto externo ya hecho).
- Idempotencia: `operations_get` (resultado registrado de un `operationId`).
- Ficha completa con lo que le falta para avanzar: `videos_get`. Búsqueda y actividad: `space_search`, `activity_list`.
- Cuota: `budgets_get`. Conexión de Google: `channels_youtube_connect`, `analytics_authorize`. Estado real de un envío dudoso: `publication_reconcile`.
- Efectos públicos fuera del núcleo: `business_forms_pause`, `business_forms_archive`, `business_deliveries_retry`, `community_reply` (con `send: true`), `community_moderate`, `publication_upload`, `publication_schedule`, `publication_reschedule`, `publication_cancel_pending`, `publication_metadata_apply`.
