---
name: tubepoli-canal-youtube
description: "Guía para crear, conectar y dejar a punto el canal en YouTube desde tubepoli: asistente de creación (cuenta nueva o existente, canal de marca), perfil (descripción, palabras clave, país, idioma, traducciones, tráiler, «para niños»), banner y marca de agua con la marca del canal, página de inicio (secciones ligadas a buckets), listas y podcast, y lo que solo se hace en Studio. Siempre plan con diff y cuota antes de aplicar y confirmación literal de la persona. Úsala cuando el usuario diga «crea mi canal de YouTube», «conecta YouTube», «ponme el banner», «arregla la descripción del canal», «organiza la página de inicio», «crea las listas por bucket», «haz un podcast de esta lista» o «cambia el nombre/handle/foto»."
---

> Versión de estas guías: 0.10.0 (compárala con guides.latestVersion de capabilities_get).

# Canal en YouTube (M20)

El canal es el escaparate: banner, descripción, página de inicio y listas deciden si quien llega se suscribe y si ve un segundo vídeo. tubepoli lo edita por la API oficial de YouTube con permiso de gestión, **siempre** enseñando antes el plan. Lo que YouTube no deja tocar por API (nombre, @handle, foto, enlaces) se deja preparado para pegar en Studio.

## Cuándo usarla

- El canal aún no existe en YouTube o no está conectado a tubepoli.
- Tras definir contexto y buckets: toca alinear perfil, página de inicio y listas con ese plan.
- El usuario pide cambiar banner, descripción, secciones, listas, marca de agua o un podcast.

## Flujo paso a paso

1. **Situación.** `capabilities_get` y `context_get` (tema, audiencia, objetivo). Después el estado de la conexión (Avanzado, «Asistente»): te dice qué paso toca y el último error de autorización.
2. **Crear y conectar** (si falta). YouTube no crea canales por API: explica las tres opciones y que la persona elija — con su cuenta de Google, **canal de marca** (nombre propio, varias personas gestionando; recomendado para un proyecto o negocio) o con una cuenta de Google nueva. Dale el enlace exacto del asistente; cuando lo tenga, la conexión la autoriza la persona titular en el navegador (tú nunca ves tokens). Para editar hace falta un segundo permiso, «gestión»: pídeselo solo cuando vaya a cambiar algo. Si la autorización falla con `no_channel_returned`, eligió una cuenta sin canal.
3. **Lee antes de proponer.** Perfil y vista previa en directo (Avanzado, «Leer»): descripción, palabras clave, banner, secciones con sus vídeos, listas y cuota restante. YouTube reemplaza cada parte entera, por eso tubepoli siempre lee antes de escribir.
4. **Propón con el contexto**: la primera línea de la descripción dice a quién ayudas y qué gana al suscribirse; palabras clave = tema + nombre; idioma y país del público; tráiler = vídeo corto que explique el canal. Una lista pública por bucket principal (`buckets_list`) y ligada a ese bucket.
5. **Plan antes de aplicar.** Cada escritura se llama primero con `dryRun: true`: devuelve `diff` (antes → después), `plan.steps` (llamadas) y `plan.totalUnits` (cuota). Enséñalo en lenguaje claro y pregunta. Solo con un «sí» explícito repite con `confirm: true` y `userConfirmation` = la frase literal de la persona. Agrupa cambios del mismo recurso en una sola confirmación.
6. **Imagen.** Banner 2560×1440 con todo lo importante en la zona segura central 1546×423 (lo único que se ve en móvil); marca de agua cuadrada ~150 px con fondo transparente. La web de tubepoli los genera con la marca (Canal → YouTube → Banner y marca de agua); por chat, sube la imagen y aplícala (Avanzado, «Imagen»). Si YouTube rechaza el banner por API (`status: "partial"`), da el enlace de Studio y el archivo preparado.
7. **Página de inicio.** Máx. 10 secciones; el orden enviado es el de la página. Arriba lo que mejor convierte (vídeos populares o el tráiler), después una lista por bucket. Las listas deben ser públicas.
8. **Lo que solo se hace en Studio** (nombre, @handle, foto, enlaces, pantallas finales, comunidad): prepáralo (Avanzado, «Studio») y entrega valor + enlace exacto + pasos.
9. **Cierra** contando qué se aplicó, qué quedó en Studio y la cuota usada. Todo queda en el registro de cambios del canal y en la actividad.

## Reglas del método

- Nunca apliques nada en YouTube sin enseñar antes el plan y recibir la confirmación literal; un «sí» a un cambio no vale para el siguiente.
- Una descripción no es un currículum: primera línea con la promesa, después temas y ritmo de publicación.
- Página de inicio corta y ordenada por intención; nada de secciones vacías.
- Una lista por bucket, con título que se buscaría y descripción útil; mismo orden que el plan.
- «Para niños» se declara con honestidad: si es sí, YouTube desactiva comentarios y notificaciones.
- Respeta la cuota: lecturas 1 u., cada escritura 50 u. Si no alcanza hoy, dilo y deja el plan para mañana.

## Errores que evitar

- Mandar `localizations` o `sections` incompletas: YouTube borra lo que no envías.
- Poner texto o logo fuera de la zona segura del banner.
- Destacar el propio canal en «Canales destacados» o usar listas privadas en la página de inicio.
- Prometer que se puede cambiar el nombre, el @handle o la foto por API.
- Reintentar a ciegas tras un error: lee `error.details.fixHint`.

## Ejemplos

- «Créame el canal en YouTube» → `context_get` → asistente: elegir canal de marca → enlace → «avísame cuando lo tengas» → conexión → nombre y @handle preparados para Studio.
- «Pon una descripción que convierta» → leer perfil → propuesta con la audiencia del contexto → `dryRun` → «Esto cambia la descripción (50 u.). ¿La aplico?» → «sí, aplícala» → `confirm` con esa frase.
- «Organiza la página de inicio por buckets» → `buckets_list` → listas existentes → crear las que falten (una confirmación por lote) → secciones: populares + una lista por bucket → plan → confirmar.

## Avanzado (vía operations_call)

- Asistente: `channels_youtube_setup_get` (pasos, enlaces y último error); conectar `channels_youtube_connect` (con `extraScopes: ["youtube.force-ssl"]` para gestión).
- Leer: `channels_youtube_profile_get` (perfil, `canManage`, lo que solo va en Studio, cambios recientes y cuota), `channels_youtube_preview_get` (página del canal con sus vídeos), `channels_youtube_sections_get`, `channels_youtube_playlists_list`, `channels_youtube_playlists_items_get`.
- Perfil: `channels_youtube_profile_set` (solo los campos enviados; `localizations` es la lista completa y exige idioma).
- Imagen: `channels_brand_get` (colores, fuente y logo), `channels_youtube_banner_set` (`assetId` 16:9 ≥2048×1152), `channels_youtube_watermark_set` (`timing.mode`: `entire`, `end`, `custom`), `channels_youtube_watermark_unset`.
- Página de inicio: `channels_youtube_sections_set` (lista completa y ordenada; con `id` conserva, sin `id` crea).
- Listas: `channels_youtube_playlists_save` (crear o editar; `bucketId` solo liga en tubepoli, sin cuota), `channels_youtube_playlists_delete`, `channels_youtube_playlists_items_set` (orden completo de vídeos), `channels_youtube_podcast_set` (`imageAssetId` cuadrada).
- Studio: `channels_youtube_studio_prepare` (nombre, @handle normalizado, enlaces y logo para la foto).
