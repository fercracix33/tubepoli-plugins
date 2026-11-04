# Paquete de publicación (subida directa o vía Studio)

`publication_prepare` guarda; `publication_validate` comprueba el paquete **exacto** que se exportará; `publication_export` fija esa versión y devuelve el texto. Cualquier cambio posterior invalida la validación.

## Campos del paquete (`changes`)

| Campo | Notas |
| --- | --- |
| `title` / `pairRef` | Título de la pareja elegida en Packaging (`packaging_get` → selección vigente). |
| `thumbnailAssetId` | Archivo verificado de la miniatura (JPG/PNG ≤ 50 MB; en Shorts se aplica cuando YouTube termina de procesar). |
| `description` | Primera línea: enlace del CTA (lead magnet o vídeo central). Luego contexto breve. No hace falta rellenarla de palabras clave. |
| `chapters` | Del corte final: desde 00:00, al menos 3, de ≥10 s. |
| `tags`, `categoryId`, `defaultLanguage` | Opcionales. Con los catálogos de YouTube en caché (ver «Avanzado»), la categoría debe ser asignable y el idioma conocido por YouTube. |
| `playlist` / `playlistYoutubeId`, `playlistPrivacy` | Nombre de la lista (se busca por título y, si no existe, se crea con esa visibilidad) o su ID. |
| `localizations` | Traducciones `{"en": {"title", "description"}}` (≤ 30; no repitas `defaultLanguage`). |
| `captionsAssetId`, `captionsLanguage`, `captionsName`, `captionsDraft` | Subtítulos SRT/VTT (≤ 100 MB); se suben tras el procesado. |
| `brandPartner` | `{channelHandle: "@marca"}` o `{channelId: "UC…"}`; solo con promoción pagada. |
| `notifySubscribers`, `embeddable`, `license`, `publicStatsViewable` | Ajustes de YouTube (por defecto: avisar, insertable, licencia estándar, vistas públicas). |
| `visibility`, `mode` | `mode: "manual"` para publicar a mano en Studio; si se programa, `scheduledLocalDate`, `scheduledLocalTime`, `scheduledTimeZone` (fecha futura). |
| `madeForKids`, `containsSyntheticMedia` | Declaraciones del canal: deben ser explícitas (pregunta al usuario). |
| `cta`, `conversionUrl`, `pinnedComment` | Texto del CTA y comentario a fijar (con el enlace del vídeo, creado en `business_forms_links_create`). |
| `nextVideoId`, `nextVideoReason` | Vídeo al que lleva la pantalla final (cola de reproducción). |
| `adoptCurrentFinal: true` | Si el final cambió después de preparar el paquete. |

## Enlaces atribuibles por vídeo

Antes de escribir la descripción, comprueba `business_ctas_get` (`videoId`): trae el enlace de ese vídeo si existe. Si no, créalo en `tubepoli-negocio-y-leads` (`business_forms_links_create` con `placement: "description"` y otro con `placement: "pinned_comment"`).

## Después de publicar

1. `publication_link_existing` con la URL o ID. Con conexión lee el estado real; sin ella se guarda como declarado (pendiente de verificar).
2. Tareas de Studio (pantalla final, comentario fijado, prueba de título/miniatura) y promoción (newsletter, piezas derivadas, comunidad): repásalas con el usuario a partir del texto exportado.
3. Recortar algo del vídeo publicado (p. ej. la intro) se hace en Studio; si cambia el montaje, regístralo como versión nueva (`cuts_register`).

## Avanzado (vía operations_call)

- Estado real tras publicar: `publication_reconcile` (privado, programado, público, procesando).
- Tareas como objetos: `distribution_tasks_prepare` sin `items` crea las de Studio a partir del paquete; con `items` para promoción (`kind: "promotion"`, `"derived_piece"`, `"community_post"`…), prueba A/B (`"ab_test"`) o cambio del montaje publicado (`"cut_change"`). Confirmar: `distribution_tasks_update` (`status: "confirmed"` con `evidence`).
- Subida directa (activada por defecto; cada envío exige un «sí» explícito para ese vídeo): `publication_upload` (`dryRun: true` → `plan.calls` con método, bolsa y coste; `plan.blockers`), `publication_schedule`, `publication_reschedule`, `publication_cancel_pending` y `publication_metadata_apply` (`fields` incluye `localizations` y `settings`). Requiere el permiso `youtube.force-ssl` y `publication_settings_get` → `nativeUpload` (el espacio puede desactivarla).
- Cuota de YouTube por bolsas: cada subida de vídeo cuenta 1 en la bolsa de subidas (100/día); cada búsqueda, 1 en la de búsquedas (100/día); el resto va en unidades generales (10.000/día): miniatura y añadir a lista 50, subtítulos 400, actualizar metadatos 50, lecturas 1. `budgets_get` y `publication_get` → `quota` muestran lo que queda.
- Errores con `reason` y qué hacer: `scope_missing` (reconectar con permiso de gestión), `native_upload_disabled` (vía Studio o activarla en ajustes), `package_not_validated` (validar), `external_commitment_pending` (reconciliar, no reintentar), `processing_timeout`/`youtube_processing_failed` en un paso (reconciliar con `retrySteps`), `youtube_quota_exceeded` (esperar `retryAfterSeconds` o vía Studio).
