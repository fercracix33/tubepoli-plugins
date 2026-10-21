---
name: tubepoli-publicacion-y-analisis
description: "Lleva un vídeo de tubepoli de montaje final a publicado y analizado: prepara y valida el paquete para YouTube Studio, lo exporta, vincula el vídeo publicado, organiza CTA, comentario fijado, pantallas finales y promoción, y hace los chequeos tras publicar (0 h, 24 h, 72 h, 7 días) de forma condicional, con diagnóstico impresiones → CTR → retención frente a un vídeo propio de referencia y aprendizajes que vuelven a buckets e ideas. Úsala cuando el usuario diga «prepara la publicación», «descripción y capítulos», «ya lo publiqué», «¿cómo va el vídeo?», «chequeo de 24 horas», «por qué no funciona este vídeo» o «qué aprendimos»."
---

# Publicación, distribución y análisis (M10 + M11)

Publicar es siempre una acción explícita: mover la ficha de etapa no publica nada. La vía de v1 es **vía Studio**: tubepoli prepara y valida, la persona publica en YouTube Studio y después se vincula. La subida nativa por API existe pero está desactivada por defecto; no la actives.

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

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- Publicación: `publication_reconcile` (estado real con conexión), tareas de Studio y promoción como objetos con `distribution_tasks_prepare` (sin `items`, las de Studio a partir del paquete) y `distribution_tasks_update` (`status: "confirmed"` con `evidence`), y la vía nativa (ver [reference/publicar-via-studio.md](reference/publicar-via-studio.md)).
- Fuentes de datos: `analytics_available_reports`, `analytics_authorize` (URL para la persona titular) y `analytics_sync` (job); detalle por vídeo: `analytics_video_get`.
- Retención por bloques: `analytics_retention_map` (`estimate: true` o tramos reales).
- Aprendizajes: `learning_act` (`create_idea`, `review_packaging` o `link`) y, más tarde, `learning_update` con el `outcome`.
- Pruebas registradas en Packaging: `packaging_tests_record` (`kind: "studio_native"`) y `packaging_tests_close` (con contexto de tiempo visto).

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

- Publicar, programar o cambiar metadatos en YouTube sin un «sí» explícito (y nunca por la vía nativa si no está habilitada).
- Mostrar un dato pendiente como 0 o concluir «no funciona» a las pocas horas.
- Cambiar la miniatura por un CTR calculado con unas decenas de impresiones.
- Mezclar datos declarados de Studio con datos de la API como si fueran lo mismo.
- Registrar un aprendizaje sin la siguiente ficha o tarea donde se aplicará.

## Ejemplos

- «Prepara la publicación del martes a las 18:00» → `publication_get` → `publication_prepare` → `publication_validate` → `publication_export` → «Cópialo en Studio y prográmalo; avísame al publicar».
- «Ya está publicado: youtu.be/abc» → `publication_link_existing` → repasar pantalla final, comentario fijado y promoción → `pipeline_move` a `analysis` → `analytics_checks_prepare`.
- «¿Cómo va el vídeo de ayer?» → `analytics_checks_list` (`dueOnly`) → `analytics_diagnose` con un vídeo de referencia → «Lleva 22 h y 300 impresiones: muestra pequeña para juzgar el CTR. Recomiendo esperar al chequeo de 72 h» → `analytics_checks_record` (`conclusion: "wait"`).
