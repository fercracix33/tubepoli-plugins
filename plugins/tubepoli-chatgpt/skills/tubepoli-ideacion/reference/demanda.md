# Evidencia de demanda de una idea

Una idea limitada por el interés del público tiene un techo de vistas que ningún packaging ni guion puede superar; por eso conviene comprobar la demanda antes de elegirla.

## Procedimiento

1. Fija con el usuario cuántas vistas querría de verdad para este vídeo (10.000, 100.000, 500.000…). tubepoli usa 500.000 por defecto (`demandTargetViews: null`); se cambia en `ideas_brief_update`.
2. Busca en YouTube el título tentativo o palabras parecidas (con permiso si gasta cuota; ver `tubepoli-investigacion`).
3. Si ningún vídeo del tema llega a ese objetivo, la señal es que el tema, tal como está planteado, no interesa a suficiente gente: **reformula** hacia el deseo general (de «5 automatizaciones de n8n para vender más» a «Cómo conseguir más clientes para tu negocio»).
4. Registra lo encontrado.

## Registrar en tubepoli

En el brief, con `ideas_brief_update` (`expectedBriefRevision`, `changes`):

- `demandTargetViews`: el objetivo acordado (`null` = 500.000 por defecto).
- `contextRefs.add`: cada referencia encontrada como `{type: "url", id: <url del vídeo>, label: "<título> · <vistas o «vistas desconocidas»> · <edad>", note: "<consulta, fecha y cobertura de la búsqueda>"}`. La evidencia comercial (clientes que piden esto, ventas, llamadas) o un problema propio documentado van igual, con la señal (`{type: "own_signal", id}`) o una nota en `technicalBrief`.
- `demandDecision` (`continue | research_more | reframe`) y `demandDecisionReason`.

## Cómo interpretarlo

- No encontrar una referencia por encima del objetivo significa «sin esa evidencia en la búsqueda realizada», no «idea mala».
- Vídeos de conversión o B2B específicos: menos vistas pueden convertir mejor. Si hay evidencia comercial, se puede elegir con `exceptionReason` en `videos_select_idea`.
- Vistas desconocidas son `null`, nunca 0. Cuenta la edad del vídeo de referencia.
- Una referencia popular indica interés, no garantiza vistas.

## Avanzado (vía operations_call)

`ideas_evidence_add` (`operationId` obligatorio) guarda cada evidencia como objeto propio:

- `kind: "public_views"`: `query`, `title`, `url`, `views` (o `null` si no se conoce), `ageDays`, `observedOn`, `coverage` (`complete | partial | unknown`) y `coverageNote` (qué buscaste y cuánto).
- `kind: "commercial"`: evidencia de negocio con `note` y `sourceRef`.
- `kind: "own_problem"`: problema propio o de clientes documentado (enlaza la señal de M17 si existe).
