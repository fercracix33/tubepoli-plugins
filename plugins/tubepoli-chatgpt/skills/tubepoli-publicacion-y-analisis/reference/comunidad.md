# Comunidad: bandeja de comentarios del canal

La bandeja es una copia local: leerla no gasta cuota de YouTube; sincronizarla sí (1 unidad por página). Responder, comentar, editar, borrar y moderar tienen **EFECTO PÚBLICO**: enseña al usuario el texto o la lista exacta y espera su «sí» por acción o por lote.

## Avanzado (vía operations_call)

Ninguna de estas operaciones es una tool del núcleo: llámalas con `operations_call`.

### Flujo

1. **Sincronizar** si `community_inbox_list` devuelve `sync.stale: true` o nunca se sincronizó: `community_inbox_sync` (`operationId`; incremental por defecto, se detiene en el último comentario visto; `full: true` la primera vez; `youtubeVideoId` para un solo vídeo; `maxPages` 1–10). Devuelve `fetched`, `created`, `updated` y `unitsUsed`. Si avisa `sync_partial`, repite más tarde; `moderation_queues_unavailable` = la conexión no permite leer retenidos/spam.
2. **Leer**: `community_inbox_list` (`view`: `needs_reply` por defecto — preguntas, posibles clientes y negativos sin respuesta del canal —, `leads`, `questions`, `negative`, `held`, `spam`, `open`, `done`, `all`; `q`, `sort: "priority"`, `limit` ≤ 50, `cursor`). La prioridad es una regla fija por palabras (no IA): revísala tú antes de actuar y corrígela con `community_triage_set` (`priority`).
3. **Abrir el hilo**: `community_thread_get` (`commentId`) da todas las respuestas, la ficha del vídeo, el historial de acciones y la URL vigente de imágenes/GIF (caduca; no la guardes).
4. **Responder**: redacta y guarda borrador con `community_reply` (`commentId` del hilo, `text`, `send: false`; `videoId` opcional). Enséñalo; con el «sí», reenvía con `send: true`, `actionId` y `expectedRevision` del borrador. Plantillas: `community_templates_list` / `community_templates_save` (variables `{nombre}`, `{canal}`, `{video}`); personaliza cada una, no pegues la misma respuesta a muchos.
5. **Comentario del canal** en un vídeo (CTA del paquete): `community_comment_post` (`videoId`, `fromPackage: true` o `text`; `send` tras el «sí»). Fijarlo es una tarea de Studio.
6. **Moderar en lote**: primero `community_moderate_batch` con `dryRun: true` (`commentIds` ≤ 50, `status`: `published` aprueba, `heldForReview` retiene, `rejected` oculta); enseña `changes` y `skipped`; con el «sí», la misma llamada sin `dryRun` y un `operationId` nuevo. `banAuthor: true` solo con `rejected` (si no → `ban_requires_rejected`) y solo si el usuario lo pide expresamente.
7. **Editar/borrar lo propio**: `community_comment_edit` / `community_comment_delete` (solo comentarios del canal; otros → `not_own_comment`, usa moderación).
8. **Cerrar el trabajo**: `community_triage_set` (`commentIds` ≤ 100, `triage: "done" | "ignored" | "open"`). Una pregunta que merece vídeo: `community_question_to_idea` (`commentId`, `workingTitle`) → idea en Ideación y comentario marcado como hecho. Un posible cliente: sigue `tubepoli-negocio-y-leads`.

### Errores frecuentes

- `comment_not_in_inbox` → sincroniza (`community_inbox_sync`) o pasa `videoId` de la ficha.
- `PROVIDER_AUTH_REQUIRED` / `scope_missing` → el canal debe reconectar YouTube con permiso de gestión de comentarios desde Ajustes; mientras, solo borradores.
- `REVISION_CONFLICT` al enviar un borrador → vuelve a leer el hilo y usa la revisión vigente.
- Un envío queda `sending` hasta que YouTube confirma; `outcome_unknown` → revisa el hilo antes de reintentar.
