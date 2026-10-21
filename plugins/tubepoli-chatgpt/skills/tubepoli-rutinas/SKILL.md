---
name: tubepoli-rutinas
description: "Ejecuta en tubepoli las tres rutinas de revisión del método como checklists con sus operaciones: revisión diaria operativa (qué desbloquear hoy), revisión semanal de métricas (qué aprendimos y en qué próxima ficha se aplica) más la revisión semanal de fuentes propias, y revisión mensual de estrategia (nivel, foco, plan, ritmo, colchón e ideación del mes). Úsala cuando el usuario diga «revisión diaria», «qué hago hoy», «qué está bloqueado», «revisión semanal», «qué aprendimos esta semana», «revisión del mes», «cierre de mes» o «planifica el próximo mes»."
---

# Rutinas de revisión

El método distingue una reunión diaria de producción (avance, cuellos de botella) y una semanal de métricas (impresiones, clics, retención, pruebas y qué cambiar en el siguiente vídeo). tubepoli añade la revisión mensual de estrategia (M15) y la semanal de fuentes (M17). Son recetas que se lanzan a mano o desde una tarea programada de tu cliente; tubepoli no ejecuta agentes por su cuenta.

## Cuándo usarla

- El usuario pide una revisión o «qué hago ahora».
- Al empezar el día, la semana o el mes con el canal.

## Flujo paso a paso

### Diaria (5 minutos) — qué desbloquear hoy

1. `pipeline_daily_review`: tareas bloqueadas y qué las resuelve, vencidas y de hoy, fichas sin siguiente acción, trabajo pendiente de otra persona o de la IA.
2. `analytics_checks_list` con `dueOnly: true`: chequeos tras publicar que tocan hoy (hazlos con `tubepoli-publicacion-y-analisis`).
3. Si hay publicaciones esta semana: `plans_progress_get` (o `pipeline_list` con `dueFrom`/`dueTo`) para ver qué está comprometido.
4. Propón al usuario **3 acciones como máximo** para hoy. Con su acuerdo: `tasks_complete` (`taskId`, `expectedRevision`; solo si está hecho; con dependencias abiertas pide motivo) o `pipeline_move` si una ficha cumple lo de su etapa. Reasignar o reprogramar tareas, desde la web (o ver «Avanzado»).

Salida: lista corta «Hoy: 1) … 2) … 3) …».

### Semanal — métricas (qué aprendimos)

1. `reviews_prepare` con `kind: "weekly"` (y `periodStart`/`periodEnd` si no es la semana anterior): publicaciones con cobertura, totales, aprendizajes, ajustes pendientes de resultado, chequeos vencidos y decisiones de la revisión anterior. Anota su `id` y `revision`.
2. Para los vídeos de la semana, `analytics_compare` (`mode: "videos"`, misma ventana) o `analytics_diagnose` con un vídeo de referencia.
3. Revisa los ajustes anteriores: `learning_list` con `pendingOutcome: true`; su resultado se anota en la decisión de esta semana (`previousOutcome`).
4. Decide con el usuario (máximo 3–5 decisiones): `reviews_decisions_set` (`expectedRevision`) con `decision` (`keep` Mantener, `change` Cambiar, `test` Probar), `subject`, `text`, **`nextRef` = la próxima ficha, bloque o tarea donde se aplicará** y `previousOutcome` si revisa una decisión anterior.
5. Convierte lo accionable: `learning_record` y, si da para una idea, `ideas_capture_batch` (`origin: "learning"`, `originRef: {type: "learning", id}`).
6. `reviews_close` con `conclusion` y `nextCycle`. Avisa si alguna decisión no enlaza dónde se aplicará.

### Semanal — fuentes propias (otra revisión)

1. Si tu cliente tiene acceso al calendario, correo o notas del usuario (y él lo autoriza), extrae solo fragmentos útiles: preguntas, problemas, objeciones, experiencias.
2. `own_sources_capture` con esos fragmentos → `own_sources_list` (`from` = inicio de la semana) para repasarlos con el usuario.
3. Decide con el usuario por señal: idea (`ideas_capture_batch` con `originRef: {type: "own_signal", id}`), historia (señal `experience` con su `note`), usable más adelante o descartar.

### Mensual — estrategia y mes siguiente

1. `reviews_prepare` con `kind: "monthly"` (publicaciones del mes, aprendizajes y comparación de buckets con muestra y método).
2. Evidencia: `strategy_get` (nivel, foco, fase, colchón y progreso del plan), `plans_progress_get` y `business_funnel_summary` si hay objetivo de leads.
3. Decide con el usuario y **aplica cada cambio con su operación**: `strategy_level_propose`/`strategy_level_accept`, `strategy_focus_set`. Si el plan terminó, sigue la revisión del ciclo de `tubepoli-estrategia-y-buckets`.
4. `reviews_decisions_set` con una decisión por tema (nivel, foco, mezcla, plan, ritmo, fase; `subject: {type: "channel", id: <channelId>, label: "<tema>"}`) y `reviews_close` con `conclusion` y `nextCycle`.
5. Mes siguiente: ideación mensual (ocho ideas, una por bucket, ver `tubepoli-ideacion`) y días de grabación por lotes para grabar el doble de lo que se publica (ver `tubepoli-produccion-y-montaje`).

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- Diaria: `pipeline_calendar` (`from`, `to`), `tasks_update` (responsable, fecha, bloqueo, `completionCriteria`) y `tasks_create` para lo que falte.
- Mensual: sesión de ideación con plantilla, `ideas_session_create` (`template: "monthly_8"`); sesiones de grabación, `production_session_create`; cambio de fase, `strategy_phase_set`.
- Semanal: `reviews_get`, `learning_update` (`outcome` del ajuste) y `learning_act` (`create_idea` o `review_packaging`).
- Fuentes: `own_sources_weekly_review_prepare` → `own_sources_weekly_review_close` (`privacyResolved`), `own_sources_convert_to_idea`, `stories_create`.
- Mensual de estrategia (M15): `strategy_review_prepare` (instantánea de nivel, foco, fase, mezcla, capacidad, colchón y plan) → `strategy_review_close` (`summaryMarkdown`, `decisions` con `topic`: `level`, `focus`, `mix`, `plan`, `capacity`, `phase`, `other`; `nextPlanId`); colchón detallado `strategy_buffer_get`; cambios de ritmo y mezcla `strategy_capacity_update`, `strategy_mix_update`; ajustes del plan `plans_update`.

## Reglas del método

- Diario: detectar cuellos de botella antes de que empiece la jornada.
- Semanal: qué pasó con impresiones, clics y retención, qué dieron las pruebas y qué cambia en el **siguiente** vídeo.
- Mensual: ideas del mes de una vez, una por bucket hasta ocho, sin cambiar la estrategia a mitad de mes.
- Un solo foco cada vez; se cambia en la revisión mensual con motivo.
- Revisión semanal de métricas ≠ revisión semanal de fuentes.

## Errores que evitar

- Convertir la diaria en una auditoría larga.
- Cerrar la semanal sin decir dónde se aplica cada aprendizaje.
- Aplicar cambios de estrategia «por debajo» al cerrar la revisión mensual: cada cambio es una operación explícita.
- Prometer recordatorios automáticos: dependen de que el cliente del usuario tenga tareas programadas.
- Volcar datos personales de leads o de reuniones en el resumen.

## Ejemplos

- «¿Qué hago hoy?» → `pipeline_daily_review` → `analytics_checks_list` (`dueOnly`) → «Hoy: 1) desbloquear la miniatura del vídeo X (falta la foto), 2) chequeo de 24 h del vídeo Y, 3) revisar el montaje v2 del vídeo Z».
- «Revisión de la semana» → `reviews_prepare` (`weekly`) → comparar → `reviews_decisions_set` con `nextRef` → `reviews_close`.
- «Cierre de mes» → `reviews_prepare` (`monthly`) → evidencia → decisiones aplicadas una a una → `reviews_close` → ideas del mes (`tubepoli-ideacion`).
