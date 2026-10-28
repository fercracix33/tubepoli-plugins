---
name: tubepoli-montaje
description: "Monta en tubepoli el vídeo de una ficha desde la conversación: proyecto de montaje sobre los materiales grabados o un corte, propuesta de cortes (silencios, muletillas, repeticiones, mejor toma), edición por operaciones con revisión (dividir, recortar, mover, quitar con ripple, música, rótulos, subtítulos), preview, revisión visual con fotogramas y hoja de contactos, corrección y render final que llega a Postproducción como un montaje más. Úsala cuando el usuario diga «móntame el vídeo», «quita los silencios», «corta las muletillas», «haz un primer montaje», «pon música», «añade un rótulo», «subtítulos», «enséñame cómo va el montaje» o «renderiza el final»."
---

# Montaje en tubepoli (M19)

El montaje cuenta la historia: su misión es que la gente vea el vídeo más tiempo. Aquí el primer montaje lo haces tú con el usuario, por operaciones que quedan en un historial y se pueden deshacer; el resultado se revisa como cualquier montaje de Postproducción.

## Cuándo usarla

- La ficha tiene materiales grabados recibidos (Producción) o un montaje con archivo en tubepoli (Postproducción).
- El usuario quiere un primer montaje rápido, quitar silencios y muletillas, ajustar cortes, añadir música, rótulos o subtítulos, o ver cómo va.
- Para revisar el montaje de un editor externo usa `tubepoli-produccion-y-montaje` (comentarios por tiempo y capa).
- Para intros, datos o explicaciones animadas a medida (escenas de motion graphics con storyboard) usa `tubepoli-motion`.
- **Siempre junto con `tubepoli-retencion`**: ritmo por tramos, material sin repetir, lectura, música y la crítica P0/P1/P2 antes de terminar. Si no hay grabación (locución + material), su modo «sin grabación» va primero: lista de planos y presupuesto de material antes de montar.

## Antes de empezar

1. `production_get` (`videoId`): guion elegido, bloques y materiales. Sin materiales recibidos no hay nada que montar: dilo y propón subirlos.
2. Lee `tubepoli-retencion` (baremos y, si no hay cara, «sin grabación»): sin lista de planos y material suficiente no se empieza a montar.
3. Confirma con el usuario **qué** se monta (todas las tomas en el orden del guion o un montaje concreto) y **qué quiere** en esta pasada: por el método, primero el montaje (qué se queda y en qué orden) y la música; gráficos y efectos después.
4. Si la ficha aún no tiene proyecto de montaje, créalo con la operación de creación del apartado Avanzado (desde los materiales o desde un corte). Si ya existe, úsalo: un vídeo puede tener varios, pero no los dupliques sin motivo.

## Flujo paso a paso

1. **Leer** — `edit_timeline_get` (`projectId`): clips por pista (`id`, `kind`, `start`, `end`, `track`, `mediaStart`…), duración, avisos (solapes, huecos) y la `revision` vigente. Pistas por convención: 0 vídeo, 5 B-roll, 10 audio, 20 texto.
2. **Transcribir y proponer** — con las operaciones de «Transcripción» y «Sugerencias» del apartado Avanzado: si no hay transcripción lista, pídela (o, sin proveedor de transcripción, pide al usuario un SRT/VTT e impórtalo); después pide sugerencias de silencios, muletillas, repeticiones y falsos comienzos (y mejor toma si hay varias tomas de un bloque del guion). Enséñaselas agrupadas, una línea por propuesta (tiempo, texto y motivo). Sin transcripción, propón tú los cortes a partir de lo que el usuario cuente.
3. **Aplicar** — las sugerencias aprobadas se aplican en una sola revisión y las descartadas se rechazan (Avanzado, «Sugerencias»); para cortar por texto, tachando palabras, también hay operación propia (Avanzado, «Transcripción»). El resto de ediciones, con `edit_ops_apply` (`projectId`, `expectedRevision`, `ops`, `label`), todo-o-nada:
   - `clip.split {clipId, at}` · `clip.trim {clipId, start?, end?, ripple}` · `clip.move {clipId, to | afterClipId, track?, ripple}` · `clip.delete {clipId, ripple}` (con `ripple: true` lo posterior avanza y no quedan huecos).
   - `clip.add {assetId, at, duration?, mediaStart?, track?, insert?}` (música en la pista 10, B-roll en la 5), `clip.set {clipId, volume, fadeIn, fadeOut}`.
   - `text.add {text, at, duration, preset: lower_third | title | caption | plain}` y `text.update` (texto simple). Para rótulos animados con la marca del canal, subtítulos, B-roll y transiciones, ver Avanzado, «Gráficos, B-roll y sonido».
   - Para un cambio grande, primero `dryRun: true`: devuelve el antes/después sin guardar; enséñalo y aplica después.
   - Los tiempos son de la **composición** (lo que se ve), no del archivo original; vuelve a leer la línea de tiempo tras cada lote.
   - Cada respuesta trae `craft` (resumen de oficio: repetición, estáticos, planos largos, lectura…). Con P0 o P1, corrígelo antes de seguir; el detalle con arreglos, en `edit_craft_check`. Las imágenes entran con deriva suave por defecto (`motion: "none"` para dejarlas quietas) y un vídeo ya usado entra desde un tramo libre.
4. **Preview** — `edit_render_request` (`kind: "preview"`, 540p; `range: [inicio, fin]` para revisar solo un tramo): devuelve `render.id` y `nextStep`. Di que está en marcha y lee el resultado con `edit_renders_get` (`renderId`, `waitSeconds: 55`: el servidor espera a que termine; si sigue en curso, repite — no sondees en bucle): vídeo, métricas (sonoridad, picos, negros, congelados) y hallazgos con `fixHint`. Si responde `render_worker_not_configured`, el montaje se puede editar pero no renderizar: dilo. Cuando el preview esté listo, **pásale al usuario `render.appUrl`** (enlace de tubepoli que no caduca y abre ese render en el Montaje); nunca le pases `videoUrl`, que es una URL firmada que caduca en minutos. Si quiere comentarlo por tiempo en Postproducción, envíalo a revisión (Avanzado, «Verificación»).
5. **Revisar** — `edit_craft_check` (`detail: true`) y la crítica P0/P1/P2 de `tubepoli-retencion` sobre la hoja de contactos del montaje completo. `show_editor` (`videoId` o `projectId`) enseña al usuario el preview, los fotogramas, la línea de tiempo, la transcripción con los cortes tachados y las sugerencias pendientes. Para verlo tú, pide la hoja de contactos de los cortes o fotogramas concretos (Avanzado, «Verificación»): llegan como imágenes con su descripción. Comprueba: cortes limpios (sin palabras a medias), intro visualmente parecida a la miniatura, rótulos legibles y fuera de la zona de botones, sonido sin saltos.
6. **Corregir** — vuelve al paso 3 con lo que falle. Si algo quedó peor, deshazlo (historial, Avanzado) en lugar de rehacerlo a mano.
7. **Final** — sin P0 ni P1 pendientes y con el visto bueno del usuario, `edit_render_request` (`kind: "final"`, 1080p con los originales): al terminar, tubepoli registra el archivo como un **montaje nuevo en Postproducción** (`cuts_list` lo muestra con su versión). La elección del final sigue siendo explícita: con el usuario, `cuts_final_select` (`cutId`, `expectedCutRevision`, `expectedFinalId`) y después `pipeline_move` a `distribution`. Nada se publica desde aquí.

## Revisiones y conflictos

- Toda edición lleva `expectedRevision` (la `revision` que leíste). Si el usuario editó en la web o hubo otra edición tuya, responde `REVISION_CONFLICT`: vuelve a leer con `edit_timeline_get`, rehaz el plan sobre lo vigente y **no pises** su cambio.
- Un clip con `locked: true` no se toca: el usuario lo ha protegido.
- Cada `edit_ops_apply` es una revisión con su `label`: pon etiquetas que el usuario entienda («Quitar silencios del bloque 2»).

## Avanzado (vía operations_call)

- Proyecto: `edit_project_create` (`operationId`; `source: {type: "materials"}` con `videoId`, `{type: "cut", cutId}` o `{type: "assets", assetIds}`; `durations` si tubepoli no conoce la duración de un archivo), `edit_projects_list`, `edit_project_get` (revisión, archivos con su estado de preparación, renders, sugerencias pendientes, lint).
- Atajos de edición: `edit_clips_split`, `edit_clips_trim`, `edit_clips_move`, `edit_clips_delete`, `edit_clips_add`, `edit_audio_add` (música con volumen y fundidos), `edit_variables_set`.
- Transcripción y edición por texto (paso 2–3): sin transcripción lista, `edit_transcript_request`; si responde `asr_not_configured`, `edit_transcript_import` con el SRT/VTT del usuario. Cortar o reponer por texto: `edit_transcript_cut` (`mode: "cut" | "restore"`, `words: [{transcriptId, wordIds}]`, `expectedRevision`). Además: `edit_transcript_request` (`operationId`, `language` opcional), `edit_transcript_import` (`assetId`, `format: srt | vtt | words`), `edit_transcript_get` (palabras con `at` en la composición o `null` si están cortadas; `byClip: true` para el texto por clip), `edit_transcript_cut`.
- Sugerencias (paso 2–3): `edit_suggestions_propose` (`kinds`: `silence`, `filler`, `repeat`, `false_start`, `best_take`), `edit_suggestions_list`, `edit_suggestions_apply` (`ids` o `all`, `expectedRevision`, `dryRun`), `edit_suggestions_reject` (`ids`).
- Gráficos, B-roll y sonido (paso 3): rótulos animados con la marca con `edit_text_add` (`blockId` de `edit_blocks_list`, `variables`, `at`, `duration`). Además: `edit_blocks_list`, `edit_text_add`, `edit_text_update`, `edit_broll_add`, `edit_transitions_set`, `edit_captions_generate`, `edit_audio_ducking_set`, `edit_templates_list`, `edit_templates_apply`; marca del canal con `channels_brand_get` / `channels_brand_set`; stock con `edit_stock_search` / `edit_stock_import` y archivos por enlace con `edit_assets_import_url` (solo con licencia que permita el uso).
- Verificación (pasos 4–5 y 7): el resultado de cualquier render está en `edit_renders_get` (`renderId`): `pending`, avance, `render.appUrl` (el enlace que se comparte con el usuario; `videoUrl` solo sirve para reproducir y caduca), métricas y los `findings` más graves con `fixHint`. No sondees: pasa `waitSeconds: 55` y el servidor espera a que termine. Las imágenes de verificación solo llegan con `includeImages: true`; `full: true` da todos los hallazgos. Además: `edit_snapshot` (`at`), `edit_contact_sheet` (`mode: review | grid`), `edit_compare` (`revisionA`, `revisionB`, `at`), `edit_check` (composición; con `renderId` de un vídeo terminado, además sonoridad −14 LUFS, picos, negros y congelados). Todas devuelven `render.id`: el resultado está en `edit_renders_get`. Para llevar un preview a Postproducción como montaje para comentar y poder elegirlo como final: `edit_render_promote` (`renderId`, `operationId`; idempotente). `edit_renders_latest` (`videoId`) da el último preview y final de cada proyecto de la ficha. También `edit_renders_list`, `edit_render_cancel` y `edit_assets_derive` (rehacer copia ligera, audio y picos de un archivo).
- Historial: `edit_history_list` (quién hizo qué), `edit_history_undo` (deshace una revisión; si algo cambió después, avisa en vez de deshacer a ciegas), `edit_history_restore`.
- Fuente (modo experto): `edit_source_get`, `edit_source_put` (HTML de la composición con la revisión esperada; sin scripts externos ni red).

## Reglas del método

- **Primero el montaje, luego la música**, después gráficos y efectos: son lo que más pesa. Casi siempre hay más que quitar que añadir.
- **La intro decide**: se entiende sin sonido (subtítulos), se parece a la miniatura y cambia de plano con frecuencia (≈ cada 2 s como referencia, no norma).
- **Nada de relleno**: si falta material para cubrir la voz, para y pide más; nunca repitas clips ni alargues imágenes quietas (baremos en `tubepoli-retencion`).
- **Transiciones que no se notan**; cortes en el silencio entre frases, nunca a mitad de palabra.
- **Música por debajo de la voz** (con ducking si está disponible) y sonoridad final cerca de −14 LUFS.
- **Rótulos cortos**: se leen de un vistazo; mejor un refuerzo visual en la grabación que una animación cara.
- El usuario decide: propones, enseñas y aplicas con su visto bueno; el final lo elige él.

## Errores que evitar

- Editar sin la `revision` vigente o reintentar un `REVISION_CONFLICT` con la misma revisión.
- Quitar con `ripple: false` en la pista de vídeo (deja huecos negros) sin que el usuario lo pida.
- Pedir el render final antes de revisar un preview, o elegir el final sin el usuario.
- Copiar tiempos de un montaje a otro: cada render es un montaje nuevo con sus tiempos.
- Insertar música, B-roll o stock sin derechos claros.
- Dar el montaje por terminado con avisos P0/P1 de `edit_craft_check` o sin haber mirado la hoja completa.

## Ejemplos

- «Móntame el vídeo con lo que grabé ayer y quita los silencios» → `production_get` → crear el proyecto desde los materiales (Avanzado) → sugerencias de silencios → resumen al usuario → `edit_ops_apply` con los cortes aprobados → `edit_render_request` (`preview`) → `show_editor`.
- «La toma 3 sobra y pon el bloque de la demo antes del caso» → `edit_timeline_get` → `edit_ops_apply` (`clip.delete` con `ripple` y `clip.move` con `afterClipId`, `dryRun` primero) → confirmación → aplicar.
- «Añade un rótulo con mi nombre al principio» → plantillas de rótulo inferior y rótulo con nombre y cargo en `at: 2`, `duration: 4` (Avanzado, «Gráficos») → fotograma en 3 s para comprobarlo (Avanzado, «Verificación») → `show_editor`.
- «Ya está, renderiza el final» → `edit_render_request` (`final`) → resultado del render cuando deje de estar pendiente (Avanzado, «Verificación») → `cuts_list` → «¿Lo marco como archivo final?» → `cuts_final_select`.
