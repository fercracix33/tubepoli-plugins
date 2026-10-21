# Búsqueda en YouTube y cuota

La cuota de la YouTube Data API es diaria y compartida (día del Pacífico). En el núcleo, la investigación no usa la búsqueda de YouTube: los canales llegan por URL o @handle y su inventario por playlist, que es mucho más barato.

| Acción | Herramienta | Coste |
| --- | --- | --- |
| Añadir URL/@handle (canal o vídeo) | `research_resolve` | 1–2 unidades |
| Inventario de subidas de un canal | `research_channels_sync` | ~1 unidad por 50 vídeos + detalles |
| Ver el inventario con el ratio sobre la mediana | `research_channels_get` | Ninguno |
| Guardar un vídeo del inventario como referencia | `references_save` con `fromInventory` | Ninguno |

Reglas:

- Pide permiso antes de lo que gaste cuota y di cuántos canales vas a añadir o sincronizar.
- Para encontrar canales candidatos, usa tu propia búsqueda web o los que conozca el usuario; luego `research_resolve`.
- CTR, retención o ingresos de vídeos ajenos no existen en la API. No los estimes.
- Suscriptores ocultos o vistas no devueltas = desconocido, nunca 0.
- Si recibes `QUOTA_EXCEEDED`, trabaja con lo ya sincronizado o espera `retryAfterSeconds`.

## Avanzado (vía operations_call)

| Acción | Herramienta | Coste |
| --- | --- | --- |
| Cuota restante | `budgets_get` | Ninguno |
| Ver qué haría una búsqueda, filtros admitidos y coste | `search_preview` | Ninguno |
| Buscar en YouTube | `search_run` con `scope: "youtube"` | 1 búsqueda por página (bolsa diaria limitada) + unidades |
| Buscar en el inventario de canales seguidos | `search_run` con `scope: "followed_channels"` | Ninguno |
| Buscar en referencias guardadas | `search_run` con `scope: "saved"` o `references_list` | Ninguno |
| Volver a filtrar un lote ya recuperado | `search_local` (`runId`) | Ninguno |
| Recuperar una búsqueda anterior | `search_runs_get` | Ninguno |
| Guardar una búsqueda recurrente | `searches_save` | Ninguno |
| Refrescar vistas de referencias | `research_refresh` | ~2 unidades por 50 |

- Pide permiso antes de `search_run` con ámbito `youtube` y di cuántas páginas vas a usar (`pageBudget`). `search_preview` lista como no admitidos los filtros que la API no da (CTR, retención…).
- Los filtros locales (vistas, duración, suscriptores…) se aplican **sobre lo recuperado**: «no aparece» significa «no está en este lote», no «no existe en YouTube».
- Benchmark del nicho: `research_benchmark` calcula la media de las medianas de vistas de tus canales de referencia seguidos (con muestra y fecha). Sirve para distinguir un nicho pequeño de vídeos que no funcionan; no es la media de todo el nicho.
