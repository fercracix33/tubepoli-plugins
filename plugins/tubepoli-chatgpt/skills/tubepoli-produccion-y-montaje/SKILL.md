---
name: tubepoli-produccion-y-montaje
description: "Organiza en tubepoli la grabación y el montaje de un vídeo según el método: tomas derivadas del guion elegido, sesiones de grabación por lotes (grabar el doble de lo que se publica), confirmación de grabación, entrega al editor, versiones de montaje con comentarios por tiempo y capa, revisión por bloques (montaje y música antes que gráficos y efectos) y elección del archivo final. Úsala cuando el usuario diga «prepara la grabación», «lista de tomas», «sesión de grabación», «ya grabé», «pásaselo al editor», «revisa este montaje», «comentarios al editor» o «este es el final»."
---

# Producción y postproducción (M08 + M09)

Lo que pasa después de grabar pesa menos en el resultado que idea, packaging y guion: un buen guion mal editado funciona; un mal guion bien editado, no. Aquí el objetivo es ejecutar rápido lo preparado y montar para que se entienda y retenga.

## Cuándo usarla

- La ficha tiene guion elegido (`scripts_select`) y va a grabarse.
- Llegan archivos grabados o montajes del editor.
- Hay que decidir el archivo final.

## Flujo paso a paso — Producción

1. `production_get` (`videoId`): guion elegido y bloques, tomas, materiales, comprobaciones y qué falta para una entrega completa.
2. Qué grabar: los bloques del guion elegido y sus recursos declarados (`resources`: plano, demo, pantalla, **foto para la miniatura**). Lo que falte se añade al bloque con `scripts_blocks_update` → `resources`. Repásalo con el usuario como lista de tomas.
3. Lotes: agrupa con el usuario los vídeos que se graban el mismo día y en el mismo set (mira en `pipeline_list` con `stage: "production"` cuáles están listos) y anota fecha, lugar y lo imprescindible: audio, foco, iluminación; en un screencast, pantalla y micro.
4. Durante o después de grabar, con datos que te da el equipo: archivos con `production_materials_attach` (`items`: `externalUrl` autorizado —p. ej. la carpeta de Drive— o `assetId` si la persona lo subió a tubepoli; `kind`: `original`, `audio`, `photo`…; `label`). Un archivo que aún se sube no cuenta como recibido.
5. Confirmación: `production_confirm_recorded` (`coverage: "complete"` o `"partial"`; con `evidence` si la registras tú; `complete` exige que no queden tomas pendientes o por repetir en la lista de tomas, si la hay). Es la base del colchón; una sesión completada no cuenta como vídeo grabado.
6. Entrega al editor: `production_handoff_prepare` (`videoId`, `instructions` con las indicaciones: gráficos, énfasis, foto de miniatura) → enseña al usuario qué va y qué falta → `production_handoff_share` (`handoffId`, `expectedRevision`, `recipient` si es otra persona del espacio; `acceptPartial: true` solo si el usuario acepta entregar incompleto). Después `pipeline_move` (`toStage: "postproduction"`): sin entrega compartida responde `DEPENDENCY_NOT_READY`.
7. Colchón: `strategy_get` (colchón en semanas según el ritmo).

## Flujo paso a paso — Postproducción

1. `cuts_register` por cada versión de montaje (`videoId`, `assetId` o `externalUrl` + `durationSeconds`, `label`). Cada versión es nueva; no hereda comentarios.
2. Revisión por bloques en este orden: intro → ejecución → conexión → resto → vídeo completo; y por capas: primero montaje (`edit`), luego música (`music`), luego visuales y sonido.
3. Comentarios concretos: `cuts_comments_add` (`cutId`, `text`) con tiempo **de ese montaje** (`startSeconds`/`endSeconds`), `layer` (`edit | music | visuals | sound | general`), `kind` (`problem | intent | question | praise`) y `blockId`. El resultado de cada bloque («aprobado» o «cambios») va como comentario `kind: "intent"` o `"praise"`.
4. Nueva versión del editor: `cuts_register` y vuelve a comentar lo que siga vigente con el **nuevo** tiempo (los tiempos de un montaje no valen para otro).
5. Final: `cuts_list` (`videoId`) da cada versión con su `revision` (cambia con cada comentario) y el final vigente. Con el visto bueno del usuario, `cuts_final_select` (`cutId`, `expectedCutRevision`, `expectedFinalId`: el id del final vigente o `null`). Después `pipeline_move` (`toStage: "distribution"`).
6. Fotograma para la miniatura: la persona lo exporta del montaje y lo sube desde la web (Packaging).

## Avanzado (vía operations_call)

- Tomas: `production_takes_derive` (una por bloque y una por recurso declarado; repetirlo no duplica), `production_takes_update` (`status`: `recorded` exige `evidence` si lo haces tú; `not_applicable` exige `reason`) y `production_takes_add` (toma suelta).
- Sesiones por lotes: `production_session_create` (`name`, `videoIds` hasta 20, `localDate`, `startTime`, `location`, `needs`) y `production_sessions_list`.
- Producción: `production_checks_set` (`template`: `studio`, `screencast`, `podcast`), `production_notes_add` (indicaciones con `atSeconds`), `production_materials_update`.
- Colchón detallado: `production_buffer_get` o `strategy_buffer_get`.
- Postproducción: `cuts_get`, `cuts_review` (`scope`: `intro | block | full`; `layer`; `result`), `cuts_comments_list`, `cuts_comments_resolve`, `cuts_comments_migrate` (traslada comentarios a la versión nueva con su **nuevo** tiempo), `cuts_maps_set` (bloque → tiempo, útil para leer la retención), `cuts_export` (Markdown para un editor externo) y `media_still_extract` (`atSeconds`, si el servidor tiene FFmpeg).

## Reglas del método

- **Graba como estés cómodo.** Imprescindibles: audio, foco e iluminación; la cámara no lo es.
- **La intro se graba con guion palabra por palabra**; el resto puede improvisarse dentro de lo pautado (conflicto, contexto, reenganches).
- **Grabar el doble de lo que publicas** para tener uno o dos meses de colchón; sesiones por lotes en un día fijo.
- **Refuerzo visual en la grabación** (folios, pizarra, pantalla) en vez de animaciones caras en edición.
- **El editor cuenta la historia**, no embellece; su misión es que la gente vea el vídeo más tiempo.
- **Intro**: se entiende sin sonido (subtítulos), visualmente parecida a la miniatura, cambios visuales frecuentes (≈cada 2 s como referencia, no norma).
- **Por bloques y por capas**: primero el montaje, luego la música, luego gráficos y efectos de sonido. Montaje y música son lo que más pesa; gráficos y efectos son extras. Casi siempre hay más que quitar que añadir.
- **Transiciones que no se notan**.
- **Revisión en dos pasadas**: primer feedback sobre montaje e intro, revisión final y aprobado.

## Errores que evitar

- Marcar como grabado o recibido algo que nadie ha confirmado.
- Copiar tiempos de comentarios de un montaje a otro como si fueran iguales.
- Revisar gráficos antes de que el montaje y la música estén aprobados.
- Elegir el final sin el usuario o cambiar un final ya publicado sin avisar (marca para revisión el paquete de publicación).
- Retrasar vídeos esperando una cámara o un editor mejores.

## Ejemplos

- «Prepara la grabación del viernes con estos tres vídeos» → `production_get` ×3 → resumen de bloques, recursos y necesidades del lote.
- «Ya grabé, los archivos están en Drive» → `production_materials_attach` (`externalUrl`) → «¿Confirmo la grabación como completa?» → `production_confirm_recorded` (con `evidence`) → `production_handoff_prepare` → `production_handoff_share` → `pipeline_move` a `postproduction`.
- «Revisa el montaje v2: la intro es lenta en 0:12–0:20 y la música del bloque 2 no pega» → `cuts_comments_add` ×2 (capa `edit` y `music`, con sus tiempos de la v2).
