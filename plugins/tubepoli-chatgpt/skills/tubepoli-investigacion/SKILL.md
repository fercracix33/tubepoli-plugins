---
name: tubepoli-investigacion
description: "Investiga para un canal en tubepoli siguiendo el método: reúne 20–30 canales de referencia, saca su inventario por playlist (sin gastar búsquedas), detecta outliers con el método de la mediana, analiza el hook en cinco componentes, guarda referencias con «qué te interesa» y «cómo adaptarlo», propone ideas propias y recoge señales e historias reales del creador (Mis fuentes). Úsala cuando el usuario diga «busca referencias», «qué canales sigue mi audiencia», «encuentra outliers», «analiza este vídeo», «guarda este vídeo», «qué problemas tiene mi audiencia» o «esta semana un cliente me preguntó…»."
---

# Investigación y fuentes propias (M03 + M12 + M17)

## Cuándo usarla

- Antes de definir buckets (hacen falta los mejores vídeos de tu nicho).
- Para alimentar la ideación con referencias, problemas y señales reales.
- Cuando el usuario comparte un enlace o una experiencia que quiere aprovechar.

## Flujo paso a paso

### A. Canales de referencia (objetivo orientativo: 20–30)

1. Parte del contexto (`context_get`): audiencia y sus problemas.
2. Encuentra candidatos buscando en YouTube los problemas de la audiencia. Opciones, de más barata a más cara:
   - URLs o @handles que te da el usuario o que encuentres con tu propia búsqueda web (sin gastar cuota de tubepoli).
   - Búsqueda dentro de YouTube desde tubepoli: es avanzada y gasta cuota (ver [reference/busqueda-y-cuota.md](reference/busqueda-y-cuota.md)).
3. Añade cada canal con `research_resolve` (`input`: URL, @handle o ID; `reason`; `topic`; `bucketIds`). No duplica si ya existe. Devuelve el canal de referencia con su id.
4. Inventario barato: `research_channels_sync` con los `researchChannelIds` (lee la playlist de subidas: ~1 unidad por 50 vídeos, sin búsquedas). Devuelve `jobId`: di que está en marcha y vuelve después.
5. `research_channels_get` (`researchChannelId`, `sort: "ratio"`): vídeos ordenados por veces sobre la mediana del canal. Señala al usuario los canales pequeños con outliers.

### B. Outliers

1. En `research_channels_get` con `sort: "ratio"`, cada vídeo trae su ratio frente a la mediana del canal (método `median_ratio_v1`: misma clase de duración, ≥14 días, los 30 vídeos más cercanos en fecha; con poca muestra, datos insuficientes). Cuenta muestra y método al usuario; no es un dato oficial ni dice nada de CTR o retención.
2. Guarda el candidato con `references_save` usando `fromInventory` (`researchChannelId`, `youtubeVideoId`), sin gastar cuota de nuevo. Anota en `whyInteresting` el ratio y la muestra, y tu decisión editorial («outlier apoyado», «candidato», «sin evaluar») con su motivo.

### C. Hook en cinco componentes

Analízalo antes de guardar la referencia, para que vaya en la misma llamada de `references_save` (si ya estaba guardada, devuelve la existente sin cambiarla; editarla es avanzado). Escribe en `noteMarkdown` los cinco componentes del hook — **idea**, **título**, **miniatura**, **enfoque** e **intro** —, cada uno con qué hay (observación) y por qué funciona (principio), más `howToAdapt` (qué aportaría el canal) y `observedPattern` (etiquetas cortas del patrón). Si el usuario te da una transcripción, cita solo los fragmentos que sostienen el análisis.

### D. Guardar referencias bien anotadas

`references_save` (`url`, o `fromInventory`) con:
- `whyInteresting` — «Qué te interesa» (lo observable).
- `howToAdapt` — «Cómo podrías adaptarlo» (tu propuesta para este canal).
- `observedPattern`, `bucketIds` (así queda asociada al bucket), `audienceProblem`, `format`.
Guarda las anotaciones en la primera llamada: es la que crea la referencia.

### E. De referencia a idea propia

`ideas_capture_batch` con `items: [{workingTitle, origin: "reference", originRef: {type: "video_reference", id: <referenceId>}}]` y `bucketId`: título en lenguaje del espectador, propio (no el de la referencia). Crea la ficha en Ideación con su origen; qué se adapta y qué aporta el canal van al brief en `tubepoli-ideacion` (`ownContribution`).

### F. Problemas de la audiencia y Mis fuentes

- Problemas encontrados (preguntas de buscadores, comentarios, investigación tuya): con el usuario, añádelos al contexto del canal (`channels_context_update` → `bodyMarkdown`, sección «Problemas de la audiencia» con su fuente; lee el cuerpo vigente y añade, no sustituyas). Son problemas con fuente, no demanda medida.
- Señales e historias de la semana real del creador: ver [reference/mis-fuentes.md](reference/mis-fuentes.md) (`own_sources_capture` y, para convertirlas en idea, `ideas_capture_batch` con `originRef`).

## Avanzado (vía operations_call)

- Listado y marcas de canales: `research_channels_list`, `research_channels_update` (`highlighted: true`).
- Búsqueda en YouTube y cuota: `search_preview`, `search_run`, `searches_save`, `budgets_get` (detalle en [reference/busqueda-y-cuota.md](reference/busqueda-y-cuota.md)); benchmark del nicho: `research_benchmark`; seguir el trabajo de sincronización: `jobs_get`.
- Outliers sobre una referencia guardada: `outliers_evaluate` (`referenceId`) y decisión editorial `outliers_mark` (`supported | candidate | not_evaluated` + `reason`).
- Hook estructurado: `patterns_record` (`components.idea`, `.title`, `.thumbnail`, `.angle`, `.intro` con `observation` y `principle`, más `ownContribution`). Transcripciones: `documents_import` y `sources_search`.
- Revisar y editar referencias: `references_list`, `references_update` (p. ej. `status: "reviewed"`), `buckets_references_add`.
- Idea desde referencia con `adapts` y `contribution`: `research_propose_idea` (y `research_ideas_materialize` si queda `pending_pipeline`).
- Problemas como objetos: `research_problems_add` (`statement`, `sourceKind`, `sourceUrl`).
- Mis fuentes completas: `own_sources_consolidate`, `own_sources_permissions_set`, `own_sources_convert_to_idea`, `stories_create`, `authority_record` (ver [reference/mis-fuentes.md](reference/mis-fuentes.md)).

## Reglas del método

- Público concreto → sus problemas → buscarlos en YouTube → lista de 20–30 canales → sus mejores vídeos → agrupar en buckets.
- Outlier = vídeo que rinde muy por encima de lo normal en su canal (p. ej. 10× su media). Replicar outliers propios y ajenos es la fuente principal de ideas al arrancar.
- Copiar el formato está bien si aportas un **grado de diferencia**: información distinta o más completa, juntar varios outliers, mejor producción, tu experiencia o un formato distinto.
- Entender el principio, no copiar: analizar idea, título, miniatura, enfoque e intro de la referencia.
- Las mejores ideas salen de la semana real: calendario, reuniones, llamadas, problemas del equipo.
- Inspirarse en miniaturas y títulos que tu audiencia ya clicó, sin copiarlos tal cual.

## Errores que evitar

- Buscar en YouTube para todo: para canales concretos, inventario por playlist (`research_channels_sync`) es mucho más barato.
- Tratar un ratio con muestra insuficiente como outlier probado.
- Guardar referencias sin «qué te interesa / cómo adaptarlo»: luego no sirven para idear.
- Copiar el título de la referencia como título propio (guárdalo separado; propone uno nuevo).
- Guardar conversaciones completas o datos personales como señales: solo el fragmento útil.

## Ejemplos

- «Encuéntrame canales que vea mi audiencia» → contexto (`context_get`) → proponer búsquedas por problemas (con tu búsqueda web o la que haga el usuario) → elegir canales → `research_resolve` ×N → `research_channels_sync`.
- «¿Qué vídeos han petado en estos canales?» → `research_channels_get` (`sort: "ratio"`) → `references_save` (`fromInventory`, con el ratio y la muestra en `whyInteresting`) → resumen con muestra y método.
- «Hoy un cliente me preguntó cómo automatizar sus facturas» → `own_sources_capture` (`kind: "question"`, `sourceType: "call"`) → «¿La convertimos en idea?» → `ideas_capture_batch` (`origin: "client"`, `originRef: {type: "own_signal", id}`).
