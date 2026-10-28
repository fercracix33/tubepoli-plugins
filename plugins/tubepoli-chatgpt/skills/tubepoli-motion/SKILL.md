---
name: tubepoli-motion
description: "Crea en tubepoli escenas animadas (motion graphics) para un vídeo: intro, titular cinético, dato o gráfico animado, explicación paso a paso, cierre. Sigue el método por fases: brief, storyboard por beats aprobado por el usuario, escena de prueba, resto de escenas, revisión con tira de fotogramas y hoja de contactos con crítica P0/P1/P2, colocación en el montaje y render. Escenas en HTML con CSS/WAAPI, SVG o canvas, deterministas y con la marca del canal. Úsala cuando el usuario diga «hazme una intro animada», «anima este dato», «quiero un gráfico animado», «un titular que entre con fuerza», «explica esto con una animación», «haz el storyboard» o «mejora esta escena»."
---

# Escenas animadas en tubepoli (motion)

Una escena animada es código (HTML + CSS/WAAPI, SVG o canvas) que tubepoli renderiza fotograma a fotograma: se edita con una frase y se vuelve a renderizar en minutos. Funciona muy bien para lo estructurado (tipografía cinética, datos, diagramas, logos, intros) y mal para lo orgánico (personas realistas, física real): no lo intentes.

## Cuándo usarla

- El vídeo necesita una intro, un titular, un dato, un gráfico o una explicación visual que la grabación no da.
- Ya hay proyecto de montaje (si no, primero `tubepoli-montaje`).
- Para rótulos estándar del catálogo (rótulo inferior, subtítulos) basta `tubepoli-montaje`; esta guía es para piezas a medida.
- **Siempre con `tubepoli-retencion`**: cuántas escenas (1–2 por minuto en vídeos sin cara), cuánto duran y cómo se critican.

## Antes de empezar

1. `edit_timeline_get` (`projectId`): duración, clips y `revision` vigente; mira dónde irá la escena.
2. Lee la marca del canal y las escenas y storyboard que ya existan (Avanzado, «Marca» y «Storyboard»). **Si la marca no está configurada, pídesela al usuario y guárdala antes de animar** (Avanzado, «Marca»): con la paleta por defecto todo parece plantilla.
3. Pregunta al usuario lo mínimo: qué debe entender el espectador, tono (sobrio, enérgico, didáctico) y duración aproximada.

## Flujo paso a paso

1. **Brief** — en 2–4 frases: objetivo de la pieza, público, tono, duración total y dónde va en el montaje. Enséñaselo.
2. **Storyboard** — escribe los beats (título, texto en pantalla, duración, cómo se ve) con `edit_storyboard_set`. En un vídeo sin cara, cubre 1–2 escenas por minuto de narración (no 3 escenas para 3 minutos). Revisa los avisos: ningún texto en pantalla menos de 1,2 s ni más de ~3 palabras por segundo. Enséñaselo como lista numerada y **no animes hasta que el usuario lo apruebe**; apruébalo solo con su confirmación literal (o que lo apruebe él en la pestaña Storyboard).
3. **Escena de prueba** — crea UNA escena (la más representativa) con `edit_scenes_create` siguiendo el contrato técnico ([reference/escenas.md](reference/escenas.md)). Si el lint devuelve errores, corrígelos antes de seguir.
4. **Míralo tú** — `edit_scenes_preview` (`mode: "strip"`: 12 instantes, más densos en entradas y salidas; `sheet` para la hoja 4×3) y el resultado en `edit_renders_get` (`waitSeconds: 55`, `includeImages: true`). Las imágenes te llegan con su descripción: mira de verdad, no supongas. Si le enseñas un vídeo al usuario, pásale `render.appUrl` (no caduca), nunca `videoUrl`.
5. **Crítica P0/P1/P2** — escribe la lista antes de tocar nada:
   - **P0 (bloquea):** texto cortado, solapado o ilegible; algo fuera de la zona segura (márgenes de escritorio y de Shorts); fotogramas vacíos o rotos; contraste insuficiente.
   - **P1 (corrige):** cola estática (la animación acaba y la escena sigue quieta: un beat cada 2–3 s o acórtala), ritmo (texto que no da tiempo a leer, cambios bruscos), jerarquía (lo importante no destaca), entradas y salidas incoherentes, rebotes que no encajan con el tono.
   - **P2 (pule):** espaciado, curvas de movimiento, detalles de marca.
   Corrige P0 y P1, vuelve al paso 4. Dos o tres vueltas bastan; si no mejora, simplifica la escena.
6. **Enséñala** — con el visto bueno de la prueba, haz el resto de escenas del storyboard (paso 3–5 cada una), colócalas en el montaje con `edit_scenes_place` (como capa encima del vídeo o insertadas en la pista base), corrige con `edit_scenes_update`, pasa `edit_craft_check` y revisa con `show_editor`.
7. **Render** — `edit_render_request` (`kind: "preview"`, con `range` del tramo de las escenas) y, con el visto bueno del usuario, el `final` como en `tubepoli-montaje`.

## Reglas del método

- **Una idea por beat** y texto corto: si hay que leer mucho, no es una escena, es un párrafo.
- **Legible**: titulares ≥ 1,2 s en pantalla, ~3 palabras/s como máximo, contraste alto, tipografía de la marca.
- **Movimiento con intención**: curvas suaves (ease-out/in-out) por defecto; nada de rebotes elásticos salvo que el tono del canal lo pida.
- **Marca del canal, no plantilla**: colores y fuente de la marca, composición propia; evita el aspecto genérico de «todo entra rebotando».
- **Zona segura**: el texto lejos de los bordes (≈ 5 % en escritorio; en Shorts, lejos del tercio inferior y del lateral derecho).
- **Sonido aparte**: las escenas no llevan audio; la música y los efectos van en la pista de audio y por debajo de la voz.
- El usuario decide: aprueba el storyboard, ve la prueba y elige el final.

## Errores que evitar

- Animar sin storyboard aprobado, o aprobarlo sin la confirmación literal del usuario.
- Hacer todas las escenas a la vez antes de validar una de prueba.
- Dar una escena por buena sin mirar la tira de fotogramas.
- Usar reloj de pared, temporizadores, azar sin semilla, red o GSAP (el lint lo rechaza y el render no sería determinista).
- Escenas largas y densas: mejor varias de 3–8 s que una de 40 s.
- Escenas que terminan de animarse en 2 s y se quedan quietas 7 s.
- Personas, caras o movimiento orgánico realistas: no salen bien con código.

## Ejemplos

- «Hazme una intro de 5 s con el título del vídeo» → brief → storyboard de 2–3 beats → aprobación → escena de prueba con titular cinético (CSS) → tira de fotogramas → crítica → colocarla insertada al inicio → `edit_render_request` (`preview`, `range: [0, 6]`) → `show_editor`.
- «Anima que el 73 % de los canales no cuida la intro» → beat único de ~3 s → escena SVG con la cifra que cuenta y una barra que se llena → hoja de contactos → colocarla como capa en el momento en que lo dice → preview del tramo.
- «La escena se lee muy rápido» → leer el storyboard → alargar el beat o recortar texto → actualizar la escena con su duración nueva → tira de fotogramas → preview.

## Avanzado (vía operations_call)

- Storyboard: `edit_storyboard_get`, `edit_storyboard_set` (`brief`, `beats: [{id, title, onScreenText, durationSeconds, visualNotes, sceneId}]`, `expectedRevision`; devuelve `warnings` de legibilidad), `edit_storyboard_approve` (`approved`, `userConfirmation` con lo que dijo el usuario, literal).
- Escenas: `edit_scenes_create` (`title`, `html` = solo el cuerpo, `durationSeconds`, `sceneId` opcional, `notes`, `place` opcional `{at, mode: overlay | insert, durationSeconds}`; devuelve `lintScene`), `edit_scenes_update` (`sceneId`, `html`/`title`/`durationSeconds`/`notes`), `edit_scenes_get` (cuerpo tal como lo escribiste y lint), `edit_scenes_list` (con sus colocaciones), `edit_scenes_place` (`sceneId`, `at`, `mode`), `edit_scenes_remove`.
- Vista previa: `edit_scenes_preview` (`sceneId`, `mode: strip | sheet | video`, `width`) devuelve `render.id` y los instantes; el resultado (imágenes o vídeo 540p de la escena sola) está en `edit_renders_get` (`renderId`, `waitSeconds: 55`, `includeImages: true`): espera en el servidor en lugar de sondear.
- Marca: `channels_brand_get` (colores y fuente; en las escenas, `var(--tp-primary)` y compañía).
- Verificación del montaje completo: `edit_contact_sheet`, `edit_snapshot`, `edit_check` como en `tubepoli-montaje`.
