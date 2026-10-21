# Plantillas de plan de prueba

Son dos variantes del método. Se elige una; no se combinan ni se acumulan sus requisitos, porque sumarlas daría un plan contradictorio (15 publicaciones y a la vez tres meses fijos).

## `sept_5x3` — prueba de 5 buckets × 3 publicaciones

- Cinco buckets × tres publicaciones = **15 huecos**.
- Ritmo propuesto: dos publicaciones por semana → algo menos de dos meses (≈7,5 semanas de capacidad). Con una por semana serían casi cuatro meses.
- Ideación una vez al mes: una idea por bucket hasta ocho ideas (dos por semana × cuatro semanas), repartidas por igual entre buckets.
- Cada idea: encaja en un bucket, cruza el tema con un formato que ya funcionó y cumple los cinco criterios de alta audiencia.
- **Termina por publicaciones confirmadas**: con las 15 publicadas se abre la revisión.
- Después: clasificar buckets por vistas y por leads y reorientar la mezcla según el objetivo.

## `june_3_months` — fase de prueba de tres meses

- Fase 1 de **unos tres meses** a dos publicaciones por semana (≈24 vídeos, 4–5 por bucket).
- Ideas solo de tres fuentes: outliers propios, outliers externos, tendencias (estas, con moderación: son volátiles).
- Rotación equitativa entre los cinco buckets; una idea que no encaja en un bucket se guarda para más adelante.
- **Termina por horizonte** (fin del periodo), no por número de vídeos.
- Fase 2 (optimización, ~3 meses): quedarse con los 3 buckets con mejor media de vistas o mejor conversión a leads; mantener al menos un bucket de alcance.
- Fase 3 (acordeón, sin fin): 3 pilares + máximo 2 buckets de prueba.

## Cómo crearlo en tubepoli

1. `plans_create`: `template`, `buckets[]` (`bucketId`, `targetPublications`, `role`, `trial`), `question`, `calendar` opcional. Parámetros distintos de la plantilla quedan marcados como adaptación.
2. `plans_calendar_propose`: `startDate` (AAAA-MM-DD), `timeZone` (IANA), `weeklySlots` (`weekday` 1–7, 1 = lunes, y `time` HH:mm), `pauses`. Devuelve `proposalId`, fechas, rotación A→B→C… y conflictos (cambio de hora, fechas insuficientes).
3. `plans_calendar_apply` (`proposalId`, `expectedPlanRevision`) → `plans_activate`. Máximo un plan en curso por canal (`active_plan_exists`).
4. Para cambiar fechas antes de activar, vuelve a `plans_calendar_propose` con el calendario nuevo y aplícalo. Cada ficha real ocupa su hueco con `plans_slots_assign`.

Las fechas del plan son **propuestas**. La fecha comprometida de una ficha es otra cosa y la programación en YouTube es otra distinta (Distribución). Ninguna de ellas publica nada.

## Avanzado (vía operations_call)

- Ajustes de un plan en curso: `plans_calendar_adjust` (mover huecos), `plans_slots_add`, `plans_slots_remove`, `plans_update`, `plans_pause`, `plans_resume`.
- Fecha comprometida de una ficha: `commitments_set` (`localDate`, `expectedRevision` de la ficha).
