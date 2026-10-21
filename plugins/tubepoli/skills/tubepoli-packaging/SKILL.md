---
name: tubepoli-packaging
description: "Trabaja el packaging (título + miniatura) de un vídeo en tubepoli con el método AIC (atención, interés, curiosidad) y la brecha de información: 30 títulos para quedarse con 3, tres conceptos de miniatura radicalmente distintos, parejas comparadas con su promesa, elección versionada y pruebas de YouTube Studio (máximo 3 opciones). Úsala cuando el usuario diga «hazme títulos», «ideas de miniatura», «packaging», «¿qué título pongo?», «compara estas miniaturas», «sube esta miniatura» o «quiero hacer un test A/B»."
---

# Packaging (M06)

El packaging es la promesa del vídeo: decide si alguien hace clic y condiciona la intro. Va **antes** del guion: el packaging define la intro, no al revés.

## Cuándo usarla

- La idea está elegida y la ficha está en Packaging (o el usuario quiere probar el packaging de varias finalistas).
- Hay que adjuntar o cambiar una miniatura, o registrar una prueba.

## Flujo paso a paso

1. `packaging_get` (`videoId`): brief (gran recompensa, destinatario, enfoque), títulos, conceptos, parejas, selección vigente y pruebas, con sus revisiones. Si el brief está flojo, vuelve a `tubepoli-ideacion`.
2. **Títulos**: genera ~30 en el chat y quédate con los 3 mejores. Guarda solo las alternativas buenas con `packaging_titles_create` (`items` de 1–20: `text` ≤200, `intent`, `inspiration`, `reason`). Inspírate en fórmulas que tu audiencia ya clicó; si una exige una variable que no tienes, pregúntala, no la inventes.
3. **Miniaturas**: tres conceptos **radicalmente distintos**, no variaciones. Cada uno con `packaging_concepts_create` (`level: "concept"`): `description`, `protagonist`, `composition`, `visibleText`, `curiosity`, `needsPhoto`, `rights`. El concepto basta para seguir; el arte final puede llegar después de grabar.
4. **Parejas**: compara título + concepto en el chat (o enséñalas con `show_packaging`). Para las serias, `packaging_pairs_save` (`titleId`, `conceptId`, `expectedRevision: 0` si es nueva) con `changes`: `promise`, `audienceKnows` (lo que ya sabe), `wantsToKnow` (lo que quiere saber) y `aic.items` (`yes | doubt | no`; sin marcar = no revisado) — ver [reference/aic.md](reference/aic.md).
5. Presenta al usuario 2–3 parejas con su promesa y brecha. Él elige.
6. **Elegir y continuar**: `packaging_pairs_select` (`titleId`, `conceptId`) con `expectedTitleRevision`, `expectedConceptRevision`, `expectedPairRevision` (0 si no existe), `expectedSelectionVersion` (0 si no hay selección) y la promesa. Después `pipeline_move` (`toStage: "storytelling"`). Elegir **no** cambia nada en YouTube.
7. **Imagen** (tubepoli no genera imágenes): deja el concepto elegido y que la persona suba la imagen desde la web de tubepoli (ficha › Packaging). Si falta una foto del creador para la miniatura, `needsPhoto: true` la lleva a Producción.
8. **Variaciones** del concepto elegido (después, no antes): `packaging_concepts_create` con `level: "variation"` y `parentConceptId` (pequeñas variaciones del ganador, p. ej. el color de la camiseta, para exprimir lo que ya funciona).
9. **Pruebas** (tras publicar): la persona configura la prueba en YouTube Studio (hasta 3 opciones; tubepoli no la lanza). Acordad las opciones (pareja elegida y 1–2 alternativas guardadas) y, al acabar, registra los datos de Studio con `analytics_import` (`values` con `variant` y `abTestWatchShare`) y la conclusión como aprendizaje (`learning_record` con `kind: "packaging"` y el `videoId`).

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- Fórmulas del canal: `packaging_formulas_suggest` → `packaging_formulas_apply` (si falta una variable responde `variables_pending`).
- Archivar títulos descartados: `packaging_titles_update` (`archived: true`). Vista conjunta guardada: `packaging_pairs_compare`.
- Subir la imagen desde el chat (si tu cliente puede subir archivos): `assets_upload_init` (`kind: "image"`, `mimeType`, `sizeBytes`) → PUT a la URL firmada → `assets_upload_complete` → `packaging_assets_attach` con `file: {assetId}` y `provenance` (`source: "agent_client"`, `generated: true`, `tool`); fotos del creador con consentimiento: `packaging_library_list` y `file: {creatorPhotoId}`. `markFinal: true` solo con archivo verificado.
- Pruebas registradas en Packaging: `packaging_tests_record` (`kind: "studio_native"` con hasta 3 opciones, `editorial` o `external_feedback`) y `packaging_tests_close` (`decided | inconclusive | no_data`; en Studio, decidir exige `watchTimeContext`).

## Reglas del método

- **AIC**: llamar la **atención** (concepto claro, contraste, cara mirando a cámara, acción que «no debería pasar», elemento reconocible), apelar a un **interés** (se entiende de un vistazo de qué va; destacar una sola cosa: lo que el espectador desea) y crear **curiosidad**.
- **Brecha de información**: distancia entre lo que el espectador sabe y lo que quiere saber. Un título que ya responde («Envía 100 emails al día para vender más») la cierra; muestra el resultado, no el cómo.
- El título también da contexto a YouTube (impresiones): palabras claras de qué va el vídeo; un cambio de título puede recuperar impresiones.
- CTR es relativo al canal; ~7 % es un punto de partida orientativo para la mayoría de canales, no un umbral.
- La promesa del packaging se reafirma en los primeros segundos de la intro; si no cuadra, se pierde audiencia.
- Si la idea no se puede representar en una miniatura, se descarta y se pasa a la siguiente.
- Tres títulos × tres miniaturas son nueve combinaciones **editoriales**, no nueve opciones de una prueba nativa; YouTube Studio solo admite hasta 3 opciones por prueba.
- En una prueba, gana lo que da más **tiempo total de visualización** (CTR × retención), no solo el CTR.
- Inspírate en miniaturas y fórmulas de título que tu audiencia ya clicó, sin copiarlas.
- Desglosa CTR de espectadores nuevos y recurrentes en Studio cuando importe; entra como dato declarado (`analytics_import`).

## Errores que evitar

- Escribir el guion antes de tener la pareja elegida.
- Tres «miniaturas» que solo cambian colores (eso son variaciones).
- Títulos que responden la pregunta o prometen algo que el vídeo no cumple (`delivers: "no"`).
- Declarar ganadora una opción por CTR con pocas impresiones.
- Decir que tubepoli generará la imagen: la genera tu cliente o la persona.
- Cambiar título o miniatura en YouTube desde aquí: eso es Distribución y requiere confirmación.

## Ejemplos

- «Hazme títulos para este vídeo» → `packaging_get` → proponer 30 en el chat agrupados por ángulo → el usuario marca favoritos → `packaging_titles_create` con los 3–6 buenos.
- «Tres ideas de miniatura» → `packaging_concepts_create` ×3 (conceptos distintos) → `show_packaging` o comparación en el chat → `packaging_pairs_save` con promesa y AIC.
- «Me quedo con el título B y la miniatura 2, sigue» → `packaging_pairs_select` → `pipeline_move` a `storytelling` → «Pareja elegida (Packaging v1). ¿Escribimos el guion?».
