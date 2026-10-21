---
name: tubepoli-negocio-y-leads
description: "Monta y opera en tubepoli la conversión del canal en clientes: oferta, lead magnet, formulario de 1–3 preguntas de cualificación con consentimiento, enlaces atribuibles por vídeo y ubicación, CTA de cada vídeo (hacia el primer tercio), vídeo central y recorrido, mini CRM de leads con sus estados y medición del embudo. Úsala cuando el usuario diga «quiero conseguir clientes con el canal», «crea un lead magnet», «hazme el formulario», «pon el enlace en la descripción», «qué CTA pongo», «¿cuántos leads me ha traído este vídeo?», «revisa mis leads» o «¿por qué no convierte?»."
---

# Negocio, leads y conversión (M16)

Nivel 2 del método: con vistas, el siguiente problema es convertirlas en leads cualificados. Las tres palancas son lead magnet, atribución por vídeo y colas de reproducción.

## Cuándo usarla

- El objetivo del canal es `leads` o `revenue`, o el foco activo es `create_lead_magnet`.
- Hay que preparar la conversión de un vídeo concreto (CTA, enlace, texto).
- El usuario quiere revisar leads o el rendimiento del formulario.

## Flujo paso a paso — Montar la conversión

1. `business_setup_get`: ofertas, lead magnets, formularios, enlaces por vídeo, estado del correo y **siguiente paso recomendado**.
2. **Oferta**: defínela con el usuario (nombre, tipo, promesa, audiencia, problema, precio, evidencia) y escálala hacia arriba y hacia abajo de precio hasta encontrar el recurso gratuito. Si no está en tubepoli, guárdala en el contexto del canal (`channels_context_update` → `bodyMarkdown`, sección «Oferta»; lee el cuerpo y añade) o que la persona la cree en Ajustes › Negocio.
3. **Lead magnet**: `business_magnets_register` (`name`, `problem` que resuelve del todo, `nextProblem` que revela, `result`, `offerId` si la oferta existe, `format`, `deliveryUrl`). Sin `deliveryUrl` no se puede entregar.
4. **Formulario** (nace en borrador): `business_forms_create` (`name`, `leadMagnetId`) con `content`:
   - `headline`, `intro`; **1–3 preguntas** de cualificación (`short_text`, `single_choice`, `multiple_choice`).
   - `qualificationRules` (p. ej. «menos de 10 empleados» no cumple).
   - `privacyPolicyUrl` (obligatoria), `consentText` (hay uno por defecto) y `marketingOptInText` opcional.
   - `deliveryMode`: `thank_you_page` (enlace al recurso al enviar) salvo que `business_setup_get` diga que el correo tiene dominio verificado (entonces `email`); `bookingUrl` si quien cumple puede reservar.
   Ver [reference/formulario.md](reference/formulario.md).
5. Entrega por correo con dominio propio: la persona la configura en Ajustes › Negocio (registros DNS). Sin dominio, `thank_you_page`.
6. **Publicar el formulario** (efecto público): enseña el resumen y, con un «sí» explícito, `business_forms_publish` (`formId`, `expectedRevision`).
7. **Enlace por vídeo y ubicación**: `business_forms_links_create` (`formId`, `videoId`, `placement`: `description`, `pinned_comment`, `end_screen_qr`, `video_mention`, `social`, `other`). Cada vídeo, su enlace.
8. **CTA del vídeo**: `business_ctas_get` (`videoId`) y `business_ctas_set` (`role`: `reach | conversion | mixed`; `destination`: `lead_magnet | central_video | offer | booking`; `linkId`; `momentPct` ≈33; `text`; `placements`: `script`, `description`, `pinned_comment`, `end_screen`; `awarenessLevel`; `expectedRevision` si ya existe). En el guion, bloque CTA (ver `tubepoli-guion`); en el paquete, primera línea de la descripción y comentario fijado (ver `tubepoli-publicacion-y-analisis`).
9. **Recorrido**: revisa con `business_setup_get` (enlaces por vídeo) y `business_ctas_get` de cada vídeo que los vídeos de conversión tengan enlace y CTA ubicada, y que los de alcance lleven al vídeo central o al lead magnet (vídeos de alcance → vídeo central → lead magnet → oferta).

## Flujo paso a paso — Operar

1. Leads: `business_leads_list` (filtros por `status`, `qualification`, `videoId` de origen; devuelve totales). **Datos personales: no los copies a resúmenes generales.**
2. Cualificación y estado comercial (`new → qualified → call → client` o `discarded`): la cualificación por reglas es automática; el estado comercial lo lleva la persona en la web (Negocio › Leads). Un clic en el enlace de reserva **no** es una llamada. tubepoli no envía nada al lead.
3. Embudo: `business_funnel_summary` (visitas, envíos, leads, cumplen criterios, entregas, clics de reserva; `groupBy: "video" | "link" | "form_version"`). Tasas solo con numerador y denominador compatibles.
4. Señales de audiencia: lo que repiten los leads en sus respuestas, resumido sin datos personales, va como señal (`own_sources_capture`, `sourceType: "lead_response"`).
5. Ajustes del formulario, enlaces y pruebas del lead magnet, y derechos del lead (exportar o borrar sus datos): con `operations_call` (ver «Avanzado»).

## Avanzado (vía operations_call)

- Oferta: `business_offers_create` (`name`, `kind`, `promise`, `audience`, `problem`, `priceText`, `evidence`; `ladderUp`/`ladderDown`).
- Correo con dominio propio: `domains_status` → `domains_create` → la persona publica los registros DNS → `domains_verify`.
- Recorrido: `business_ctas_list` (avisos: vídeos de conversión sin enlace, CTA sin ubicar) y `business_journey_get`. Niveles de consciencia por bucket: `buckets_awareness_set`.
- CRM: `business_leads_qualify` (`meets | does_not_meet | pending_review`, `source: "external"`), `business_leads_update` (estado comercial), `business_leads_notes_add`.
- Señales: `business_forms_answers_summary` (anonimizado) → `own_sources_import` con `origin.module: "m16_leads"`.
- Ajustes: `business_forms_update` (crea versión nueva; el público ve la publicada hasta volver a publicar), `business_forms_links_update`, `business_utility_tests_create`/`business_utility_tests_update`.
- Efectos públicos: `business_forms_pause`, `business_forms_archive`, `business_deliveries_retry` (solo con confirmación).
- Derechos del lead: `business_leads_export` (con `leadId`: todos sus datos) y `business_leads_delete` (irreversible; solo a petición expresa).

## Reglas del método

- **Lead magnet**: algo gratis o casi, a cambio del contacto y el permiso; da a quien está listo la información para decidir y filtra a quien no lo está.
- **Ridículamente bueno**: resuelve del todo al menos un problema concreto de una audiencia concreta; pregúntate si podrías cobrar 50–300 € por él; pruébalo con varias personas y mejora donde se atascan.
- **Vídeo central** muy de nicho y muy valioso, rodeado de vídeos de alto TAM que llevan a él con la pantalla final; al final revela el siguiente problema y el siguiente paso.
- **Fricción**: más preguntas = menos leads pero más cualificados; si llegan leads poco cualificados, añade cualificación o haz los vídeos más de nicho.
- **CTA hacia el 33 % del vídeo**, enlace en la primera línea de la descripción y en el comentario fijado; usar el lead magnet dentro del vídeo si tiene sentido.
- **Atribución por vídeo** (UTM o enlace propio) para saber qué vídeos y buckets traen leads.
- **Colas de reproducción**: cada vídeo termina llevando a otro; el CTA se ve varias veces.
- **Niveles de consciencia**: un bucket de alcance por curiosidad y los demás bajando a la audiencia hasta estar lista para comprar. Anótalo en la definición del bucket y en `awarenessLevel` de la CTA.

## Errores que evitar

- Publicar el formulario o reintentar entregas sin confirmación: son efectos públicos.
- Prometer entrega por email sin dominio verificado.
- Poner el CTA solo al final del vídeo.
- Un único enlace para todos los vídeos (se pierde la atribución).
- Copiar nombres o emails de leads a contextos generales o a otras herramientas.
- Confundir «cumple criterios» con «cliente», o «clic en reservar» con «llamada».

## Ejemplos

- «Quiero captar clientes para mi consultoría con el canal» → `business_setup_get` → definir la oferta → proponer lead magnet → `business_magnets_register` → `business_forms_create` (2 preguntas, `thank_you_page`) → (sí) `business_forms_publish`.
- «Prepara la conversión de este vídeo» → `business_ctas_get` → `business_forms_links_create` (descripción y comentario fijado) → `business_ctas_set` (`momentPct: 33`) → texto del CTA para el guion.
- «¿Qué vídeos me traen leads?» → `business_funnel_summary` (`groupBy: "video"`) → resumen con conteos y cobertura, sin datos personales.
