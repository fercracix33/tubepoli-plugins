# Plugins de tubepoli

Marketplace oficial de [tubepoli](https://tubepoli.com), el sistema de trabajo para canales de YouTube. Contiene el plugin **tubepoli** para Claude, ChatGPT y Antigravity: conecta tu IA con tu cuenta de tubepoli (tú eliges canales y permisos al autorizar) y le añade las guías del método para que trabaje paso a paso contigo.

Añadir el marketplace en lugar de subir el archivo del plugin tiene una ventaja: cuando publicamos una versión nueva de las guías, te llega sin volver a instalar nada.

## Añadirlo en Claude

1. En Claude, ve a **Personalizar › Plugins**, pulsa **Añadir › Añadir marketplace › Añadir desde un repositorio**.
2. Escribe `fercracix33/tubepoli-plugins` y confirma.
3. Instala el plugin **tubepoli**, abre su pestaña de conectores y pulsa **Conectar**.
4. Se abrirá tubepoli: elige tu canal y pulsa **Autorizar**.

En Claude Code: `/plugin install tubepoli --marketplace fercracix33/tubepoli-plugins` (o `/plugin marketplace add fercracix33/tubepoli-plugins` y después `/plugin install tubepoli@tubepoli`).

## Añadirlo en ChatGPT

En la web, ChatGPT conecta solo los servidores añadidos a mano, así que la conexión y las guías van por separado:

1. **Conexión:** en la web de ChatGPT, ve a **Plugins › Personales**, pulsa **Añadir › Añadir servidor MCP personalizado**, escribe `tubepoli` como nombre, `https://tubepoli.com/mcp` como dirección y **OAuth** como autenticación, y crea el plugin. Ábrelo, instálalo (**+**) y pulsa **Conectar**; se abrirá tubepoli para que elijas tu canal y pulses **Autorizar**.
2. **Guías:** en **Plugins › Personales**, pulsa **Añadir › Añadir un marketplace**. En **Origen** escribe `fercracix33/tubepoli-plugins` y en **Referencia de Git**, `main`; deja vacías las rutas dispersas. Instala el plugin **tubepoli** del marketplace.
3. En modo Work, escribe `@tubepoli` en un chat.

En Codex: `codex plugin marketplace add fercracix33/tubepoli-plugins --ref main`.

## Añadirlo en Antigravity

Antigravity instala el plugin directamente desde este repositorio (la raíz es el plugin: `.antigravity-plugin/plugin.json`, `mcp_config.json` y las guías):

1. En la terminal: `agy plugin install https://github.com/fercracix33/tubepoli-plugins`.
2. La primera vez que tu agente use tubepoli se abrirá el navegador para iniciar sesión: elige tu canal y pulsa **Autorizar**.

Si solo quieres la conexión (sin las guías), añade el servidor MCP `https://tubepoli.com/mcp` desde los ajustes de MCP de Antigravity o en `mcp_config.json`. Antigravity no muestra las vistas visuales de tubepoli: las herramientas `show_*` responden con un resumen en texto y el enlace para abrirlo en tubepoli.

## Actualizaciones

Cada plugin lleva un número de `version` en su `plugin.json`, y lo subimos cada vez que cambian las guías.

- **ChatGPT:** pulsa el botón de refrescar (↻) junto al buscador de Plugins.
- **Claude:** abre **Personalizar › Plugins › tubepoli** y actualízalo desde el marketplace. Si lo subiste con un archivo .zip, quítalo y añade este marketplace: el .zip no se actualiza.
- **Claude Code:** `/plugin marketplace update tubepoli`.
- **Antigravity:** vuelve a ejecutar `agy plugin install https://github.com/fercracix33/tubepoli-plugins` (el plugin se reemplaza por la versión nueva).
- **Codex:** vuelve a ejecutar `codex plugin marketplace add fercracix33/tubepoli-plugins --ref main`.

Cada guía lleva al principio «Versión de estas guías»; tu IA la compara con la vigente en tubepoli y te avisa si toca actualizar.

## Contenido

- `.claude-plugin/marketplace.json`: catálogo para Claude, que apunta a `plugins/tubepoli/`.
- `.agents/plugins/marketplace.json`: catálogo para ChatGPT y Codex, que apunta a `plugins/tubepoli-chatgpt/`.
- `.antigravity-plugin/plugin.json` y `mcp_config.json`: el plugin de Antigravity (usa las guías y el logo de `plugins/tubepoli/`).

Este repositorio se genera automáticamente desde tubepoli, así que no aceptamos cambios aquí. Para dudas o soporte: https://tubepoli.com/soporte · hola@tubepoli.com
