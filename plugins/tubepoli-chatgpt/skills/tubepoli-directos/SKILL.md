---
name: tubepoli-directos
description: "Programa, emite y modera directos de YouTube desde tubepoli: crea la emisión con su stream para OBS, pasa a prueba, a directo y termina con comprobaciones, lee el chat de forma eficiente (solo lo nuevo, preguntas detectadas aparte), escribe, borra, silencia, bloquea, nombra moderadores, lanza encuestas y pausas publicitarias, resume Super Chat de 30 días, vincula el vídeo resultante a su ficha y sincroniza miembros del canal cuando YouTube concede el acceso. Úsala cuando el usuario diga «programa un directo», «dame la clave para OBS», «empieza el directo», «qué preguntan en el chat», «silencia a este», «lanza una encuesta», «pon anuncios», «termina el directo», «cuánto Super Chat», «quiénes son mis miembros»."
---

> Versión de estas guías: 0.10.0 (compárala con guides.latestVersion de capabilities_get).

# Directos y miembros (M20)

Un directo es público por naturaleza: **cada efecto en YouTube (programar, editar, borrar, pasar a prueba, a directo o terminar, escribir en el chat, moderar, encuestas, anuncios) exige `confirm: true` y la aprobación explícita del usuario para esa acción** (o para un lote concreto que él haya nombrado: «silencia a todos los que pongan enlaces durante el directo»). Leer no necesita confirmación. Todas estas operaciones están en el catálogo completo: ejecútalas con `operations_call` `{"operation": "<nombre>", "input": {…}}`.

## Cuándo usarla

- El usuario quiere hacer un directo, prepararlo, emitirlo o moderarlo.
- Pregunta por Super Chat o por los miembros del canal.

## Requisitos

- Canal de YouTube conectado desde la web con el permiso de gestión (`youtube.force-ssl`). Si falta, el error trae `fixHint`: reconectar en **Ajustes → YouTube** con ese permiso.
- El canal debe tener los directos activados en YouTube (`youtube_liveStreamingNotEnabled` → el usuario lo activa en YouTube Studio; tarda hasta 24 h la primera vez).

## Flujo paso a paso

Todas las operaciones de directos y miembros están en el catálogo completo: los pasos concretos, con nombres y campos, están en «Avanzado (vía operations_call)» justo debajo.

1. Mira qué directos hay (programados, en curso, terminados) y su siguiente paso; si el usuario los creó en Studio, sincroniza primero.
2. Programa con su OK a título, día, hora, privacidad y «contenido para niños» (pregúntalo siempre).
3. Entrega la URL y la clave del stream para OBS una sola vez.
4. Con la señal de OBS llegando: prueba → (OK) directo. Durante el directo, chat: lee solo lo nuevo, resume preguntas y Super Chat, y modera o escribe con OK.
5. Termina la emisión con OK explícito y vincula el vídeo resultante a su ficha.

## Avanzado (vía operations_call)

### Programar y emitir

1. `live_broadcasts_list`: directos guardados por grupo (`upcoming`, `active`, `completed`) con `nextStep`. Si el usuario los creó en Studio, `live_broadcasts_sync`.
2. Programar (con su OK al título, día, hora y privacidad): `live_broadcasts_schedule` con `operationId`, `confirm: true`, `title`, `scheduledLocal` `{date, time, timeZone}` (zona del espacio) o `scheduledStartAt`, `privacy`, `settings` opcionales (`latency` normal/low/ultraLow — ultraLow sin subtítulos—, `dvr`, `madeForKids` **siempre preguntado**, `autoStart`, `midrollEveryMinutes`) y `videoId` si sale de una ficha. Crea emisión + stream y los enlaza.
3. OBS: `live_stream_key` devuelve URL de ingesta y **clave secreta**. Dásela solo a la persona; no la repitas después, no la pegues en el chat ni la guardes en notas.
4. Cuando OBS ya envía señal: `live_broadcasts_get` (mira `health.streamStatus = active` y avisos). Después `live_broadcasts_transition` `to: "testing"` (vista previa privada), y con su OK `to: "live"`. Si la señal no llega, el error `stream_inactive` lo dice: no reintentes en bucle, pide revisar OBS.
5. Al acabar: `live_broadcasts_transition` `to: "complete"` (irreversible; pide confirmación aunque «ya terminó en OBS»).
6. Vídeo resultante: asocia la ficha (`live_broadcasts_update` solo con `videoId` y `expectedRevision`, no toca YouTube) y vincúlalo con `publication_link_existing` (`youtube` = `youtubeBroadcastId`). A partir de ahí se analiza y se sacan Shorts como cualquier vídeo (guías `tubepoli-publicacion-y-analisis` y `tubepoli-montaje`).

### Chat durante el directo

1. `live_chat_read` (`broadcastId`, `limit` 20–50): mensajes recientes compactos, `questions` (preguntas detectadas), `superChats`, `pageToken` y `pollAfterMs`. En la siguiente lectura pasa el `pageToken` tal cual para traer **solo lo nuevo**, y espera al menos `pollAfterMs`. Cada lectura cuesta cuota: no leas más de una vez por minuto salvo que el usuario esté moderando contigo.
2. Resume al usuario: preguntas pendientes primero, Super Chat a agradecer, mensajes problemáticos. No respondas en su nombre sin su OK al texto.
3. Acciones (cada una con `operationId`, `confirm: true`):
   - Escribir como el canal: `live_chat_send` (`text` ≤ 200).
   - Borrar: `live_chat_delete` (`messageId`).
   - Silenciar: `live_chat_ban` `type: "temporary"`, `durationSeconds` (300 = 5 min). Bloquear: `type: "permanent"`. Deshacer: `live_chat_unban` con el `youtubeId` devuelto.
   - Moderadores: `live_chat_moderators_list`, `live_chat_moderators_add` (`authorChannelId`), `live_chat_moderators_remove`.
   - Encuesta: `live_poll_start` (`question` ≤ 100, 2–4 `options` ≤ 35) y `live_poll_end` (`pollMessageId`).
   - Anuncios: `live_broadcasts_ad_break` (`durationSeconds` 30–180; solo canales monetizables). Mid-rolls automáticos: `settings.midrollEveryMinutes` al programar o editar.
4. `no_live_chat`: el chat existe a partir de «En prueba»; si ya empezó en Studio, `live_broadcasts_sync`.

### Super Chat y miembros

- `live_superchats_list`: Super Chat y Super Stickers de los últimos 30 días (YouTube no guarda más) con `totals` por moneda. Para «cuánto saqué en el directo» filtra por fecha.
- Miembros: primero `members_status`. `state`:
  - `not_enabled` → la integración aún no está activa en este despliegue; dilo sin prometer fecha.
  - `pending_access` → YouTube concede el permiso de miembros a petición: comparte `accessRequest.steps` y `accessRequest.url`.
  - `ready` → `members_sync` (`mode: "all_current"` la primera vez, `"updates"` después) y `members_list` (`level`, `since`).

## Reglas del método

- Un directo es una publicación: nada sale a YouTube sin confirmación explícita para esa acción o lote.
- El directo alimenta el método como cualquier vídeo: al terminar, a su ficha, a análisis y a Shorts.
- Las preguntas del chat son materia prima para ideas: propón capturar las buenas como ideas tras el directo.

## Errores que evitar

- Pasar a directo o terminar «porque tocaba» sin un OK explícito para esa transición.
- Mostrar la clave del stream más de una vez o en un resumen.
- Leer el chat en bucle sin `pageToken` (repite mensajes y gasta cuota).
- Contestar preguntas del chat inventando datos del creador.
- Declarar `madeForKids` por tu cuenta.

## Ejemplos

- «Prográmame un directo el jueves a las 19:00 de preguntas» → propón título, privacidad y si es para niños → con su OK, programar (vía operations_call) → «Listo. ¿Te paso la URL y la clave para OBS?».
- «¿Qué preguntan?» (en directo) → leer el chat con el último `pageToken` → «3 preguntas nuevas: … ¿Respondo alguna por escrito?».
- «Silencia a Pepe 10 minutos» → silenciar 600 s (`type: "temporary"`, `confirm: true`) → «Hecho; si te equivocaste lo deshago».
