---
name: tubepoli-ideacion
description: "Dirige la ideación de vídeos en tubepoli con el método: ejercicio 100→10→1 o sesión mensual de ocho ideas repartidas por buckets, evaluación cualitativa de los cinco criterios de alta audiencia (TAM, evergreen, fácil de ver en una miniatura, grado de diferencia, formato repetible), gran recompensa y brief, evidencia de demanda (500.000 vistas como orientación editable, evidencia comercial en B2B) y elección de la idea. Úsala cuando el usuario diga «dame ideas», «sesión de ideas del mes», «¿es buena esta idea?», «elige la mejor», «qué vídeo hago ahora» o «tengo una idea para un vídeo»."
---

# Ideación (M05 sobre fichas de M04)

La idea es lo que más pesa: si es mala, nada de lo posterior la salva. Cada idea se apunta como un **título tentativo en lenguaje del espectador**, no como un tema.

## Cuándo usarla

- Toca generar ideas (semanal, o mensual según el plan).
- El usuario trae una idea y quiere saber si merece la pena.
- Hay que elegir qué ficha pasa a Packaging.

## Flujo paso a paso

1. Contexto: `context_get`, `buckets_list` y, si hay plan, `plans_progress_get` (qué buckets necesitan ideas para mantener la rotación). Mira también aprendizajes recientes (`learning_list` con `inContext: true`) y señales propias (`own_sources_list`, con `outcome: "open"`).
2. Elige con el usuario el ejercicio:
   - Sesión mensual → una idea por bucket hasta ocho al mes.
   - 100→10→1 → 100 ideas, quedarse con 10, elegir 1.
   - Libre para algo puntual. Son plantillas, no cuotas.
3. Genera candidatas **sin juzgarlas**. Fuentes preferentes: outliers propios, outliers externos, tendencias con moderación, señales reales del creador, cruces tema × formato que ya funcionó.
4. Guarda las candidatas con `ideas_capture_batch` (1–20 por llamada en `items`: `workingTitle`, `origin`, `originRef`; y a nivel de llamada `bucketId` y `selection`). Una llamada por bucket si van a buckets distintos. En 100→10→1, pregunta al usuario si quiere guardar las 100 o solo las 10 finalistas (`selection: "shortlisted"`).
5. Filtra en la conversación: descarta lo que no cumple los criterios (dilo con su motivo) y guarda solo las finalistas, ya como `selection: "shortlisted"`.
6. Para cada finalista seria:
   - `ideas_brief_update` (`expectedBriefRevision`: la `brief.revision` que da `packaging_get` para esa ficha —0 si aún no hay brief; capturar con origen ya lo crea—; después, la que devuelve cada respuesta): `audience` (destinatario), `problem`, `reward` (gran recompensa: qué sabrá o conseguirá el espectador al final), `framing` (enfoque), `ownContribution`, `intendedRole` (`reach | conversion | mixed`), `technicalBrief` (lo que el creador sabe y hay que explicar; hechos por comprobar marcados).
   - `ideas_criteria_set` con los cinco criterios (ver tabla) y su argumento.
   - Demanda: búscala y regístrala en el brief (ver [reference/demanda.md](reference/demanda.md)).
7. Presenta al usuario 2–3 finalistas con recompensa, criterios y evidencia en pocas líneas. Él elige (o delega la elección explícitamente).
8. Elegir: `videos_select_idea` (`expectedRevision` de la ficha —la `revision` de su tarjeta en `pipeline_list` o en la respuesta de `ideas_capture_batch`— y `expectedBriefRevision`; `reason`). Exige gran recompensa y destinatario, o `exceptionReason`. No mueve de etapa.
9. Después: si hay plan, `plans_slots_assign` (hueco libre de su bucket: `byBucket[].missingSlotIds` en `plans_progress_get`); `pipeline_move` (`toStage: "packaging"`) lo pasa a Packaging; si falta algo, responde `DEPENDENCY_NOT_READY` con `details.missing`.

## Los cinco criterios

| Clave | Criterio | Cómo evaluarlo |
| --- | --- | --- |
| `tam` | Mercado total disponible alto: a cuánta gente podría interesar. | Orientativo, sin cifra exacta. Reformula para ampliar («5 secretos de un centro comercial» → «de los mejores centros comerciales del mundo»). Interprétalo según audiencia y rol del vídeo. |
| `evergreen` | Interesa hoy, hace dos años y dentro de dos años. | Excepción consciente: tendencia o noticia relevante del nicho (dura poco). |
| `single_image` | Fácil de representar en una miniatura. | **Innegociable**: si no se puede visualizar, se descarta antes de seguir. |
| `difference` | Grado de diferencia frente a lo que ya existe. | Información distinta o más completa, unir outliers, mejor producción, experiencia propia o formato distinto. |
| `repeatable` | Formato repetible (plantilla de título con una variable). | Genera ideas más rápido y colas de reproducción; negociable para una idea puntual que merezca la pena. |

Estados: `not_evaluated` (≠ no cumple), `supported`, `questioned`, `insufficient_data`. No hay nota numérica.

> **Nota sobre guardar candidatas:** `ideas_capture_batch` admite 20 ideas por llamada. En el 100→10→1, genera y filtra las 100 en la conversación y guarda en tubepoli solo las ~10 finalistas (`selection: "shortlisted"`) dentro de la sesión; guarda también las descartadas solo si el usuario quiere conservarlas para más adelante.

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- Cruces tema × formato guardados: `patterns_combinations_preview` (no guarda nada) e `ideas_patterns_cross` (propone texto, no cambia el título).
- Descartar una idea ya guardada: `ideas_discard` (`expectedRevision` de la ficha, `reason`; se puede recuperar con `ideas_restore`). Marcar finalistas ya capturadas: `ideas_shortlist`.
- Evidencia de demanda como objetos: `ideas_evidence_add` (ver [reference/demanda.md](reference/demanda.md)).
- Leer el brief y sus versiones: `ideas_brief_get`.
- Sesiones con plantilla: `ideas_session_create` (`name`, `template`: `monthly_8`, `funnel_100_10_1` o `free`) y `sessionId` en `ideas_capture_batch`; cerrarla con su decisión: `ideas_session_close` (`expectedRevision`, `decision`, `chosenVideoIds`).
- Ficha completa y qué le falta: `videos_get`. Comprobar sin mover: `pipeline_validate_move`. Fecha comprometida: `commitments_set`.

## Reglas del método

- 100 ideas a la semana si publicas una por semana; si cuesta, 10 al día con una alarma y un empujón final.
- Descartar es más fácil que elegir: quita las 90 peores. «Una idea es tan buena como las que descartaste para llegar a ella».
- Con recursos, desarrolla el packaging de las 10 y pregunta a gente en cuál haría clic.
- Sesión mensual: cada idea encaja en un bucket, misma cantidad por bucket, cruza el tema con un formato que ya funcionó y cumple los cinco criterios.
- Antes de delegar: enfoque, gran recompensa y brief técnico del creador.
- Demanda: elige un objetivo realista de vistas y busca vídeos del tema que lo alcancen; 500.000 es el valor por defecto editable de tubepoli. En vídeos de conversión o B2B, la evidencia comercial vale.

## Errores que evitar

- Evaluar mientras se genera (bloquea la creatividad).
- Anotar temas («vídeo sobre cafeína») en vez de títulos («Dejé la cafeína 30 días»).
- Convertir los criterios en una puntuación o marcar `supported` sin argumento.
- Rechazar una idea B2B solo porque no hay un vídeo de 500.000 vistas.
- Elegir la idea sin gran recompensa ni destinatario (la operación lo rechazará).
- Romper la rotación: cinco ideas para un bucket y ninguna para otro.

## Ejemplos

- «Hagamos las ideas de noviembre» → `plans_progress_get` → proponer 2 por bucket → con el usuario, quedarse con 8 → `ideas_capture_batch` (`selection: "shortlisted"`, una llamada por bucket) → brief y criterios de cada una.
- «¿Es buena idea "Cómo uso n8n para facturar"?» → criterios: TAM bajo (solo quien conoce n8n) → reformular hacia el deseo («Automatiza tu facturación sin programar») → buscar demanda → `ideas_brief_update` (`demandDecision: "reframe"` y la referencia en `contextRefs`).
- «Elige tú la mejor de las tres» → explicar la recomendación con criterios y evidencia → con su confirmación (o delegación), `videos_select_idea` → `pipeline_move` a `packaging`.
