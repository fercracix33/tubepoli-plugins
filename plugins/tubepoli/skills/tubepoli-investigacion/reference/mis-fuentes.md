# Mis fuentes: señales, historias y autoridad (M17)

El método saca ideas e historias de lo que el creador vive cada semana: reuniones, llamadas, consultorías, problemas del equipo. tubepoli no lee tu calendario ni tu correo: si tu cliente de IA tiene esos conectores, tú extraes el fragmento útil y lo guardas aquí.

## Señales

1. `own_sources_capture` (hasta 20 por llamada en `items`): `phrase` (una frase basta), `kind` (`question | problem | objection | experience | feedback`), `problem`, `sourceType` (`call`, `meeting`, `ticket`, `comment`, `voice_note`, `agent_conversation`, `form`…), `sourceLabel`, `externalRef` (id del evento o ticket, para contar fuentes distintas y no duplicar), `occurredAt`, `personRole` (rol, no nombre), `bucketIds`. Guarda solo el fragmento útil, nunca la reunión entera. Emails o teléfonos detectados pasan a revisión.
2. Repeticiones: `own_sources_list` (`view: "grouped"` agrupa las repetidas; `view: "signals"` las lista una a una, con filtros `kinds`, `from`/`to`, `outcome`). La frecuencia cuenta **fuentes distintas** («5 llamadas»), no frases; no inventes representatividad.
3. Privacidad: una señal es contexto privado. No la cites en público (guion, descripción) sin el permiso explícito del usuario, y nunca con datos que identifiquen a la persona.
4. Convertir en idea: `ideas_capture_batch` con `items: [{workingTitle, origin, originRef: {type: "own_signal", id}}]` (`origin`: `client`, `question`, `experience`…) y `bucketId`. La idea conserva el enlace a su señal.

## Historias

- Guarda la experiencia como señal `kind: "experience"` con `note`: situación, conflicto, decisión, resultado y aprendizaje, y qué hechos son verificables.
- Una historia propuesta por ti no es una experiencia real: **nunca** la guardes como tal; eso lo decide la persona.
- En el guion, las historias van como pruebas del bloque (`scripts_blocks_update` → `proofs.add` con `sourceKind: "own_experience"` y `verification: "declared"` salvo evidencia), solo con permiso público y priorizando las verificables y parecidas al cliente ideal, porque quien ve el vídeo se identifica más con ellas.

## Autoridad

Frase recurrente de presentación, credenciales, resultados demostrables, esfuerzo de investigación o postura (experto o aprendiz) por tema: con el usuario, añádelo al contexto del canal (`channels_context_update` → `bodyMarkdown`, sección «Autoridad», con sus pruebas). Alimenta la «prueba» de la intro.

## Revisión semanal de fuentes

Cada semana: captura los fragmentos de la semana (`own_sources_capture`), repásalos con el usuario (`own_sources_list` con `from` = inicio de la semana) y decide por señal: idea (`ideas_capture_batch`), historia (señal `experience` con su `note`), usable más adelante o descartar. Es distinta de la revisión semanal de métricas (ver `tubepoli-rutinas`).

## Avanzado (vía operations_call)

- Importar señales ya minimizadas de leads o pilotos: `own_sources_import` (`origin.module`: `m16_leads`, `f52_pilot`, `external`).
- Consolidar repeticiones en un objeto propio: `own_sources_consolidate` con un `criterion` explícito.
- Privacidad y permiso formales: `own_sources_review` (`captured → needs_review → usable | restricted | archived`; `usable` exige nota de revisión) y `own_sources_permissions_set` (`private_context`, `public_anonymized`, `public_identifiable` con evidencia).
- Idea desde señal o consolidación: `own_sources_convert_to_idea` (`from`, `workingTitle`, `bucketId`, `contribution`); si queda `pending_pipeline`, `own_sources_ideas_materialize`.
- Banco de historias: `stories_create` (`situation`, `conflict`, `decision`, `result`, `learning`, `facts[]` con certeza `verified | plausible | unverified`, `origin`: `real_experience`, `client_case`, `composite`, `agent_generated`; una historia tuya es `agent_generated`) y `scripts_stories_suggest` para usarlas en un guion.
- Autoridad estructurada: `authority_record` y `authority_list`.
- Fotos del creador: `photos_attach` (archivos subidos con `assets_upload_init`/`assets_upload_complete`) con consentimiento por uso (`thumbnails`, `packaging`, `ai_variations`); las variaciones por IA solo con `ai_variations` y nunca son evidencia de hechos.
- Revisión semanal formal: `own_sources_weekly_review_prepare` (ventana por defecto: semana anterior) → decisión por señal (`idea`, `story`, `usable`, `discard`, `keep`) → `own_sources_weekly_review_close`.
