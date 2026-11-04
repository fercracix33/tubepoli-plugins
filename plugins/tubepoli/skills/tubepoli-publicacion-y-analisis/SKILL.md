---
name: tubepoli-publicacion-y-analisis
description: "Lleva un vídeo de tubepoli de montaje final a publicado y analizado: prepara y valida el paquete, lo sube a YouTube como privado con confirmación (o lo exporta a Studio), vincula el vídeo publicado, organiza CTA, comentario fijado, pantallas finales y promoción, y hace los chequeos tras publicar (0 h, 24 h, 72 h, 7 días) de forma condicional, con diagnóstico impresiones → CTR → retención frente a un vídeo propio de referencia y aprendizajes que vuelven a buckets e ideas. Úsala cuando el usuario diga «prepara la publicación», «descripción y capítulos», «ya lo publiqué», «¿cómo va el vídeo?», «chequeo de 24 horas», «por qué no funciona este vídeo» o «qué aprendimos»."
---

> Versión de estas guías: 0.10.0 (compárala con guides.latestVersion de capabilities_get).

# Publicación, distribución y análisis (M10 + M11)

Publicar es siempre una acción explícita: mover la ficha de etapa no publica nada. Hay dos vías: **subida directa** (activada por defecto: tubepoli sube el vídeo a YouTube como privado y después lo programa, cada envío con un «sí» explícito del usuario para ese vídeo) y **vía Studio** (tubepoli prepara y valida, la persona publica en YouTube Studio y después se vincula). Pregunta cuál prefiere si no lo ha dicho.

## Cuándo usarla

- Hay montaje final elegido y toca publicar.
- El vídeo ya está publicado y hay que vincularlo, hacer los chequeos o entender sus resultados.

## Flujo paso a paso — Publicar vía Studio

1. `publication_get` (`videoId`): paquete, validación, final elegido, estado y tareas.
2. `publication_prepare` (`expectedRevision` 0 si aún no hay paquete) con `changes`: título y miniatura de la pareja elegida (`title`, `pairRef`, `thumbnailAssetId`), `description` (enlace del CTA en la primera línea), `chapters` del corte final (`startSeconds`, `title`), `tags`, `visibility`, `mode: "manual"`, fecha/hora/zona si se programa en Studio, declaraciones explícitas (`madeForKids`, `containsSyntheticMedia`), `pinnedComment`, `nextVideoId` + `nextVideoReason`. Guardar no ejecuta nada. Ver [reference/publicar-via-studio.md](reference/publicar-via-studio.md).
3. `publication_validate` (`expectedRevision`) → corrige los errores que devuelva (límites de YouTube, capítulos, miniatura, fecha).
4. `publication_export` (`expectedRevision`) → entrega al usuario el texto para copiar en Studio.
5. Cuando el usuario diga que ya está publicado: `publication_link_existing` (`youtube`: URL o ID; sin conexión, `declaredPublishedAt` y `declaredVisibility`).
6. Tareas de Studio y promoción: con el usuario, deja claro qué hacer en Studio y fuera de él (pantalla final hacia el siguiente vídeo, comentario fijado del texto exportado, prueba de título/miniatura, promoción en newsletter o piezas cortas). Las que no se hagan hoy, como tareas en la ficha desde la web.
7. `pipeline_move` (`toStage: "analysis"`) y `analytics_checks_prepare` (crea los chequeos 0 h, 24 h, 72 h y 7 días).

## Flujo paso a paso — Analizar

1. Fuentes: con YouTube conectado (desde la web de tubepoli), los datos se sincronizan solos cada pocas horas. Datos que solo da Studio (CTR por nuevos/recurrentes, primeras horas, prueba A/B) o un canal sin conectar: `analytics_import` (`kind: "manual"` o `"studio_screenshot"` con `values`, o `"studio_csv"` con `csv`; quedan como declarados).
2. Chequeos pendientes: `analytics_checks_list` (`dueOnly: true`). Para cada uno sigue [reference/chequeos.md](reference/chequeos.md) y regístralo con `analytics_checks_record` (`checkId`, `expectedRevision`, `conclusion`: `wait | insufficient_data | review | experiment`, `observation`, `hypothesis`, `action`, `referenceVideoId`, `createLearning`).
3. Diagnóstico de un vídeo: `analytics_diagnose` (`videoId`, `referenceVideoId` = vídeo propio parecido, `days` = misma ventana desde publicación). Para enseñarlo en el chat, `show_analysis`.
4. Retención: mira con el usuario la curva de Studio y relaciónala con los bloques del guion (`scripts_get`). Una caída sugiere investigar, no prueba la causa.
5. Comparar: `analytics_compare` (`mode: "videos"` con 2–10 `videoIds` en la misma ventana; `mode: "buckets"` con `statistic: "mean"` y `"median"`).
6. Aprendizaje: `learning_record` (`title`; `scope: "video"` con `videoId` del vídeo analizado, o `scope: "bucket"` con `bucketId`, o `"channel"`) separando `observation` (hechos), `hypothesis` (por qué creemos) y `proposal` (qué haremos), con `decision` (`keep | change | test`) y `appliesToVideoId`. Conviértelo en trabajo: una idea nueva con `ideas_capture_batch` (`origin: "learning"`, `originRef: {type: "learning", id}`) o una tarea en la siguiente ficha.
7. Prueba de título/miniatura en Studio (máximo 3 opciones): datos con `analytics_import` (`variant`, `abTestWatchShare`) y conclusión con `learning_record` (`kind: "packaging"`, `videoId`); gana la que da más tiempo total de visualización.

## Avanzado (vía operations_call)

- Publicación: `publication_upload` (`dryRun` para el plan), `publication_reference_get`, `publication_reconcile` (estado real; `retrySteps` para reintentar pasos fallidos), `publication_schedule`/`reschedule`/`cancel_pending`, `publication_metadata_apply`, tareas de Studio y promoción como objetos con `distribution_tasks_prepare` (sin `items`, las de Studio a partir del paquete) y `distribution_tasks_update` (`status: "confirmed"` con `evidence`), y la vía nativa (ver [reference/publicar-via-studio.md](reference/publicar-via-studio.md)).
- Fuentes de datos: `analytics_available_reports`, `analytics_authorize` (URL para la persona titular) y `analytics_sync` (job); detalle por vídeo: `analytics_video_get`.
- Retención por bloques: `analytics_retention_map` (`estimate: true` o tramos reales).
- Desgloses (formatos Shorts/largos/directos, audiencia, tráfico, interacción, listas, directos, ingresos): `analytics_insights_catalog` (qué hay y para qué alcance), `analytics_insights_get` (canal o `videoId`; `sections` o `insights`; top-N con cuota, `limit` 1–25, `detail: true` para todas las métricas; devuelve 3–5 `findings`, comparativa por buckets del canal y `liveCounts` del vídeo) y `analytics_insights_refresh` (valida antes de pedir: un desglose no admitido para el alcance o ingresos sin permiso devuelve `VALIDATION_ERROR` con fixHint). Ingresos solo con `analytics_authorize` (`includeRevenue: true`) y canal en el Programa de Socios; si no, `state: "not_eligible"`.
- Informes masivos (pantallas finales, tarjetas, subtítulos, básico/combinado diario, listas): `analytics_report_facts_get` (`report`, `from`/`to`, `videoId`); llegan con hasta 48 h de demora y YouTube guarda cada informe 60 días. Contadores públicos casi en tiempo real: `analytics_live_stats_get` (seguimiento automático cada 15 min durante 72 h tras publicar; no es la métrica `views` de Analytics).
- Aprendizajes: `learning_act` (`create_idea`, `review_packaging` o `link`) y, más tarde, `learning_update` con el `outcome`.
- Comunidad (bandeja de comentarios de todo el canal: sincronizar, priorizar, responder, moderar en lote, convertir preguntas en ideas): ver [reference/comunidad.md](reference/comunidad.md). Ejemplo: «revisa los comentarios» → `community_inbox_list` (si `stale`, antes `community_inbox_sync`) → `community_thread_get` de los prioritarios → borradores con `community_reply` (`send: false`) → «Estas 3 respuestas, ¿las publico?» → con el «sí», `send: true`.
- Pruebas registradas en Packaging: `packaging_tests_record` (`kind: "studio_native"`) y `packaging_tests_close` (con contexto de tiempo visto).

## Avanzado (vía operations_call): subida directa a YouTube

Operaciones fuera del núcleo: búscalas con `operations_search` y ejecútalas con `operations_call`.

1. Pasos 1–3 de arriba (paquete y validación). Con conexión, `publication_reference_get` trae categorías asignables, idiomas y listas del canal; después la validación comprueba `categoryId`, `defaultLanguage` y traducciones contra ellos.
2. Campos extra del paquete: `localizations` (`{"en": {"title", "description"}}`), `playlist` (nombre; se busca y, si no existe, se crea con `playlistPrivacy`) o `playlistYoutubeId`, `captionsAssetId` + `captionsLanguage`/`captionsName`/`captionsDraft` (SRT o VTT), `brandPartner` (`channelHandle` o `channelId`, solo con promoción pagada), `notifySubscribers`, `embeddable`, `license`, `publicStatsViewable`. Ver [reference/publicar-via-studio.md](reference/publicar-via-studio.md).
3. `publication_upload` con `dryRun: true` → `plan.calls` (cada llamada con su bolsa y coste), `plan.cost` (p. ej. `uploads: 1`, `general: 150`) y `plan.blockers`. Enséñale al usuario qué se enviará y el coste; resuelve los bloqueos (validar, ampliar permiso `youtube.force-ssl`).
4. Con su «sí» para este vídeo: `publication_upload` sin `dryRun` → se sube como **privado** (siempre se envía `privacyStatus`). Sigue con `publication_status`: `attempt.progress` en la subida; después `attempt.steps` (`thumbnail`, `playlist`, `captions`: `waiting_processing` mientras YouTube procesa, luego `done` o `failed` con su motivo).
5. Con el usuario: `publication_schedule` (programa en la fecha del paquete o publica, según `mode`). Otro «sí» explícito.
6. Si un paso falló: `publication_reconcile` con `retrySteps: true` reintenta solo lo fallido (`stepsRetried`); lo ya aplicado no se repite. Si la subida quedó en `outcome_unknown`, `publication_reconcile` reanuda la misma sesión: nunca repitas `publication_upload`.
7. Si el usuario prefiere hacerlo a mano o no queda cuota (`QUOTA_EXCEEDED`, `retryAfterSeconds`): `publication_export` y vía Studio.

## Reglas del método

- **Orden del diagnóstico: impresiones → clic (CTR) → consumo (AVD/AVP)**; el éxito es el tiempo total de visualización.
- **Compara con un vídeo propio parecido en la misma ventana** desde la publicación; no compares edades distintas.
- **Chequeos:** al publicar, impresiones, vistas y CTR; a las 24 h, retención; a las 72 h, análisis global; a la semana, conclusión y aprendizajes documentados para el próximo vídeo.
- **Condicionales:** con pocas impresiones el CTR no significa nada; en nivel 0 el problema suele ser de audiencia, idea o buckets. Antes de tocar nada, comprueba volumen, edad, demora de los datos y comparación. «Aún no lo sabemos» es una conclusión válida.
- **Acciones tempranas** solo con muestra suficiente: CTR especialmente bajo → miniatura (revisar título y miniatura juntos); impresiones especialmente bajas → título y metadatos (y revisar la idea).
- **Los metadatos son pistas, no magia**; la distribución real es que se entere la máxima gente cualificada: newsletter, redes, piezas cortas, como un lanzamiento. Pago: experimento opcional.
- **Colas de reproducción:** pantalla final hacia otro vídeo con 2–3 razones para verlo.
- **CTA del lead magnet en la primera línea de la descripción y en el comentario fijado**.
- **Media y mediana** juntas al comparar buckets; la media sola se deja dominar por un viral.
- Lo aprendido vuelve al sistema: siguiente idea, ajuste de packaging o cambio de bucket.

## Errores que evitar

- Publicar, subir, programar o cambiar metadatos en YouTube sin un «sí» explícito para ese vídeo (un lote solo si el usuario lo autoriza como lote).
- Repetir la subida tras un error incierto: reconcilia (gastaría otra subida de la bolsa de 100 diarias).
- Mostrar un dato pendiente como 0 o concluir «no funciona» a las pocas horas.
- Cambiar la miniatura por un CTR calculado con unas decenas de impresiones.
- Mezclar datos declarados de Studio con datos de la API como si fueran lo mismo.
- Registrar un aprendizaje sin la siguiente ficha o tarea donde se aplicará.

## Ejemplos

- «Súbelo y prográmalo para el martes» → `publication_prepare` → `publication_validate` → plan de la subida directa (ver «Avanzado») → «Subiré el vídeo como privado (1 de 100 subidas de hoy), luego miniatura, lista "Sistemas" y subtítulos en español (≈ 500 u.). ¿Lo subo?» → «sí» → subir → seguir el estado hasta privado con los pasos aplicados → «¿Lo programo para el martes 18:00?» → programar.
- «Prepara la publicación del martes a las 18:00» → `publication_get` → `publication_prepare` → `publication_validate` → `publication_export` → «Cópialo en Studio y prográmalo; avísame al publicar».
- «Ya está publicado: youtu.be/abc» → `publication_link_existing` → repasar pantalla final, comentario fijado y promoción → `pipeline_move` a `analysis` → `analytics_checks_prepare`.
- «¿Cómo va el vídeo de ayer?» → `analytics_checks_list` (`dueOnly`) → `analytics_diagnose` con un vídeo de referencia → «Lleva 22 h y 300 impresiones: muestra pequeña para juzgar el CTR. Recomiendo esperar al chequeo de 72 h» → `analytics_checks_record` (`conclusion: "wait"`).
