# Chequeos tras publicar (0 h, 24 h, 72 h, 7 días)

Son recordatorios condicionales: cada uno puede concluir «esperar» o «datos insuficientes».

## Antes de cada chequeo

1. ¿Hay datos? `analytics_checks_list` dice qué datos se esperan en cada momento y `analytics_diagnose` qué hay ya, con su cobertura. La API llega con demora (los informes de alcance pueden tardar hasta 48 h); los datos de primeras horas y el desglose nuevos/recurrentes solo están en Studio → `analytics_import` como declarados.
2. ¿Cuánta muestra? Impresiones y vistas absolutas, no solo porcentajes.
3. ¿Con qué comparo? Un vídeo propio parecido en la **misma ventana** de días desde la publicación (`analytics_diagnose` con `referenceVideoId` y `days`).
4. ¿En qué nivel está el canal? En nivel 0, el problema suele ser que YouTube aún no sabe a quién enseñar los vídeos (audiencia, idea, buckets), no la miniatura.

## Qué mirar en cada momento

| Momento | Qué mirar | Acción posible (solo con muestra suficiente) |
| --- | --- | --- |
| 0 h (al publicar) | Impresiones, vistas, CTR frente a lo normal del canal | CTR especialmente bajo → revisar miniatura (y título con ella). Impresiones especialmente bajas → revisar título y metadatos (y la idea). |
| 24 h | Curva de retención, AVD/AVP | Una caída fuerte en una parte concreta → valorar recortar (p. ej. la intro); relaciónala con los bloques del guion (`scripts_get`). |
| 72 h | Visión global del vídeo | Decidir si se cambia algo o se deja. |
| 7 días | Rendimiento general | Conclusión y aprendizajes documentados para el próximo vídeo (`learning_record`). |

En pruebas de título/miniatura, quédate con la opción que dé más **tiempo total de visualización** (CTR × retención), no solo más CTR.

## Registrar

`analytics_checks_record` con:
- `conclusion`: `wait` (esperar), `insufficient_data`, `review` (revisar algo concreto), `experiment` (probar un cambio).
- `observation` (hechos), `hypothesis` (por qué), `action` (qué haremos y dónde).
- `referenceVideoId` si comparaste.
- `createLearning: true` para dejar el aprendizaje enlazado.

Con 0 h o muestra pequeña, la herramienta avisa si eliges `review` o `experiment`: explica al usuario por qué esperar suele ser lo correcto. `skip: true` omite un chequeo que no aplica.

## Cómo contarlo al usuario

Tres líneas: dato con su cobertura («300 impresiones en 22 h; CTR 4,1 %, muestra pequeña»), comparación («tu vídeo de referencia tenía 1.200 a las 24 h») y decisión («esperamos al chequeo de 72 h»). Nunca conviertas un dato desconocido en 0.

## Avanzado (vía operations_call)

- Qué informes hay y su demora: `analytics_available_reports`; detalle de un vídeo: `analytics_video_get`.
- Retención mapeada a los bloques del guion: `analytics_retention_map`.
