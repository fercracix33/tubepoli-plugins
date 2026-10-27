# Vídeo sin grabación (locución + material)

Cuando no hay nadie a cámara, **todo lo que se ve es material que hay que conseguir**. El fallo típico: una locución de 3 minutos con 40 s de vídeo distinto, rellenada repitiendo clips e imágenes quietas. Se evita planificando antes de montar.

## 1. Lista de planos por bloque del guion

Antes de tocar la línea de tiempo, escribe (y enséñale al usuario) una tabla por bloque:

| Bloque | Frase (o tramo) | Qué se ve | Tipo | Dura | Archivo / fuente |
| --- | --- | --- | --- | --- | --- |
| Gancho | «Si tu perro mayor…» | perro despierto de noche, plano corto | vídeo | 2 s | stock / generado |

- Un plano nuevo cada 2 s en la intro y cada 3–4 s después (ver los baremos).
- Tipos: vídeo (stock, generado o propio), imagen con movimiento, escena animada (dato, lista, proceso, idea clave), rótulo sobre vídeo.
- Las ideas abstractas, cifras y listas van como escena animada; las situaciones concretas, como vídeo.

## 2. Presupuesto de material

- Suma los segundos **únicos** que necesitas por tipo y compáralos con lo que hay: duración de cada vídeo (tramos distintos cuentan aparte) e imágenes (≤ 3–5 s cada una).
- Escenas animadas: 1–2 por minuto en el storyboard (`edit_storyboard_set`), de 3–8 s cada una.
- **Si falta material, para**: dile al usuario cuántos planos o segundos faltan y de qué tipo, y propón cómo conseguirlos (buscar stock, generarlos fuera y subirlos, más escenas). No rellenes repitiendo.

## 3. Montaje

- Locución en la pista 10; vídeos e imágenes en la base (0) o como B-roll (5); escenas en su pista; subtítulos de toda la locución.
- Un mismo vídeo como mucho 2 veces y en tramos distintos (al repetir un archivo, tubepoli elige un tramo libre y avisa `footage_reused` si no queda).
- Imágenes siempre con movimiento (deriva suave por defecto); ningún plano quieto más de 3–5 s.
- Cada respuesta de edición trae `craft`; antes de seguir, `edit_craft_check` sin P0 ni P1.

## 4. Antes de decir que está terminado

- Hoja de contactos del vídeo entero y crítica escrita P0/P1/P2 (ver [checklist.md](checklist.md)): si un plano aparece dos veces, si hay tres fotogramas casi iguales seguidos o si la escena se queda quieta, se corrige.
- Preview completo visto (con audio) y `edit_check` del render: sonoridad, negros, congelados.
