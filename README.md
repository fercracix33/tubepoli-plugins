# Plugins de tubepoli

Marketplace oficial de [tubepoli](https://tubepoli.com), el sistema de trabajo para canales de YouTube. Contiene el plugin **tubepoli** para Claude y para ChatGPT: conecta tu IA con tu cuenta de tubepoli (tú eliges canales y permisos al autorizar) y le añade las guías del método para que trabaje paso a paso contigo.

Añadir el marketplace en lugar de subir el archivo del plugin tiene una ventaja: cuando publicamos una versión nueva de las guías, te llega sin volver a instalar nada.

## Añadirlo en Claude

1. En Claude, ve a **Personalizar › Plugins**, pulsa **Añadir › Añadir marketplace › Añadir desde un repositorio**.
2. Escribe `fercracix33/tubepoli-plugins` y confirma.
3. Instala el plugin **tubepoli**, abre su pestaña de conectores y pulsa **Conectar**.
4. Se abrirá tubepoli: elige tu canal y pulsa **Autorizar**.

En Claude Code: `/plugin install tubepoli --marketplace fercracix33/tubepoli-plugins` (o `/plugin marketplace add fercracix33/tubepoli-plugins` y después `/plugin install tubepoli@tubepoli`).

## Añadirlo en ChatGPT

1. En la web de ChatGPT, ve a **Plugins › Personales** y pulsa **Añadir › Añadir un marketplace**.
2. En **Origen** escribe `fercracix33/tubepoli-plugins` y en **Referencia de Git**, `main`. Deja vacías las rutas dispersas.
3. Instala el plugin **tubepoli** y pulsa **Conectar**.
4. Se abrirá tubepoli: elige tu canal y pulsa **Autorizar**. Después, en modo Work, escribe `@tubepoli`.

En Codex: `codex plugin marketplace add fercracix33/tubepoli-plugins --ref main`.

## Actualizaciones

Cada plugin lleva un número de `version` en su `plugin.json`, y lo subimos cada vez que cambian las guías.

- **ChatGPT:** pulsa el botón de refrescar (↻) junto al buscador de Plugins.
- **Claude:** se actualiza solo si activaste la actualización automática del marketplace; si no, abre el plugin y pulsa **Actualizar**.

## Contenido

- `.claude-plugin/marketplace.json`: catálogo para Claude, que apunta a `plugins/tubepoli/`.
- `.agents/plugins/marketplace.json`: catálogo para ChatGPT y Codex, que apunta a `plugins/tubepoli-chatgpt/`.

Este repositorio se genera automáticamente desde tubepoli, así que no aceptamos cambios aquí. Para dudas o soporte: https://tubepoli.com/soporte · hola@tubepoli.com
