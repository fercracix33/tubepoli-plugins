# tubepoli para tu IA

tubepoli es un sistema de trabajo para canales de YouTube: contexto del canal, estrategia y buckets, ideas, packaging (título y miniatura), guion, producción, publicación, análisis y negocio. Este plugin conecta tu asistente (Claude o ChatGPT) con tu cuenta de tubepoli y le añade las guías del método para que trabaje paso a paso contigo.

## Qué incluye

- **Conector MCP** `https://tubepoli.com/mcp`: lee y guarda en tu espacio de tubepoli con los permisos que concedas al conectar (inicio de sesión OAuth; tú eliges espacio, canales y permisos).
- **Guías del método** (`skills/`): once guías en español, desde «tubepoli-empezar» hasta las rutinas semanales. La IA las carga cuando la tarea encaja.
- **Vistas en el chat**: tablero del Pipeline, ficha de vídeo, comparador de packaging, plan y resumen de análisis.

## Instalación

1. Instala el plugin en tu IA. En Claude: Personalizar › Plugins › Añadir › Subir plugin. En ChatGPT (web): Plugins › Personales › Añadir › Subir archivo comprimido del complemento. También puedes añadir el marketplace `fercracix33/tubepoli-plugins` en cualquiera de las dos para recibir las actualizaciones.
2. Conecta el conector `tubepoli` e inicia sesión con tu cuenta de tubepoli.
3. Pide, por ejemplo: «Usa tubepoli y dime en qué punto está mi canal».

En Claude Code, en un solo paso: `/plugin install tubepoli --marketplace fercracix33/tubepoli-plugins`.

El conector publica como herramientas el núcleo del método (unas 85), las que usa el flujo principal de las guías. El resto del catálogo (lo que las guías marcan como «Avanzado») se usa desde el mismo conector con `operations_search`, `operations_describe` y `operations_call`. En Claude Code o Codex también puedes añadir el catálogo completo como herramientas: `https://tubepoli.com/mcp/completo`.

## Privacidad y soporte

tubepoli está en fase personal (beta). Privacidad: https://tubepoli.com/privacidad · Términos: https://tubepoli.com/terminos · Soporte: https://tubepoli.com/soporte · hola@tubepoli.com
