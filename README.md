# ISOMETRIC-P · kit iso-flat

Conocimiento y herramientas para que **cualquier IA** diseñe, genere, anime y revise ilustraciones **flat isométricas para web y apps**, usando código (SVG/CSS) o **Blender** como motor.

- **Para agentes de IA:** el punto de entrada es [`AGENTS.md`](AGENTS.md).
- **Para conectar el kit a tu herramienta** (Claude, Codex, Cursor, Copilot, Gemini, Kiro, Continue, ChatGPT…): [`INSTALAR.md`](INSTALAR.md).

## Tres caminos, un mismo resultado

| Camino | Cuándo | Cómo |
|---|---|---|
| **Código** | prismas, cilindros, iconos, diagramas | describes la escena en JSON → `iso.mjs build` → SVG por piezas |
| **Blender, vector** | biseles, booleanas, curvas, mecanismos, animación con curvas reales | modelas y animas en Blender → `iso_export_svg.py` → SVG por piezas: traslaciones en `@keyframes`, rotaciones (aspas, engranajes) en flipbook, sombras exactas |
| **Blender, render rico** | materiales, luz real, texturas | `iso_export_capas.py --sombras` → una capa PNG por pieza y otra por su sombra, + manifiesto para animar |

Los tres comparten la misma proyección y la misma fórmula de tres tonos. Un color de la paleta da los mismos hex en Blender, en el SVG y en la web.

Todas las salidas son animables: un `<g data-iso-part>` por pieza, colores por tokens si el proyecto los tiene (por ejemplo SYX) y movimiento solo con `transform` y `opacity`, respetando `prefers-reduced-motion`.

## Estructura

```
AGENTS.md                    entrada para agentes: qué cargar, reglas, herramientas, verificación
CLAUDE.md · GEMINI.md        punteros a AGENTS.md para Claude Code y Gemini CLI
INSTALAR.md                  cómo conectar el kit a cada IA y a Blender
agentes/iso-ilustrador.md    rol de ilustrador (subagente de Claude Code o instrucciones para otras IAs)
prompts/prompt-universal.md  versión autocontenida para IAs de chat sin archivos
skills/
  iso-flat-fundamentos/      proyección, rejilla, tres tonos, sombras, profundidad, control de calidad, fuentes
  iso-flat-svg-web/          generador iso.mjs, escáner iso-check.py, renderizador iso-render.py, CSS 3D, SVGO
  iso-flat-animacion/        GSAP, CSS, Lottie, Rive; animar en espacio de mundo
  iso-flat-blender/          iso_setup.py, iso_export_svg.py, iso_export_capas.py, iso_comun.py, con_bpy.py
  iso-flat-prompts-imagen/   prompts para Midjourney, GPT Image, Flux…
ejemplos/
  escena-servidor.json/.svg  escena descrita como datos y su SVG con tokens
  blender/                   escena.blend (flat) y escena_rica.blend (luz real), los scripts que las crean,
                             animan y preparan, y salida/ verificada (vector, capas y capas con sombras)
```

## Prueba rápida

```bash
# Código → SVG
node skills/iso-flat-svg-web/scripts/iso.mjs build ejemplos/escena-servidor.json > escena.svg
python3 skills/iso-flat-svg-web/scripts/iso-check.py --tokens escena.svg
python3 skills/iso-flat-svg-web/scripts/iso-render.py escena.svg escena.png      # requiere: pip install cairosvg

# Blender → SVG animado (con Blender instalado)
cd ejemplos/blender
blender escena.blend --background --python ../../skills/iso-flat-blender/scripts/iso_export_svg.py -- --salida mi-escena.svg --sombras --animacion
# …o sin Blender: pip install bpy  y  python3 ../../skills/iso-flat-blender/scripts/con_bpy.py escena.blend iso_export_svg.py -- --salida mi-escena.svg --sombras --animacion
```

## Qué está verificado

- **Proyección.**
  - Cámara a 54.7356° y aristas a 30°. Los vectores de eje que mide el exportador de capas tienen pendiente tan 30°.
  - El valor SSR correcto es 86.602 %, no el 86.062 % que circula en tutoriales.
  - `rotateX(60deg)` es dimétrica 2:1, no isométrica.
  - Detalle en `skills/iso-flat-fundamentos/references/fuentes.md`.
- **Color.**
  - `shade()` (JavaScript) y `tres_tonos()` (Python) dan hex idénticos (16 casos comprobados).
  - El render de Blender tras `iso_setup.py` reproduce esos hex exactos.
- **Scripts de Blender.** Probados en Blender 4.2 (original) y 5.2 (todos): biseles, booleanas, cilindros y animación.
  - Traslaciones convertidas a CSS y rotaciones en flipbook. Barrido de un ciclo completo cada 1,3 ms: siempre hay exactamente un fotograma visible y su sombra va sincronizada.
  - Sombras exactas en formas cóncavas, sobre su receptor, incluida la de una pieza que descansa sobre otra.
  - Aviso de cruces de profundidad durante la animación.
  - Capas con sombras propias: la composición difiere del render en menos del 1 % de píxeles, y mover piezas con su sombra no deja huecos.
- **Animación en navegador.** Comprobada en Chromium: pausada en varios instantes y capturada, antes y después de optimizar con SVGO. La configuración de SVGO del kit desactiva los plugins que destruían la animación (`inlineStyles`, `moveGroupAttrsToElems`, `convertTransform`…).

## Créditos

Autor: José Luis Pizarro Feo. Fuentes consultadas y contradicciones resueltas: `skills/iso-flat-fundamentos/references/fuentes.md`.
