# Kit iso-flat: instrucciones para agentes de IA

Este repositorio es un kit para que cualquier IA diseñe, genere, anime y revise **ilustraciones flat isométricas para web y apps**. Usa SVG, CSS o Blender como motor. Si eres un agente (Claude, Codex, Cursor, Copilot, Gemini, Kiro, Continue u otro), este archivo es tu punto de entrada.

## Qué cargar según la tarea

Las skills están en `skills/<nombre>/SKILL.md`. Lee siempre `iso-flat-fundamentos` y después solo la skill que necesites. Las referencias de cada skill se leen cuando la propia skill lo indica, no antes.

| Tarea | Skill |
|---|---|
| Cualquier trabajo isométrico (siempre primero) | `skills/iso-flat-fundamentos/SKILL.md` |
| Generar o editar SVG/CSS para web, validar un SVG | `skills/iso-flat-svg-web/SKILL.md` |
| Animar (CSS, GSAP, Lottie, Rive) | `skills/iso-flat-animacion/SKILL.md` |
| Modelar, animar o renderizar en Blender; exportar de Blender a web | `skills/iso-flat-blender/SKILL.md` |
| Redactar prompts para modelos de imagen (Midjourney, GPT Image, Flux…) | `skills/iso-flat-prompts-imagen/SKILL.md` |

Flujo típico: fundamentos → (svg-web | blender | prompts-imagen) → animación → verificación.

## Reglas que no se negocian

1. **Proyección explícita.** Elige `iso` (aristas a 30°, cámara a 54.7356°) o `dimetric` 2:1 (26.565°, cámara a 60°), nómbrala y no las mezcles.
2. **Ejes comunes:** x hacia abajo-derecha, y hacia abajo-izquierda, z hacia arriba. En pantalla: `sx = (x − y)·0.8660254·u`, `sy = ((x + y)·0.5 − z)·u`. En Blender: x = +X, y = −Y, z = +Z.
3. **Una luz y tres tonos por material:** superior claro, izquierdo medio y derecho oscuro (luz arriba-izquierda por defecto). Calcúlalos con `shade()` en `iso.mjs` o `tres_tonos()` en `iso_comun.py`, que son idénticos. No los inventes a ojo.
4. **No calcules puntos a mano.** Describe la escena como datos (JSON para `iso.mjs`) o modélala en Blender y exporta.
5. **Preparado para animar:** un `<g id data-iso-part>` por pieza, sin atributo `transform` en ese grupo. Se anima solo `transform` y `opacity`, dentro de `prefers-reduced-motion`.
6. **Sistema de diseño primero.** Si el proyecto tiene tokens (por ejemplo SYX), los colores salen de ellos. Valida con `iso-check.py --tokens` y declara lo consultado con `/* syx-reuse: … */`.
7. **Nada se entrega sin verificar.** Pasa el escáner y mira el render tú mismo (ver abajo).

## Herramientas

| Herramienta | Comando |
|---|---|
| Generar SVG desde JSON | `node skills/iso-flat-svg-web/scripts/iso.mjs build escena.json > escena.svg` |
| Escanear reglas | `python3 skills/iso-flat-svg-web/scripts/iso-check.py [--tokens] escena.svg` |
| Renderizar para revisar (resuelve tokens) | `python3 skills/iso-flat-svg-web/scripts/iso-render.py escena.svg revision.png [--fijar 'selector=transform']` |
| Preparar escena en Blender | `blender escena.blend --background --python skills/iso-flat-blender/scripts/iso_setup.py -- --mode iso` |
| Blender → SVG vectorial por piezas | `… --python skills/iso-flat-blender/scripts/iso_export_svg.py -- --salida escena.svg --sombras [--animacion] [--tokens prefijo]` (traslaciones → `@keyframes`, rotaciones → flipbook) |
| Blender → capas PNG + manifiesto | `… --python skills/iso-flat-blender/scripts/iso_export_capas.py -- --carpeta salida/ [--sombras]` (render rico: prepara antes con `iso_setup.py -- --materiales conservar`) |
| Scripts de Blender sin Blender instalado | `python3 skills/iso-flat-blender/scripts/con_bpy.py escena.blend <script>.py -- <opciones>` (`pip install bpy`) |

Dependencias:

- `node` (generador), sin paquetes.
- `python3` (escáner).
- `pip install cairosvg` (renderizador).
- Blender 4.2+ o `pip install bpy` (scripts de Blender).

Si no tienes terminal, entrega el código o el comando al usuario para que lo ejecute, y pídele el resultado o una captura.

### Conexión con Blender

Por orden de preferencia:

1. **Servidor MCP de Blender**, si tu cliente lo tiene: ejecuta Python en la sesión abierta, o en segundo plano con las herramientas `…_for_cli`. Carga los scripts con `runpy.run_path(ruta, run_name="__main__")`. Si da «Cannot connect to Blender at localhost:9876», pide al usuario que abra Blender e inicie el servidor del complemento. Si da «Blender executable not found», falta `BLENDER_PATH` (ver `INSTALAR.md`). Antes de tocar la sesión abierta, comprueba `bpy.data.is_dirty`.
2. **Línea de comandos** (`blender --background --python …`).
3. **Manual:** el usuario pega el script en *Scripting*.

Detalle en `skills/iso-flat-blender/SKILL.md`.

## Verificación obligatoria antes de entregar

1. `iso-check.py` da 0 errores.
2. Has renderizado a PNG y **has mirado la imagen**: aristas a 30°, sin costuras, orden de profundidad correcto, tres tonos coherentes y la imagen se lee sin animación.
3. Si hay animación, has comprobado al menos un instante intermedio, con `--fijar` o pausando las animaciones en un navegador.
4. Control de calidad de `iso-flat-fundamentos` (sección 5).

## Mapa del repositorio

```
AGENTS.md                ← este archivo (entrada para agentes)
README.md                ← guía para personas
INSTALAR.md              ← cómo conectar el kit a cada herramienta de IA
agentes/iso-ilustrador.md← rol de ilustrador listo para usar como subagente o instrucciones
prompts/prompt-universal.md ← versión autocontenida para IAs de chat sin archivos
skills/                  ← las cinco skills (formato SKILL.md estándar)
ejemplos/                ← escenas de ejemplo (JSON, Blender) y sus salidas verificadas
```
