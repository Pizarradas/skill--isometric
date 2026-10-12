---
name: iso-flat-svg-web
description: "Genera ilustraciones flat isométricas como SVG o CSS 3D listas para web: heros, infografías, diagramas, iconos y escenas por capas, con código limpio, accesible, tematizable con tokens y preparado para animar. Incluye generador desde datos (iso.mjs), escáner de reglas (iso-check.py) y renderizador de revisión (iso-render.py). Úsala cuando haya que producir, editar o validar código SVG/HTML/CSS isométrico. Para imágenes ráster con IA usa iso-flat-prompts-imagen; para escenas modeladas en 3D, iso-flat-blender."
metadata:
  author: Jose Luis Pizarro Feo
  version: '2.0'
  family: iso-flat
---

# Flat isométrico en SVG y CSS para web

Sigue las reglas de `iso-flat-fundamentos`: especificación, tres tonos y control de calidad.

## Elige la técnica

| Necesidad | Técnica |
|---|---|
| Ilustración, icono, hero o infografía con prismas y cilindros | Escena como datos JSON → `node scripts/iso.mjs build escena.json` (opción por defecto) |
| Formas complejas (biseles, booleanas, curvas, cientos de piezas) o animación con curvas reales | Modelar en Blender y exportar con `iso-flat-blender` |
| Texto, UI o imágenes planas pegadas a una cara | `<g transform="matrix(...)">` por plano (ver `iso-flat-fundamentos/references/matematicas.md`) |
| Tarjetas o rejillas HTML reales en isométrico, con hover | CSS 3D: lee `references/css-3d.md` |
| Escenas tipo juego, interactivas y con miles de piezas | Canvas/WebGL con cámara ortográfica (fuera de esta skill) |

## Flujo de trabajo

1. **Especificación.** Fija proyección, unidad, luz y paleta. En un proyecto con sistema de diseño (por ejemplo SYX), la paleta sale de sus tokens. Antes de crear tokens nuevos, consulta los existentes.
2. **Escena como datos.** No escribas puntos de pantalla a mano. Describe la escena en JSON (`ejemplos/escena-servidor.json` en la raíz del kit):

```json
{
  "title": "…", "desc": "…", "mode": "iso", "unit": 40, "prefix": "iso",
  "palette": { "azul": "#5b8def", "losa": "#c9d6e8" },
  "bg": "#f6f4ef",
  "base": { "kind": "prism", "id": "losa", "x": -3, "y": -3, "z": 0, "w": 6, "d": 6, "h": 0.4, "token": "losa" },
  "objects": [
    { "kind": "prism", "id": "rack", "x": -2, "y": -2, "z": 0.4, "w": 1.5, "d": 1.5, "h": 3, "token": "azul" },
    { "kind": "cylinder", "id": "tanque", "cx": 1.2, "cy": 1.4, "z": 0.4, "r": 0.8, "h": 1.6, "color": "#f2a541" }
  ]
}
```

   - `light: "right"` intercambia los laterales y refleja las sombras. `shadows: false` las quita. Con `base`, las sombras (de prismas y de cilindros) caen sobre la losa y se recortan a su cara superior.
   - Con `token`, la cara usa `style="fill:var(--<prefix>-<token>-top|left|right)"` y `palette` genera esos tokens con los tres tonos.
   - Con `color`, se escribe el hex directo.
   - `prefix` es el prefijo del proyecto. En SYX es el nombre del proyecto, nunca `app-` ni prefijos de SYX.

3. **Genera:** `node scripts/iso.mjs build escena.json > escena.svg`. También puedes importar el módulo y usar `prism()`, `cylinder()`, `flatShadow()`, `cylinderShadow()`, `shadowDir()`, `depthSort()`, `paletteCSS()`, `scene()` o `build()`. Para ver la demo: `node scripts/iso.mjs demo iso > demo.svg` (resultado esperado en `assets/demo-iso.png`).
4. **Valida con el escáner:** `python3 scripts/iso-check.py [--tokens] escena.svg [estilos.css]`. Corrige hasta tener 0 errores. Con `--tokens`, los colores literales pasan de aviso a error.
5. **Mírala:** `python3 scripts/iso-render.py escena.svg revision.png [--css tokens.css]` resuelve los tokens y renderiza. Abre la imagen y pasa el control de calidad de los fundamentos. Corrige y repite (dos pasadas suelen bastar).
6. **Optimiza y haz accesible el SVG:** lee `references/accesibilidad-rendimiento.md` antes de entregar.

El resultado se da por terminado cuando el escáner da 0 errores, el PNG se ve bien, el SVG tiene `role`/`title` y conserva sus `id` y `data-iso-part` después de optimizar.

## Estructura preparada para animar

**La geometría vive en atributos y el movimiento en CSS, en elementos distintos.** Si animas `transform` por CSS sobre un elemento con atributo `transform`, lo sustituyes y la pieza se rompe. Cada pieza animable se monta en tres capas:

```svg
<g id="ventilador" data-iso-part="ventilador">                   <!-- 1. Movimiento en el mundo (CSS/GSAP): sin atributo transform -->
  <g transform="matrix(0.8660254 0.5 -0.8660254 0.5 120 42)">   <!-- 2. Proyección del plano (atributo fijo) -->
    <g class="ventilador__aspas">                               <!-- 3. Movimiento en plano (CSS): rotate/scale en coordenadas planas -->
      …
    </g>
  </g>
</g>
```

- **Capa 3:** una rotación aplicada *dentro* del grupo proyectado se ve como rotación isométrica real (un ventilador que gira sobre el techo, una puerta que se abre sobre su pared). Usa `transform-box: fill-box; transform-origin: center` (o el pivote) en esa capa.
- **Capa 1:** desplazamientos a lo largo de los ejes del mundo con propiedades registradas, para que sean animables:

```css
@property --iso-u { syntax: "<number>"; inherits: false; initial-value: 0; }
@property --iso-v { syntax: "<number>"; inherits: false; initial-value: 0; }
@property --iso-w { syntax: "<number>"; inherits: false; initial-value: 0; }
[data-iso-part] {
  transform: translate(calc((var(--iso-u) - var(--iso-v)) * 0.8660254px * 40),
                       calc(((var(--iso-u) + var(--iso-v)) * 0.5 - var(--iso-w)) * 1px * 40));
}
/* 40 = unidad de rejilla en px. Ahora una animación dice "--iso-w: 0.5" (sube media unidad). */
```

- `iso.mjs` y los exportadores de Blender ya emiten `<g id data-iso-part>` sin atributo `transform`, listos para la capa 1.
- Anima solo `transform` y `opacity`, siempre dentro de `@media (prefers-reduced-motion: no-preference)`. La ilustración debe leerse bien sin animación.
- Referencia completa: `assets/ejemplo-animable.svg` (tokens, tres capas, `@property`, reduced motion y rotación en plano).
- Recetas de movimiento: `iso-flat-animacion`.

## Reglas de código

- `viewBox` sin `width`/`height` fijos: el tamaño lo decide el contenedor. Centrado en el origen del mundo cuando se genera desde datos.
- Coordenadas con 2 decimales como máximo.
- Prefiere `polygon`, `ellipse` y `path` cortos. Evita filtros (`feGaussianBlur`, `drop-shadow`): pesan al pintar y Lottie no los admite.
- El orden del documento es el orden de profundidad: suelo, sombras, objetos de atrás hacia delante.
- Color por token: usa `style="fill:var(--…)"` o clases. No pongas `var()` en el atributo `fill`, porque no es fiable en todos los navegadores y herramientas. Los valores literales solo aparecen como fallback en la declaración del token.
- Curvas sobre una cara (puertas en arco, logos): dibújalas en plano y envuélvelas en la matriz de esa cara.
- Personajes o formas orgánicas: la base y la sombra siguen la rejilla y la silueta puede ser libre.

## Qué comprueba `iso-check.py`

| Regla | Nivel |
|---|---|
| raster embebido (salvo composición declarada `data-iso-raster="capas"`) | error |
| `matrix()` que no es un plano isométrico o dimétrico conocido; mezcla de proyecciones | error |
| `data-iso-part` con atributo `transform`, o duplicado | error |
| `@keyframes` que animan algo distinto de `transform`, `opacity` o `--iso-*`; animaciones sin `prefers-reduced-motion`; `--iso-u/v/w` sin `@property` | error |
| falta `<title>` (o `aria-hidden="true"` si es decorativa), falta `viewBox` | error |
| colores literales | aviso (error con `--tokens`) |
| tamaño fijo, más de 2 decimales, filtros, piezas sin `data-iso-part`, tokens nuevos sin `/* syx-reuse: … */` | aviso |

## Ejemplo

Entrada: «Hero isométrico de un servidor con dos bloques y un tanque, en azul y naranja».
Salida esperada: un `<svg role="img">` con `<title>`/`<desc>`, una losa base, sombras planas al 12 % y grupos `#servidor`, `#bloque-1` y `#tanque` con `data-iso-part` y tres caras sombreadas. Debe pasar `iso-check.py` con 0 errores y pesar menos de 10 KB tras SVGO.
