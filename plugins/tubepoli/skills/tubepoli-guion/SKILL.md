---
name: tubepoli-guion
description: "Escribe el guion de un vídeo en tubepoli en menos de una hora con el método: empezar por la gran recompensa (el final), intro de tres P (promesa, prueba, plan; Picture y Pain como ingredientes), estructura en ejecución, conexión, profundización y cierre, reenganches, conflicto-contexto, bloques con pruebas verdaderas, CTA hacia el primer tercio y estructura de un solo cambio para vídeos educativos. Úsala cuando el usuario diga «escríbeme el guion», «hazme la intro», «estructura este vídeo», «convierte esto en guion», «revisa mi guion» o «prepara el teleprompter»."
---

# Guion y storytelling (M07)

La retención depende del guion, no de la cámara ni de la edición. Y la intro es la parte que más audiencia pierde: si tienes 4 horas, dedica 3 a la intro.

## Cuándo usarla

- La ficha tiene pareja de packaging elegida (o al menos el concepto de título y miniatura en el que hará clic la gente).
- El usuario pide intro, estructura, guion literal o esquema, o revisar uno existente.

## Flujo paso a paso (receta de menos de una hora)

1. `scripts_get` (`videoId`): promesa y pareja elegidas, gran recompensa y brief técnico, bloques con su `blockId` y revisión, palabras y duración estimada. Si `script` es `null`, `scripts_create` (`template`: `three_acts` por defecto o `single_change` para un educativo de un solo cambio; `mode`: `outline` o `literal`).
2. **Empieza por el final**: confirma la gran recompensa (qué tiene que haber pasado al final para que la promesa sea verdad). Si no existe, guárdala antes con `ideas_brief_update` (`reward`).
3. **Intro (≤30 s)** en el bloque `intro` con `scripts_blocks_update` (`scriptId`, `items: [{blockId, expectedRevision, changes}]`) → `changes.intro`:
   - `promise`: decir la promesa del título en voz alta en los primeros segundos.
   - `proof`: por qué tú (autoridad, experiencia) o el esfuerzo puesto en este vídeo; qué lo hace distinto. Búscalo en el contexto del canal (`channels_context_get`, sección «Autoridad» del cuerpo) o pregúntalo.
   - `plan`: «primero A, después B y por último C».
   - Opcionales: `picture` (cómo será el resultado) y `pain` (lo que se ahorrará).
   - Mantén la brecha de curiosidad: no reveles la respuesta en la intro.
4. **Esquema**: de la intro a la recompensa, lista en desorden los pasos que llevarían a alguien del punto A al B; luego subpuntos de cada paso. Guárdalos en `outline` de cada bloque.
5. **Ordena en actos** y asigna `role` a cada bloque:
   - `execution` (≈primer 25 %, «del 1 al 3»): avanzar mucho, contar pronto lo más valioso.
   - `connection` (≈siguiente 25 %, «del 3 al 6»): historias reales, casos de clientes, datos, anécdotas; que el espectador conecte con quien habla.
   - `deepening` (≈último 50 %): lo denso pero imprescindible.
   - `closing`: entrega la recompensa y lleva al siguiente vídeo.
   La plantilla `three_acts` ya trae los bloques en este orden; reparte el contenido entre ellos y deja vacío lo que no hace falta para llegar al final.
6. **Reenganches** entre actos: el bloque `rehook` de la plantilla («hasta ahora has conseguido X; ahora vas a descubrir Y»). Si el vídeo necesita un segundo reenganche tras la ejecución, ver «Avanzado».
7. **Conflicto-contexto**: cada bloque (y cada subpunto) parte de un problema (`conflict`) que el bloque resuelve; así el esquema se vuelve casi literal.
8. **Pruebas e historias**: en los bloques de conexión, usa historias reales del creador (señales `experience` en `own_sources_list`, o pregúntale) y añade pruebas con `scripts_blocks_update` → `proofs.add` (`claim`, `sourceKind`, `verification`, `permission`). Nada inventado: si no hay historia real, deja el hueco marcado.
9. **CTA**: `business_ctas_get` (`videoId`); si no hay, `business_ctas_set` (`role`, `destination`, `momentPct` ≈33, `text`, `placements`). En el guion, bloque `cta` hacia el primer tercio.
10. Revisa la duración estimada que da `scripts_get` (estimación, no tiempo de grabación) y, con el visto bueno del usuario, `scripts_select` (`scriptId`, `expectedRevision`; guarda la versión actual y la elige para grabar). Después `pipeline_move` (`toStage: "production"`). Teleprompter: entrega al usuario el texto `spoken` de los bloques en orden.

Detalle de bloques y pruebas: [reference/bloques.md](reference/bloques.md). Estructura de un solo cambio: [reference/un-solo-cambio.md](reference/un-solo-cambio.md).

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- Estructura: `scripts_blocks_add` (p. ej. `kind: "rehook"`), `scripts_blocks_reorder`, `scripts_blocks_remove`.
- Historias y autoridad: `scripts_stories_suggest` (solo historias con permiso público) y `authority_list`.
- Duración y versiones: `scripts_estimate_duration`, `scripts_save_version` (versión editorial sin elegirla).
- Exportar: `scripts_export` (`format: "teleprompter"` o `format: "production_pack"`).

## Reglas del método

- Packaging primero: la intro reafirma la promesa del título y la miniatura.
- La intro nunca se improvisa: palabra por palabra. El resto puede ir en esquema si el creador improvisa mejor; en la fase de prueba basta intro definida, orden de temas y cómo se conectan.
- Duración: ~13 min es un punto de partida, no una norma; las proporciones se adaptan a cualquier duración.
- Todo el equipo sabe en qué minuto y en qué parte del vídeo está trabajando.
- Rehooks: reenganchar y recordar el plan en el paso de un acto a otro.
- Contrastes (opcional, si sobra tiempo): alternar táctico/anécdota, positivo/negativo, plano claro/oscuro.
- Historias verdaderas y, mejor, de alguien parecido al cliente ideal; salen de las reuniones y llamadas de la semana.
- CTA al lead magnet hacia el 33 % del vídeo, no al final; si tiene sentido, úsalo dentro del vídeo.
- Educativo: un solo cambio de comportamiento por vídeo.

## Errores que evitar

- Escribir la intro antes de tener claro el título y la miniatura.
- Revelar en la intro la respuesta que promete el título.
- Dejar lo más denso al principio o lo más valioso al final.
- Añadir historias, cifras o testimonios que el creador no ha vivido o no puede probar.
- Pisar el trabajo de otra persona: `scripts_blocks_update` lleva `expectedRevision` por bloque; ante `REVISION_CONFLICT`, compara y propone.
- Elegir la versión para grabar sin enseñarla al usuario.

## Ejemplos

- «Escríbeme el guion de este vídeo» → `scripts_get` → (`scripts_create`) → recompensa → intro 3P → esquema por actos → `scripts_blocks_update` en lote → CTA → «¿Lo damos por bueno para grabar?» → `scripts_select`.
- «Solo la intro» → `scripts_get` → tres propuestas de intro (≤30 s) → la elegida en `changes.intro` y `spoken` del bloque intro.
- «Es un tutorial para que mis clientes facturen solos» → `scripts_create` con `single_change` → cambio, cómo, por qué (beneficios/costes) y pruebas (datos, historia real, lógica).
