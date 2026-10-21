# Paquete de publicación vía Studio

`publication_prepare` guarda; `publication_validate` comprueba el paquete **exacto** que se exportará; `publication_export` fija esa versión y devuelve el texto. Cualquier cambio posterior invalida la validación.

## Campos del paquete (`changes`)

| Campo | Notas |
| --- | --- |
| `title` / `pairRef` | Título de la pareja elegida en Packaging (`packaging_get` → selección vigente). |
| `thumbnailAssetId` | Archivo verificado de la miniatura (JPG/PNG ≤ 2 MB). |
| `description` | Primera línea: enlace del CTA (lead magnet o vídeo central). Luego contexto breve. No hace falta rellenarla de palabras clave. |
| `chapters` | Del corte final: desde 00:00, al menos 3, de ≥10 s. |
| `tags`, `categoryId`, `defaultLanguage`, `playlist` | Opcionales. |
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
- Vía nativa (no usar por defecto): `publication_upload`, `publication_schedule`, `publication_reschedule`, `publication_cancel_pending` y `publication_metadata_apply` tienen efecto público en YouTube y dependen de `publication_settings_get` → `nativeUpload` (desactivado por defecto), del permiso `youtube.force-ssl` y de que el proyecto de Google esté auditado. Si el usuario lo pide expresamente, explica estas condiciones y confirma cada acción.
