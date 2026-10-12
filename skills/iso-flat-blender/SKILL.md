---
name: iso-flat-blender
description: "Usa Blender como motor de ilustración flat isométrica para web y apps: cámara isométrica exacta, shader de tres tonos idéntico al de la web, y exportación a SVG vectorial por piezas (traslaciones de Blender convertidas a CSS y rotaciones en flipbook, con sombras exactas) o a capas PNG con sombras propias y manifiesto para animar renders ricos. Incluye cómo conectar cualquier IA a Blender (MCP, línea de comandos o consola). Úsala cuando la escena se modele, se anime o se renderice en 3D. Para escenas dibujadas directamente en código usa iso-flat-svg-web."
metadata:
  author: Jose Luis Pizarro Feo
  version: '2.1'
  family: iso-flat
---

# Blender como motor isométrico para web

Blender aporta lo que en 2D cuesta mucho: biseles, booleanas, arrays, curvas, mecanismos, escenas de cientos de piezas y animación con curvas reales. Esta skill convierte ese trabajo en piezas web animables, sin perder la proyección exacta ni los colores de la paleta. Las reglas de estilo están en `iso-flat-fundamentos`.

## Elige la salida

| Quieres | Script | Resultado |
|---|---|---|
| Vector nítido, ligero y tematizable | `iso_export_svg.py` | SVG con un `<g data-iso-part>` por objeto, tres tonos, sombras exactas sobre su receptor, traslaciones en `@keyframes` y rotaciones en flipbook |
| La riqueza de un render (materiales, luz real, texturas) y aun así animar por piezas | `iso_export_capas.py` | una capa PNG por pieza y otra por su sombra, `escena.svg` compositor y `manifiesto.json` con posiciones y vectores de eje |
| Una imagen fija o un vídeo | `iso_setup.py` + render normal | PNG, WebP o secuencia (ver «Salidas clásicas») |

Los tres scripts comparten `iso_comun.py`: misma proyección y misma fórmula de tres tonos que `iso.mjs`. Un azul `#5b8def` da los mismos tres hex en el render de Blender, en el SVG exportado y en la web.

## Flujo recomendado

1. **Modela con intención web.**
   - Un objeto por pieza que se vaya a mover (o una colección por pieza compuesta, con `--por coleccion` en capas).
   - Nombres claros: el nombre del objeto pasa a ser su `id` y su `data-iso-part`.
   - Los cortadores de booleanas se ocultan con el ojo (`H`), nunca con «Disable in Viewports». Desactivados no se evalúan y el corte desaparece.
   - Color base: el color de vista del material (Viewport Display › Color) o una propiedad personalizada `iso_color = "#5b8def"` en el objeto o el material.
2. **Prepara la escena:** `iso_setup.py` pone la cámara isométrica, el shader de tres tonos, View Transform Standard y el fondo transparente. Para un render con luz real, `--materiales conservar` solo pone la cámara y el encuadre (ver `ejemplos/blender/render_rico.py`).
3. **Anima, si hace falta,** con keyframes. Usa interpolación lineal en los bucles (`Preferences › Animation › Default Interpolation`, o el script `animar_escena.py` de ejemplo).
   - **Traslaciones:** en proyección paralela, una traslación 3D es una traslación 2D exacta. Se exportan como `@keyframes` de `transform`, sin pérdida.
   - **Rotaciones y escalas** (aspas, engranajes, puertas): un dibujo fijo no puede girar en 3D, así que se exportan en **flipbook**, N fotogramas vectoriales de la pieza que se alternan por opacidad. Para un giro continuo, haz que el ciclo cierre (por ejemplo 90° con cuatro aspas) y elige N según la suavidad que necesites.
4. **Exporta** con `iso_export_svg.py` o `iso_export_capas.py`.
5. **Verifica:**
   - `python3 ../iso-flat-svg-web/scripts/iso-check.py salida.svg` (añade `--tokens` en proyectos con sistema de diseño);
   - renderiza con `iso-render.py` y mira la imagen;
   - si hay animación, comprueba un fotograma intermedio.

## Cómo conectar una IA a Blender

Tres vías, de más a menos integrada. El kit funciona igual en las tres.

### A. Servidor MCP de Blender (Claude, Cursor, VS Code, Codex, Gemini CLI…)

El cliente de IA habla con Blender mediante un servidor MCP y un complemento dentro de Blender.

- **Interactivo:** abre Blender, activa el complemento MCP e inicia su servidor (por defecto escucha en `localhost:9876`). La IA ejecuta Python en tu sesión abierta y ves los cambios en vivo. Si la IA recibe «Cannot connect to Blender at localhost:9876», es que Blender no está abierto o el servidor del complemento no está iniciado.
- **Sin interfaz:** algunos servidores ofrecen herramientas `…_for_cli` que abren un `.blend` con `blender --background`. No necesitan Blender abierto y son ideales para exportar en lote.
- La IA carga un script con `runpy`, que le da su ruta real. Así encuentra `iso_comun.py` en su carpeta. Las rutas de Windows van con `r"…"`:

```python
import sys, runpy
sys.argv = ["blender", "--", "--salida", r"C:\ruta\web\escena.svg", "--sombras", "--animacion"]
runpy.run_path(r"C:\ruta\ISOMETRIC-P\skills\iso-flat-blender\scripts\iso_export_svg.py", run_name="__main__")
```

- Si el servidor MCP responde «Blender executable not found», el servidor no encuentra `blender.exe`. Define la variable de entorno `BLENDER_PATH` con su ruta completa (por ejemplo `C:\Program Files\Blender Foundation\Blender 5.2\blender.exe`) en la configuración del servidor MCP o en las variables de entorno de Windows, y reinicia la app de IA.

Antes de ejecutar código en una sesión abierta, la IA debe comprobar si hay cambios sin guardar (`bpy.data.is_dirty`) y no cerrar ni reemplazar el archivo del usuario sin preguntar.

### B. Línea de comandos (cualquier IA con terminal: Codex, Claude Code, Gemini CLI, Kiro…)

```bash
blender escena.blend --background --python scripts/iso_setup.py -- --mode iso --res 2048
blender escena.blend --background --python scripts/iso_export_svg.py -- --salida web/escena.svg --sombras --animacion
blender escena.blend --background --python scripts/iso_export_capas.py -- --carpeta web/capas-escena
```

Sin Blender instalado, el módulo `bpy` de PyPI (`pip install bpy`, Python 3.13 para Blender 5.x) ejecuta los mismos scripts desde Python. Así se probaron.

### C. Manual (IAs de chat sin herramientas: ChatGPT, Gemini web…)

La IA te da el comando o el bloque de Python. Lo pegas en *Scripting › Text Editor › Run Script* (o en la consola de Python de Blender) y le devuelves el resultado o una captura.

## `iso_setup.py`: preparar la escena

```bash
blender escena.blend --python scripts/iso_setup.py -- --mode iso --light left --res 2048 --outline 0 --engine eevee
```

| Parámetro | Valores | Efecto |
|---|---|---|
| `--mode` | `iso` / `dimetric` | rotación X de cámara a 54.7356° o 60°, Z = 45°, ortográfica |
| `--light` | `left` / `right` | qué lateral es el tono medio |
| `--outline` | 0 o grosor en px | Freestyle con línea oscura azulada |
| `--engine` | `eevee` / `cycles` | Cycles en CPU sirve para servidores sin GPU |
| `--materiales` | `flat` / `conservar` | `conservar`: solo cámara, encuadre y fondo transparente; no toca materiales, luces ni View Transform |

Con `flat` (por defecto) además encuadra la escena, pone View Transform = Standard y el fondo transparente, y sustituye los materiales por emisión de tres tonos según la normal. Cada material nuevo guarda su `iso_color`, así el exportador sigue sabiendo el color base. Se puede ejecutar dos veces sin degradar los colores.

## `iso_export_svg.py`: vector por piezas

```bash
blender escena.blend --background --python scripts/iso_export_svg.py -- --salida escena.svg [opciones]
```

| Opción | Efecto |
|---|---|
| `--modo iso\|dimetric`, `--unidad 40`, `--luz left\|right` | proyección, px por unidad de Blender en cada eje, lado iluminado |
| `--sombreado bandas\|suave` | `bandas`: tres tonos exactos (flat puro). `suave`: mezcla por normal; los biseles y cilindros ganan volumen |
| `--tokens PREFIJO` | colores como `style="fill:var(--PREFIJO-<material>-top)"` con la paleta en `<style>`. Para SYX o cualquier sistema de tokens; fuerza `bandas` |
| `--sombras` | sombras exactas: cada cara orientada a la luz se proyecta sobre el **receptor** (la pieza más alta que hay debajo: la losa, o el techo de la torre para lo que está encima). Funciona con formas cóncavas (aspas, huecos, letras). Se dibujan tras su receptor, recortadas a sus caras superiores, y su dirección sigue a `--luz` |
| `--animacion`, `--paso 2` | traslaciones → `@keyframes` de `transform`. Las sombras se mueven con su pieza y se corren hacia la luz si la pieza sube. Todo va dentro de `prefers-reduced-motion` |
| `--rotaciones flipbook\|ignorar`, `--fotogramas 12` | piezas que rotan o escalan: N fotogramas vectoriales alternados por opacidad, con sus sombras sincronizadas. El estado estático (sin animación o con movimiento reducido) muestra el primero. `ignorar` exporta solo la traslación |
| `--coleccion NOMBRE`, `--margen`, `--titulo`, `--desc`, `--fondo` | alcance y presentación |

Qué hace por dentro:

- usa la geometría *evaluada* (modificadores aplicados) en el fotograma inicial si hay animación, y descarta las caras de espaldas;
- clasifica cada cara como techo si su normal sube más de 30° (`nz > 0.5`), igual que el shader de `iso_setup.py`; así render y SVG coinciden y los biseles no dejan muescas;
- ordena las piezas con un grafo «está detrás de» entre cajas (con respaldo por centro si hay ciclos) y las caras de cada pieza por profundidad;
- emite `stroke` del mismo color en cada cara para evitar costuras.

También escribe `escena.json` con el orden, las cajas en coordenadas de mundo, los vectores de eje en px, la duración y `cruces`: los pares de piezas que intercambian su orden de profundidad durante la animación y desde cuándo, para reordenarlas con JS en ese instante.

**Límites:**

- Las piezas que se interpenetran pueden ordenarse mal. Divídelas en dos objetos.
- Si una animación hace que una pieza pase por delante de otra, el orden del DOM no cambia solo. El exportador lo **detecta**: avisa en la consola y lo anota en `cruces` del manifiesto. Divide el recorrido o reordena con JS en ese instante.
- El flipbook multiplica el peso de la pieza por N. Úsalo en piezas pequeñas (aspas, engranajes) y ajusta `--fotogramas`.
- Las mallas muy densas generan SVG pesados. Usa un modificador Decimate (Planar) o modela en low-poly.

## `iso_export_capas.py`: render rico por capas

```bash
blender escena.blend --background --python scripts/iso_export_capas.py -- --carpeta salida [--por objeto|coleccion] [--oclusion ninguna|corte] [--sombras]
```

- Renderiza cada pieza sola, recortada a su caja en pantalla (`border` + `crop`), con fondo transparente. Respeta el motor, las muestras, los materiales y las luces de la escena.
- Con `--oclusion ninguna` (por defecto) cada capa es la pieza completa: lo mejor para animar, porque el orden lo da el DOM. Con `corte`, las demás piezas actúan como holdout y recortan lo que tapan: composición estática exacta, pero el recorte queda horneado.
- Genera `escena.svg` (marcado `data-iso-raster="capas"`, que el escáner acepta), `manifiesto.json` y `referencia.png`.
- `ejes_px` del manifiesto dice cuánto se desplaza una capa por cada unidad de Blender en x, y, z del mundo isométrico. Por ejemplo, para mover una pieza 2 unidades en x: `translate(2·ejes_px.x)`.
- Cada pieza se renderiza sola, así que su capa no lleva sombras ajenas. Con `--sombras`, cada pieza recibe además su capa de sombra (`<id>-sombra.png`), colocada tras su receptor y marcada `data-iso-shadow-of`. Al mover la pieza, su sombra se mueve con ella y no deja huecos. Cómo se obtiene:
  - **por cociente:** se hacen dos renders limitados a la zona donde puede caer la sombra, uno sin la pieza y otro con la pieza invisible para cámara y rebotes pero proyectando. En los dos solo existen la pieza y lo que tiene debajo;
  - el alfa es `1 − luminancia(con)/luminancia(sin)`, y el color, un tinte único medido del render (la sombra conserva el matiz del ambiente);
  - funciona con EEVEE y con Cycles (`--muestras-sombra` ajusta Cycles). Con luces de tipo sol, el render se limita a la zona de la sombra;
  - necesita luces que proyecten sombra: el sombreado plano de `iso_setup.py` no las tiene. Para eso está `--materiales conservar` con luces reales.
- Límite inherente a las capas: la luz que rebota *entre* piezas (por ejemplo, el azul de una torre sobre el suelo) no está en ninguna capa. La composición difiere del render de referencia en torno a un 1 % de píxeles, sobre todo en bordes y contactos.

## Salidas clásicas

| Destino | Salida |
|---|---|
| Imagen web | PNG transparente a 2x, luego WebP o AVIF |
| Vídeo ligero | secuencia PNG convertida a WebM/MP4 con alfa, o sprite sheet |
| Lottie | Blender no exporta Lottie: exporta el SVG por piezas y reconstruye en After Effects, o anima en código (`iso-flat-animacion`) |

## Reglas manuales (si no usas los scripts)

- Cámara ortográfica con rotación (54.736°, 0°, 45°) para isométrica real o (60°, 0°, 45°) para 2:1. La posición da igual; se encuadra con `ortho_scale`.
- View Transform en Standard. AgX o Filmic desplazan los colores.
- Look flat:
  1. emisión con tono por normal (el script);
  2. Toon BSDF o Shader to RGB + ColorRamp en Constant con un sol fijo;
  3. Workbench Flat, que solo sirve para siluetas.
- Contornos: Freestyle (1 px) o el modificador Line Art de Grease Pencil.
- Girar Z en saltos de 90° mantiene la isometría. Reasigna qué lateral es «izquierdo» para conservar la luz.

## Control final

1. El cubo de referencia tiene las aristas de suelo a 30° (iso) o 26.565° (2:1). En capas, `ejes_px.x` debe tener pendiente tan 30° ≈ 0.5774.
2. Los hex del render coinciden con los del SVG y con la paleta.
3. `iso-check.py` sin errores y control de calidad de `iso-flat-fundamentos`.
