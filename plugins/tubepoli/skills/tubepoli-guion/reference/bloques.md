# Bloques del guion en tubepoli

Un guion es una lista de bloques con ID estable. Cada bloque tiene su propia revisión: `scripts_blocks_update` edita 1–20 bloques a la vez (todo o nada), cada uno con su `expectedRevision`.

## Campos de un bloque

| Campo | Uso |
| --- | --- |
| `kind` | `intro`, `section`, `rehook`, `cta`, `ending`, `other` |
| `role` | `execution`, `connection`, `deepening`, `closing` (o `null`) |
| `title`, `purpose` | Qué tiene que pasar en el bloque y para qué |
| `outline` | Esquema (puntos y subpuntos) |
| `spoken` | Texto literal (lo que se dice) |
| `intro` | Solo en la intro: `promise`, `proof`, `plan`, `picture`, `pain` |
| `conflict` | Problema del que parte el bloque (conflicto-contexto) |
| `resources` | Lista completa de tomas/recursos: `kind` (`shot`, `photo`, `broll`, `screen`, `graphic`, `other`), `description`, `status` |
| `proofs` | `add`/`update`/`remove`: `claim`, `sourceKind` (`own_experience`, `demo`, `data`, `m12_source`, `m17_story`, `url`, `other`), `sourceRef`, `verification` (`verified`, `declared`, `unsupported`), `permission` |

Los `resources` declarados aquí son la lista de tomas de Producción. Declara lo que haya que grabar o preparar: demostraciones, pantallas, fotos para la miniatura, gráficos.

## Plantillas de `scripts_create`

- `three_acts`: Intro → Ejecución → CTA → Conexión → Reenganche → Profundización → Cierre. Añade un segundo reenganche entre Ejecución y Conexión si el vídeo lo necesita.
- `single_change`: Intro → El cambio → Cómo hacerlo → Por qué hacerlo → Pruebas → Cierre.
- `blank`: solo la intro.

La estructura es orientativa: reparte el contenido entre los bloques con `scripts_blocks_update` (cambiando `kind`, `role` o `title` si hace falta) y deja vacío lo que sobre.

## Pruebas y avisos

`scripts_select` avisa de hechos por comprobar, permisos pendientes, recursos pendientes, promesa de Packaging cambiada y miniatura solo en concepto. Cuéntale al usuario los avisos relevantes antes de grabar.

- Experiencia del creador: `proofs.add` con `sourceKind: "own_experience"` (y `sourceRef: {type: "own_signal", id}` si sale de una señal), solo con permiso para usarla en público.
- Dato o fuente externa: `sourceKind: "data"` o `"url"` con su `url`.
- Lo que diga el creador sin respaldo: `verification: "declared"`. Nunca marques `verified` sin evidencia.

## Prueba de utilidad (opcional, vídeo central o lead magnet)

Para comprobar que alguien consigue el resultado siguiendo la explicación: pídele al creador que lo pruebe con 2–3 personas parecidas al cliente ideal y registra lo que pase como aprendizaje (`learning_record` con `kind: "utility"` y el `videoId`). Es una prueba editorial, no demuestra causalidad.

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- Añadir, quitar o reordenar bloques: `scripts_blocks_add`, `scripts_blocks_remove`, `scripts_blocks_reorder`.
- Historia del banco: `proofs.add` con `sourceKind: "m17_story"` y `sourceRef: {type: "story_card", id}` (solo historias con permiso público).
- Fuente de conocimiento: `sourceKind: "m12_source"` con la fuente asociada (`sources_link` al bloque si quieres trazabilidad).
- Prueba de utilidad estructurada: `pilots_record` (material, público piloto, resultado esperado, acción) y, con resultados, `pilots_review`.
