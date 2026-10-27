---
name: tubepoli-retencion
description: "Aplica en tubepoli los baremos de montaje para retención antes y durante el montaje de un vídeo largo o un Short: ritmo de cambio visual por tramo, duración máxima de planos e imágenes, variedad de material sin repetir, tiempos de lectura, subtítulos, música bajo la voz, escenas animadas que no parezcan plantilla, Shorts con gancho en 1–2 s, y la revisión de oficio P0/P1/P2 sobre la hoja completa antes de dar nada por terminado. Incluye el modo «vídeo sin grabación» (locución + material). Úsala cuando el usuario diga «que retenga lo máximo posible», «móntalo con ritmo», «hazlo dinámico», «no tengo grabación, solo la voz», «revisa si el montaje engancha», «por qué se ve aburrido», «hazme un Short que enganche» o antes de cualquier montaje o motion."
---

# Montaje para retención en tubepoli

Retener es que nadie encuentre un motivo para irse: cada plano tiene que aportar algo nuevo y la promesa del título se cumple desde el primer segundo. Los números de esta guía son **baremos** razonables, no reglas de YouTube: si la curva de retención del canal dice otra cosa, manda la curva. Úsala junto con `tubepoli-montaje` (cómo editar) y `tubepoli-motion` (escenas animadas).

## Cuándo usarla

- Antes de montar cualquier vídeo, para planificar ritmo y material.
- Cuando no hay grabación: locución más imágenes, vídeos de stock o generados (modo «sin grabación», abajo).
- Antes de un preview o del render final: la revisión de oficio y la crítica P0/P1/P2.
- Para Shorts, nuevos o sacados de un vídeo largo.

## Antes de empezar

1. `production_get` (`videoId`) y `scripts_get`: guion elegido, bloques, duración prevista y materiales.
2. `edit_timeline_get`: duración real, qué hay en cada pista. Mira si hay **cara** (tomas grabadas con su voz en la pista 0) o es **sin grabación** (la voz en la pista 10 y el resto, material).
3. La marca del canal tiene que estar configurada antes de animar o rotular (colores, fuente, logo). Si no lo está, pídela y guárdala (Avanzado, «Marca»): sin marca todo sale con la paleta por defecto y parece plantilla.

## Flujo paso a paso

1. **Planifica por tramos** (vídeo largo; ver [reference/baremos.md](reference/baremos.md)): intro 0:00–0:30 con un cambio visual cada ≤ 2–3 s y la imagen de la miniatura; cuerpo con un cambio cada 3–6 s; reenganches de 5–10 s en cada cambio de acto; segunda mitad algo más pausada; cierre sin despedida.
2. **Lista de planos ANTES de montar** — por cada bloque del guion: qué se ve en cada frase (cara, B-roll, escena, rótulo), cuánto dura y de qué archivo sale. En vídeos sin cara, sigue [reference/sin-grabacion.md](reference/sin-grabacion.md): presupuesto de material y 1–2 escenas animadas por minuto en el storyboard (`edit_storyboard_set`).
3. **Comprueba el material** — segundos únicos de vídeo e imagen frente a la duración de la narración. **Si falta, para**: di al usuario cuánto falta y pide o genera más (stock, B-roll, escenas). Nunca rellenes repitiendo el mismo clip ni alargando imágenes quietas.
4. **Monta** con `edit_ops_apply` (y las escenas con `edit_scenes_create` / `edit_scenes_place`). Cada respuesta trae `craft`: el resumen de oficio de la composición resultante. Si aparece un P0 o P1, corrígelo antes de seguir añadiendo.
5. **Revisión de oficio** — `edit_craft_check` (`detail: true`): repetición de material, imágenes fijas sin movimiento, planos largos para su tramo, escenas que se quedan quietas, poco motion, textos sin tiempo de lectura, voz sin subtítulos, música encima de la voz, marca sin configurar y, en Shorts, gancho, zona segura y duración. Cada hallazgo trae instante, clips y `fixHint`.
6. **Míralo de verdad** — preview (`edit_render_request`, `kind: "preview"`) y la hoja de contactos del montaje completo (Avanzado, «Verificación»); el resultado, en `edit_renders_get` (`waitSeconds: 55` para esperar sin consultar en bucle; `includeImages: true` para ver las imágenes). Haz la crítica con [reference/checklist.md](reference/checklist.md): escribe la lista P0/P1/P2 **antes** de decir que está bien. Si en la hoja se ve el mismo plano dos veces, es un P1 aunque nadie lo haya avisado.
7. **Corrige y repite** 4–6 hasta que no queden P0 ni P1. Solo entonces propón el render final (`edit_check` con el render para sonoridad, negros y congelados).
8. **Shorts** — ver «Shorts» en [reference/baremos.md](reference/baremos.md): tramos de 15–35 s, gancho en texto desde el primer fotograma, subtítulos dinámicos en el tercio medio y bucle (Avanzado, «Shorts»). `edit_craft_check` con `profile: "short"`.

## Reglas del método

- **Orden de impacto**: montaje (qué se queda, en qué orden, qué se quita) > música > gráficos > efectos de sonido.
- **Nada de relleno**: cada plano aporta algo nuevo. Un mismo archivo como mucho 2 veces y con tramos distintos.
- **Nada quieto**: imagen fija ≤ 3 s, o ≤ 5 s con zoom/paneo lento (por defecto ya entra con deriva suave); escena animada ≤ 10 s con un beat cada 2–3 s.
- **Se lee**: texto en pantalla ≥ 0,3 s por palabra + 0,5 s; rótulos de ≤ 5 palabras; intro subtitulada entera; contraste alto.
- **Música 15–20 dB bajo la voz** (con ducking) y sonoridad final ≈ −14 LUFS.
- **Marca, no plantilla**: colores y fuente del canal, 3–5 plantillas de texto fijas, layouts variados.
- **No declares terminado** sin `edit_craft_check` sin P0/P1 y sin la crítica escrita sobre la hoja completa.

## Errores que evitar

- Repetir el mismo clip (o empezar siempre en el segundo 0) para cubrir la duración de la locución.
- Imágenes fijas de 6–10 s sin movimiento.
- 2–3 escenas animadas para un vídeo de 3 minutos sin cara.
- Subtítulos solo en la intro; música alta sin ducking.
- Mirar la hoja de contactos y no actuar sobre lo que se ve (planos repetidos, estáticos).
- Animar con la paleta por defecto porque la marca no está configurada.
- Dar el montaje por bueno con un preview que no se ha podido ver entero.

## Ejemplos

- «Tengo la locución de 3 minutos y unas imágenes; móntalo para que retenga» → leer guion y material → lista de planos por bloque → presupuesto: «hay 40 s de vídeo único para 176 s: faltan ~25 planos; ¿busco stock o genero más?» → storyboard con 3–5 escenas (`edit_storyboard_set`) → montar → `edit_craft_check` → hoja de contactos → crítica P0/P1/P2 → corregir → preview.
- «¿Por qué se ve aburrido?» → `edit_craft_check` (`detail: true`) → explicar los 3 hallazgos más graves con su instante → proponer los arreglos (`fixHint`) → aplicarlos con su visto bueno.
- «Sácame un Short de esto» → propuestas de 15–35 s con gancho (Avanzado, «Shorts») → crear el Short con su rótulo de gancho → `edit_craft_check` (`profile: "short"`) → preview.

## Avanzado (vía operations_call)

- Marca: `channels_brand_get`, `channels_brand_set` (colores, fuente, logo) antes de animar.
- Material: `edit_broll_add` (imágenes con `motion: "auto"` por defecto: deriva suave; vídeo ya usado entra desde un tramo libre), `edit_stock_search`, `edit_stock_import`, `edit_assets_import_url`, `production_materials_attach` (subir lo generado fuera, con su procedencia).
- Subtítulos y sonido: `edit_captions_generate` (también con la locución en la pista de audio), `edit_audio_ducking_set`, `edit_transitions_set`.
- Verificación: `edit_contact_sheet` (`mode: "review"`: escenas, cortes y vista móvil; `mode: "grid"` del vídeo entero), `edit_snapshot`; el resultado en `edit_renders_get` (`includeImages: true` para las imágenes).
- Shorts: `edit_shorts_suggest` (15–35 s por defecto; cada propuesta trae `hookText` y `loopHint`), `edit_shorts_create` (`fromSuggestion` o `range`; `hook` = promesa del primer fotograma).
