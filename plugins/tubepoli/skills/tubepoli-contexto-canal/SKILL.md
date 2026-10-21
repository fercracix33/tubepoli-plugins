---
name: tubepoli-contexto-canal
description: "Guía el onboarding y la edición del contexto de un canal en tubepoli (punto de partida, tema, audiencia, objetivo y desarrollo) haciendo una pregunta cada vez y guardando cada respuesta con su procedencia. Úsala cuando el usuario cree un canal, diga «configura mi canal», «a quién le hablo», «define mi audiencia», «actualiza el contexto», «mi canal ya tiene vídeos», o cuando context_get muestre campos pendientes antes de investigar o definir buckets."
---

# Contexto del canal (M01)

El contexto es la base que usan todas las demás guías: sin audiencia ni objetivo claros, la IA trabaja «sin contexto», que es justo lo que el método desaconseja. La estrategia empieza por definir el público lo más específico posible y sus problemas.

## Cuándo usarla

- Canal recién creado o `context_get` con `pendingFields` no vacío.
- El usuario quiere cambiar audiencia, tema u objetivo.
- Antes de proponer buckets si falta la base editorial (`hasEditorialBase: false`).

## Qué se guarda y dónde

| Pregunta (orden del onboarding) | Campo en `channels_context_update` → `changes` | Valores |
| --- | --- | --- |
| 1. «¿En qué punto está tu canal?» | `startingPoint` | `planned` (aún no existe), `existing_without_publications`, `existing_with_publications` |
| 2. «¿De qué trata tu canal?» | `topic` | texto (≤500) |
| 3. «¿A quién quieres ayudar?» | `audience.description` | texto (≤1.500); puede empezar amplio |
| 4. «¿Qué quieres conseguir con el canal?» | `primaryGoal.kind` (+ `primaryGoal.details`) | `reach`, `authority`, `leads`, `revenue`, `other` |
| Opcional al cerrar: «¿Algo más que deba saber?» | `bodyMarkdown` (añadir sección) | Markdown |

Todo lo demás (problemas concretos de la audiencia, conocimiento propio, diferenciación, tono, oferta, recursos, ritmo) va en `bodyMarkdown` cuando la tarea lo necesite, o en su módulo: señales y historias en `tubepoli-investigacion`, oferta en `tubepoli-negocio-y-leads`, ritmo y capacidad en `tubepoli-estrategia-y-buckets`.

## Flujo paso a paso

1. `channels_context_get` con `sections: ["fields","body","inputs"]`. Mira `nextQuestion` y `pendingFields`; anota `revision`.
2. Haz **una sola pregunta**: la de `nextQuestion`. Explica en una frase para qué sirve. Si el usuario ya respondió algo amplio antes, no repitas: propón cómo lo guardarías.
3. Guarda la respuesta con `channels_context_update`:
   - `expectedContextRevision`: la revisión leída.
   - `changes`: solo los campos que la respuesta cubre (omitir conserva; `null` vacía).
   - `originalInput`: `{questionKey, targetField, text}` con el texto literal del usuario (p. ej. `questionKey: "audience"`, `targetField: "audience.description"`).
   - `provenance` por campo: `{source: "user_input", kind: "declaration"}` si lo dijo el usuario tal cual; `{source: "agent_interpretation", kind: "hypothesis"}` si lo resumiste o dedujiste tú. Dilo al usuario.
4. Con la nueva `revision`, repite 1–3 hasta que `nextQuestion` sea `null`. El usuario puede posponer cualquier pregunta: sigue con la siguiente y deja la pendiente como desconocida.
5. Cierre: muestra un resumen de 4 líneas (punto de partida, tema, audiencia, objetivo) y pregunta si hay algo más; si lo hay, añádelo a `bodyMarkdown` (lee el cuerpo vigente y añade una sección, no lo sustituyas).
6. Siguiente paso según el punto de partida:
   - `planned` o sin publicaciones: nivel 0 provisional, foco «completar la prueba de buckets» → `tubepoli-estrategia-y-buckets`.
   - `existing_with_publications`: propón que la persona titular conecte YouTube desde **Ajustes** en la web de tubepoli (tú nunca recibes tokens); mientras tanto, los datos de Studio que te pase se registran como declarados (`analytics_import`). Después, diagnóstico de nivel con evidencia (`tubepoli-estrategia-y-buckets`).

## Avanzado (vía operations_call)

- Conectar YouTube desde el chat: `channels_youtube_connect` (devuelve una URL que abre la persona titular) y comprobar con `channels_youtube_status`.
- Historial del contexto: `channels_context_versions_list`, `channels_context_versions_compare` y, si el usuario lo pide, `channels_context_restore`.

## Reglas del método

- Audiencia lo más concreta posible y con sus problemas reales; busca esos problemas en YouTube para encontrar los canales que ve.
- Un canal sin datos parte de nivel 0 con foco en completar la prueba de buckets; las métricas no observadas son desconocidas, no cero (porque sin datos no se puede saber más, y un cero inventado llevaría a decisiones equivocadas).
- No mezclar nichos: el contexto debe permitir que YouTube entienda a quién enseñar los vídeos.
- El conocimiento técnico del creador es lo que no se puede delegar: guárdalo como desarrollo o brief, no lo inventes.

## Errores que evitar

- Lanzar las cuatro preguntas a la vez o un formulario largo.
- Rellenar tema u objetivo a partir de una respuesta sobre la audiencia sin decirlo (si interpretas, márcalo `agent_interpretation`/`hypothesis` y pide confirmación).
- Sustituir `bodyMarkdown` entero al añadir una nota (se pierde lo anterior).
- Inventar cifras, casos o clientes del creador para «completar» el contexto.
- Tratar «aún no lo sé» como un dato: deja el campo en `null`.

## Ejemplos

- «Mi canal es para empresarios que quieren usar IA en su negocio, quiero conseguir clientes» → guardas en una llamada `audience.description` (declaración) y `primaryGoal.kind: "leads"` con `primaryGoal.details` (literal) → preguntas lo siguiente pendiente («¿En qué punto está tu canal?»).
- «Cambia la audiencia a dueños de pymes de servicios» → `channels_context_get` → `channels_context_update` con `expectedContextRevision` → «Guardado (revisión 5). ¿Revisamos si tus buckets siguen encajando?».
- «Ya tengo 40 vídeos publicados» → `startingPoint: "existing_with_publications"` → proponer conectar YouTube desde Ajustes de tubepoli y después diagnóstico de nivel.
