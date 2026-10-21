# Formulario de captación

Plantilla única: nombre + email + 1–3 preguntas de cualificación + consentimiento. Nace en borrador (versión 1); cada cambio de contenido crea una versión nueva e inmutable.

## Ejemplo de `content` para `business_forms_create`

```json
{
  "headline": "Descarga la guía para automatizar tu facturación",
  "intro": "La descargas al momento, en la página de agradecimiento.",
  "questions": [
    { "id": "empleados", "label": "¿Cuántas personas trabajan en tu empresa?", "type": "single_choice", "required": true,
      "options": [{ "id": "e1_9", "label": "1–9" }, { "id": "e10_49", "label": "10–49" }, { "id": "e50_mas", "label": "50 o más" }] }
  ],
  "qualificationMode": "all",
  "qualificationRules": [{ "questionId": "empleados", "operator": "is", "optionIds": ["e10_49", "e50_mas"] }],
  "consentText": "Acepto la política de privacidad.",
  "privacyPolicyUrl": "https://tu-dominio.com/privacidad",
  "deliveryMode": "thank_you_page",
  "bookingUrl": null
}
```

(Ejemplo ilustrativo: pregunta y umbral los decide el usuario según su cliente ideal; por ejemplo, un despacho que solo atiende empresas de 10 o más empleados.)

## Criterios

- Identificadores de preguntas y opciones: letras, números, `-` o `_` (≤40).
- Pregunta solo lo que cambia una decisión comercial. Cada pregunta añade fricción.
- Preguntas estructuradas (`single_choice`, `multiple_choice`) para reglas deterministas; las de texto libre quedan para revisión.
- `privacyPolicyUrl` es obligatoria; `consentText` tiene un texto por defecto que puedes ajustar. El opt-in de marketing es aparte y opcional.
- `bookingUrl` solo si quien cumple puede reservar directamente.

## Publicar y medir

- `business_forms_publish` comprueba que el lead magnet tiene recurso entregable y, si entrega por email, que el dominio está verificado (si no, `DEPENDENCY_NOT_READY`).
- Tras los primeros leads: `business_funnel_summary` por versión de formulario (`groupBy: "form_version"`) para ver si un cambio de preguntas altera volumen o calidad. Pocas respuestas no son una validación.

## Avanzado (requiere el catálogo completo: https://tubepoli.com/mcp/completo)

- `business_forms_pause` / `business_forms_archive` dejan de aceptar envíos (efecto público).
- `business_forms_update` cambia preguntas o textos creando una versión nueva.
