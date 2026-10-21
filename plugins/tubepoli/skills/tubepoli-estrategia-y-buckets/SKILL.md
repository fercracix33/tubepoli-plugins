---
name: tubepoli-estrategia-y-buckets
description: "Aplica la capa estratégica del método en tubepoli: diagnostica el nivel del canal (0–3) con evidencia, fija un único foco y la fase (prueba, optimización, acordeón), define los buckets de contenido, crea el plan de prueba (plantilla 5×3 o de tres meses) con su calendario y cierra el ciclo con decisiones Mantener/Cambiar/Probar/Retirar. Úsala cuando el usuario pregunte «¿en qué nivel estoy?», «¿en qué me centro?», «define mis buckets», «haz el plan de prueba», «calendario de publicaciones», «qué buckets funcionan» o «prepara el siguiente plan»."
---

# Estrategia, buckets y plan (M15 + M02)

## Cuándo usarla

- Canal nuevo con contexto básico (tema, audiencia, objetivo) y sin buckets ni plan.
- El usuario no sabe en qué centrarse o pide un diagnóstico.
- Se completó la prueba de buckets o el plan llega a su fin.

## Flujo paso a paso

### A. Nivel, foco y fase

1. `strategy_get` (nivel, foco con `focusRevision`, fase con `strategyRevision`, capacidad, colchón, progreso del plan).
2. Reúne evidencia sin inventar: lo que el usuario sepa de Studio (impresiones, vistas, horas de visualización, vídeos publicados), `analytics_compare` si ya hay vídeos con datos y `business_funnel_summary` (leads, si aplica). Lo que no exista va como `value: null` con su `coverage`.
3. Propón el nivel con `strategy_level_propose` (`level` 0–3 o `null`, `certainty`: `provisional | supported | insufficient_data`, `rationale`, `uncertainty`, `evidence` por métrica: `impressions`, `views`, `watchTimeHours`, `leads`, `videosPublished` con `{value, coverage, period, source}`). Con poca muestra, `provisional` o `insufficient_data`. Si el canal no vende nada, `conversionNotApplicable` con el motivo.
4. Explica al usuario el nivel y su problema (tabla abajo). Solo si él lo acepta o corrige, `strategy_level_accept` (`diagnosisId`: el `level.diagnosisId` que devuelve la propuesta o `strategy_get`; `decision`: `accept | reject | correct`, con `correctedLevel` si corrige). No lo aceptes en su nombre.
5. Foco único: `strategy_focus_set` con `expectedFocusRevision` y un `preset` (`complete_bucket_trial`, `improve_packaging`, `improve_intros`, `create_lead_magnet`) o `title` propio, más `reason`, `exitCriterion` y hasta 3 `actions` (`label`).
6. Fase: un canal empieza en **prueba** (`trial`, valor por defecto del método que muestra `strategy_get`). Pasar a optimización o acordeón se decide en la revisión del ciclo (ver «Avanzado»).
7. Ritmo y mezcla: el método parte de dos publicaciones por semana, grabar el doble y ocho ideas al mes; y, para captar clientes, ~4 vídeos de conversión por 1 de alcance. Díselo al usuario y anota sus valores reales si difieren (ajustarlos en tubepoli es avanzado).

### B. Buckets

1. `buckets_list` para ver lo que hay (cada bucket con su `bucketRevision`).
2. Si faltan referencias, primero `tubepoli-investigacion` (20–30 canales y sus mejores vídeos). Los buckets salen de agrupar esos mejores vídeos por algo en común: tema, formato o ambos.
3. Propón 5 buckets con nombre, una frase de definición, `patternKind` (`topic | format | hybrid`) y `role` (`reach | conversion | mixed`). Espera la validación del usuario.
4. Crea cada uno con `buckets_create` (`operationId` distinto por bucket; `name`, `summary`, `patternKind`, `role` y `bodyMarkdown`: qué entra, qué no, problema de la audiencia, nivel de consciencia del público y ejemplos). Para corregirlo después, `buckets_update` (`expectedBucketRevision` = su `bucketRevision`, `changes`). Las referencias que lo inspiran se asocian al guardarlas con `bucketIds` (`references_save`, ver `tubepoli-investigacion`).

### C. Plan de prueba

Plantillas separadas (no se suman): ver [reference/plantillas-plan.md](reference/plantillas-plan.md).

0. Antes de crear: mira los planes abiertos en `context_get` (o `plans_list`). Si hay un borrador o un plan pausado, pregunta si se continúa (`plans_activate`) o se descarta (ver «Avanzado»): una ficha solo ocupa un hueco pendiente en los planes del canal.
1. `plans_create` con `template: "sept_5x3"` (5 buckets × 3 publicaciones = 15 huecos) o `"june_3_months"` (horizonte de tres meses), `buckets` incluidos (`bucketId`) y `question` (qué queremos aprender).
2. `plans_calendar_propose` (`planId`, `calendar`: `timeZone`, `startDate`, `weeklySlots` p. ej. martes y viernes, `pauses`). Muestra fechas, rotación y conflictos. Nada se compromete aún.
3. Si el usuario está de acuerdo: `plans_calendar_apply` (`proposalId`, `expectedPlanRevision` = el `planRevision` que devolvió el plan) y `plans_activate` (con el `planRevision` nuevo).
4. Los huecos no son fichas: cuando haya una idea real, `plans_slots_assign` (`slotId` de `byBucket[].missingSlotIds` en `plans_progress_get`, `videoId`, `expectedPlanRevision`). Seguimiento: `plans_progress_get` (también lo enseña `show_plan`). Si la asignación falla con `duplicate`, `error.details.conflict` dice en qué plan y hueco está ya la ficha: libera ese hueco (ver «Avanzado») o continúa ese plan.

### D. Revisión del ciclo y siguiente plan

Detalle en [reference/revision-ciclo.md](reference/revision-ciclo.md): `plans_progress_get` → `reviews_prepare` (`kind: "bucket_test"`) → `analytics_compare` (`mode: "buckets"`, media y mediana) → `reviews_decisions_set` (una decisión por bucket) → `reviews_close` → siguiente plan con `plans_create`.

## Avanzado (vía operations_call)

- Planes: descartar un borrador con `plans_cancel` (`reason` obligatorio) y liberar un hueco con `plans_slots_unassign` (el error `duplicate` de `plans_slots_assign` trae en `details.conflict` el `planId`, el `slotId` y el `planRevision` que necesita).
- Evidencia del nivel: `analytics_channel_get` (totales del canal sincronizados) y `research_benchmark` (media de las medianas de tus canales de referencia).
- Cambiar la fase: `strategy_phase_set` con `expectedStrategyRevision` (`phase`: `trial`, `optimization`, `accordion`; en acordeón, `accordion.maxTrialBuckets`, 2 por defecto).
- Ritmo y mezcla guardados: `strategy_capacity_update` (`useMethodDefaults: true` o valores reales) y `strategy_mix_update` (`useMethodTemplate: true`).
- Buckets: `buckets_references_add` (referencia con `reason`), `buckets_patterns_link` (formatos y fórmulas de M18) y `buckets_awareness_set` (nivel de consciencia).
- Revisión formal del plan por bucket, con «Retirar del próximo plan» y paso de fichas pendientes: `plans_reviews_open`, `plans_reviews_evidence_refresh`, `plans_reviews_decisions_set`, `plans_reviews_adjustments_request`, `plans_reviews_close`, `plans_next_prepare` y `plans_slots_move_pending` (ver el final de [reference/revision-ciclo.md](reference/revision-ciclo.md)).

## Niveles del método

| Nivel | Síntoma | Qué resolver | Qué no tocar todavía |
| --- | --- | --- | --- |
| 0 Visibilidad | Casi sin impresiones: YouTube no sabe a quién enseñar los vídeos. | Buckets con disciplina, ideas de alta audiencia, ritmo para completar 15 vídeos. | Cambiar miniaturas por un CTR con pocas impresiones (no significa nada), cámara o edición. |
| 1 Visualizaciones | Hay impresiones, pocas vistas. | Clic (packaging) y retención (guion, sobre todo la intro). | Producción y edición como palanca principal. |
| 2 Conversión | Hay vistas, pocos leads o leads no cualificados. | Lead magnet, CTA al ~33 %, atribución por vídeo, colas de reproducción. | Perseguir más alcance genérico. |
| 3 Optimización | Vistas y leads estables. | Clasificar buckets por vistas y por leads, mezcla 4 leads : 1 alcance, llevar a la audiencia por los niveles de consciencia. | Abrir demasiados frentes de prueba. |

## Reglas del método

- **Un foco cada vez:** se sube de nivel resolviendo el problema del nivel actual, no haciendo de todo. Aprender mejorando una fase cada vez mientras se sigue publicando.
- **Bucket = serie de vídeos con algo en común** (tema, formato o ambos); cinco buckets; durante la estrategia solo se publica dentro de ellos.
- **Validar un bucket = mínimo 3 vídeos publicados** antes de descartarlo; misma cantidad por bucket y rotación equitativa (1, 2, 3, 4, 5, 1…).
- **Velocidad:** dos publicaciones por semana; con una por semana, 15 vídeos llevan casi cuatro meses.
- **No cambiar la estrategia a mitad del mes** aunque unos vídeos funcionen mejor que otros.
- **Fase de prueba:** ideas solo de outliers propios, outliers externos o tendencias; idea, packaging y guion con obsesión; grabación y edición «mínimo viable».
- **Optimización:** quedarse con los 3 buckets con más vistas medias (o más leads por vídeo si el objetivo es vender); se puede bajar a un vídeo por semana para subir calidad.
- **Acordeón:** 3 buckets pilares + como máximo 2 de prueba, asumiendo que la mayoría de pruebas fallará; abrir para explorar, cerrar para explotar.
- **Mezcla por objetivo:** captar clientes ≈ 4 buckets de leads : 1 de alcance; canal de entretenimiento, al revés.
- **Medias con cuidado:** la media de vistas normaliza por número de vídeos pero un viral la domina; muestra también mediana y cada vídeo.

## Errores que evitar

- Crear 15 fichas de vídeo ficticias para «llenar» el plan: el plan tiene huecos; las fichas nacen de ideas reales.
- Mezclar la plantilla 5×3 con la de tres meses o sumar sus requisitos.
- Declarar ganador un bucket con uno o dos vídeos, o comparar vídeos de edades distintas.
- Fijar varios focos a la vez o aceptar el nivel sin el usuario.
- Activar el plan sin enseñar antes el calendario propuesto.

## Ejemplos

- «¿En qué me centro?» → `strategy_get` → evidencia (datos de Studio que te dé el usuario) → `strategy_level_propose` (nivel 0, `provisional`) → explicar → con su «sí», `strategy_level_accept` → `strategy_focus_set` (`complete_bucket_trial`).
- «Haz el plan de prueba, publico martes y viernes desde el lunes» → `buckets_list` → `plans_create` (`sept_5x3`) → `plans_calendar_propose` → mostrar 15 fechas → `plans_calendar_apply` → `plans_activate`.
- «Ya publiqué los 15, ¿qué funciona?» → guía D: `reviews_prepare` (`bucket_test`), comparar buckets con media y mediana, proponer decisiones y preparar el siguiente plan.
