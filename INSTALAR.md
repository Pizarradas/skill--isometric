# Cómo conectar el kit a cada IA

El kit se apoya en dos formatos que casi todas las herramientas entienden:

- **`AGENTS.md`**: instrucciones en Markdown en la raíz de un proyecto. Lo leen de serie muchos agentes de código.
- **Skills (`skills/<nombre>/SKILL.md`)**: carpetas con instrucciones, referencias y scripts, con frontmatter `name` y `description`. Es el formato abierto de Agent Skills. Si una herramienta no lo carga de forma nativa, basta con que lea los `SKILL.md` como Markdown.

Abajo, la vía recomendada para cada herramienta. Las rutas de configuración cambian entre versiones: si alguna no coincide, busca en la documentación de tu herramienta dónde van las «instrucciones del proyecto», las «rules» o las «skills».

## Usar el kit dentro de un proyecto web

1. Copia la carpeta del kit dentro del proyecto (por ejemplo, en `tools/iso-flat/`) o enlázala.
2. En el `AGENTS.md` (o equivalente) de tu proyecto, añade una línea: `Para ilustraciones isométricas, sigue tools/iso-flat/AGENTS.md.`

## Por herramienta

| Herramienta | Qué hacer |
|---|---|
| **Claude Code** | Abre el kit como proyecto: `CLAUDE.md` importa `AGENTS.md`. Para usarlo en otros proyectos, copia `skills/*` a `.claude/skills/` (o a `~/.claude/skills/` para todos tus proyectos) y `agentes/iso-ilustrador.md` a `.claude/agents/`. |
| **Claude (app y web)** | Sube cada carpeta de `skills/` comprimida en .zip como skill en la configuración, o crea un Proyecto con `AGENTS.md` y los `SKILL.md` como conocimiento. Con el escritorio vinculado y el MCP de Blender, Claude puede ejecutar los scripts en tu equipo. |
| **Codex (CLI e IDE)** | Lee `AGENTS.md` de la raíz automáticamente. Si tu versión soporta skills, copia `skills/*` a su carpeta de skills. |
| **Cursor** | Lee `AGENTS.md` del proyecto. Para reglas persistentes, crea `.cursor/rules/iso-flat.mdc` con una línea que remita a `AGENTS.md`. |
| **GitHub Copilot** (VS Code y agente) | Lee `AGENTS.md`. Alternativa: `.github/copilot-instructions.md` con una línea que remita a `AGENTS.md`. |
| **Gemini CLI** | `GEMINI.md` importa `AGENTS.md`. También puedes configurar `AGENTS.md` como archivo de contexto en sus ajustes. |
| **Kiro** | Copia `AGENTS.md` (o un resumen que remita a él) a `.kiro/steering/iso-flat.md`. |
| **Continue** | Crea una regla en `.continue/rules/` que remita a `AGENTS.md`. |
| **ChatGPT, Gemini web y otras IAs de chat** | Pega `prompts/prompt-universal.md` como instrucciones del GPT, Gem o Proyecto, o al principio del chat. Si admiten archivos, adjunta también `AGENTS.md` y el `SKILL.md` que toque. |
| **Cualquier otra con terminal** | Dile: «Lee AGENTS.md y síguelo». |

## Dependencias de las herramientas del kit

| Herramienta | Necesita |
|---|---|
| `iso.mjs` (generador) | Node 18+ |
| `iso-check.py` (escáner) | Python 3.9+ (solo biblioteca estándar) |
| `iso-render.py` (revisión) | Python + `pip install cairosvg` |
| Scripts de Blender | Blender 4.2 o posterior (probado en 4.2 y 5.2), o `pip install bpy` |
| Verificar animaciones en navegador (opcional) | `pip install playwright` + `playwright install chromium` |

## Conectar Blender

1. **Interactivo, por MCP.** Instala en tu cliente de IA un servidor MCP de Blender y, en Blender, su complemento. Con Blender abierto, inicia el servidor del complemento (suele escuchar en `localhost:9876`). La IA podrá ejecutar los scripts del kit en tu sesión y ver el resultado.
2. **En segundo plano.** Si tu servidor MCP tiene herramientas `…_for_cli`, o la IA tiene terminal, puede ejecutar `blender archivo.blend --background --python …` sin abrir Blender.
3. **Manual.** La IA te da el script y tú lo pegas en *Scripting › Run Script*.

En Windows, las rutas van entre `r"…"` en Python, por ejemplo `r"C:\Users\…\ISOMETRIC-P\skills\iso-flat-blender\scripts\iso_export_svg.py"`. Carga los scripts con `runpy.run_path(ruta, run_name="__main__")` (ver `skills/iso-flat-blender/SKILL.md`).

### Problemas frecuentes

| Mensaje | Causa y solución |
|---|---|
| `Cannot connect to Blender at localhost:9876` | Blender no está abierto o el servidor del complemento MCP no está iniciado. Ábrelo e inícialo desde el panel del complemento. |
| `Blender executable not found at 'blender'` | Las herramientas en segundo plano (`…_for_cli`) no encuentran `blender.exe`. Define `BLENDER_PATH` con su ruta completa (p. ej. `C:\Program Files\Blender Foundation\Blender 5.2\blender.exe`) en la configuración del servidor MCP o en las variables de entorno de Windows, y reinicia la app de IA. |
| `No module named 'iso_comun'` | El script se cargó con `exec` y no conoce su carpeta. Usa `runpy.run_path`, o añade la carpeta `scripts` a `sys.path`. |
| La booleana no corta en el SVG | El cortador está desactivado en el visor. Ocúltalo con el ojo (`H`), no con «Disable in Viewports». |
