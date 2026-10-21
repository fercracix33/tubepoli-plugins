# Revisión del ciclo de buckets

Se abre al confirmar todas las publicaciones (5×3) o al terminar el horizonte (tres meses). Antes, solo como revisión anticipada con motivo.

## Pasos

1. `plans_progress_get`: qué falta (publicaciones confirmadas, análisis) y si procede abrir la revisión. Lo no confirmado queda pendiente, no a cero.
2. `reviews_prepare` con `kind: "bucket_test"` (periodo por defecto: desde el inicio del plan; `objective` = la pregunta del plan). Reúne publicaciones con su cobertura, aprendizajes, chequeos vencidos y la comparación de buckets (mediana de vistas a 7 días, con muestra y método). Anota el `id` y la `revision` de la revisión.
3. Compara con la misma ventana desde la publicación para todos: `analytics_compare` con `mode: "buckets"`, `days`, `metric` (`views` o `leads`) y `statistic` `mean` y después `median`. Los vídeos con ventana incompleta quedan fuera de los estadísticos (se listan con motivo).
4. Redacta con el usuario una decisión por bucket con `reviews_decisions_set` (`expectedRevision`; cada decisión con `subject: {type: "bucket", id, label}`, `text` con motivo, hipótesis e incertidumbre, y `nextRef` = el siguiente plan, ficha o tarea donde se aplica):
   - `keep` = **Mantener** (motivo).
   - `change` = **Cambiar** (qué cambia en la definición, formato o rol). **Retirar del próximo plan** también es `change`, con el texto «Retirar del próximo plan» y el motivo (no borra ni archiva el bucket).
   - `test` = **Probar** (di en `text` qué queremos saber).
   - Sin decisión = pendiente (no inventes una conclusión).
   Una hipótesis no es un hecho: sepáralas en el texto.
5. Cambios concretos de definición: `buckets_update` (`expectedBucketRevision`, `changes`).
6. `reviews_close` con `conclusion` y `nextCycle` (qué cambia en el próximo plan). Avisa si alguna decisión no enlaza dónde se aplica.
7. Siguiente plan: `plans_create` con los buckets que se mantienen y los nuevos marcados `trial: true` (en acordeón respeta el máximo de buckets de prueba) y `previousPlanId`; después calendario y activación como en el plan inicial. Las fichas aún no publicadas se asignan a los huecos nuevos con `plans_slots_assign`.
8. Si cambia el foco, hazlo explícitamente con `strategy_focus_set` (la revisión no lo aplica sola). Si cambia la fase (optimización o acordeón), anótalo en la decisión y aplícalo como indica «Avanzado» (abajo).

## Cómo leer los números

- Media y mediana juntas: 1.000, 1.000 y 100.000 vistas dan media 34.000 y mediana 1.000. La media sola no demuestra que el bucket sea estable; úsala como criterio práctico, pero acompañada de la mediana.
- Tres vídeos por bucket es el mínimo del método para decidir, no una validación estadística.
- Con objetivo de clientes, ordena también por leads (o leads por cada 1.000 vistas) además de vistas.
- Mantén al menos un bucket que traiga audiencia nueva.

## Avanzado (vía operations_call)

Cambio de fase: `strategy_phase_set` (`expectedStrategyRevision`, `phase`: `optimization` o `accordion`, con `accordion.maxTrialBuckets`).

Revisión formal del plan, con las cuatro decisiones por bucket guardadas en el propio plan (`keep`, `adjust`, `continue_testing` con `nextQuestion`, `exclude` = Retirar del próximo plan):

1. `plans_reviews_open` (`planId`, `expectedPlanRevision`; `early: true` + `reason` si es anticipada) y, con datos nuevos, `plans_reviews_evidence_refresh` (`expectedReviewRevision`).
2. `plans_reviews_decisions_set` (`decisions[]` con `bucketId`, `decisionKind`, `reasonMarkdown`, `hypothesisMarkdown`, `uncertaintyMarkdown`, `nextQuestion`).
3. `plans_reviews_adjustments_request` con `kind: "bucket_definition"` y `bucketChanges` para cambios de definición.
4. `plans_reviews_close` (con `closeReason` si quedan pendientes).
5. `plans_next_prepare` (`reviewId`, `keepBuckets`, `newBuckets`) → `plans_slots_move_pending` para trasladar las fichas aún no publicadas.
