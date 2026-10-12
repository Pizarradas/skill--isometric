# Accesibilidad, optimización y entrega

Lee este archivo antes de entregar cualquier SVG o escena CSS isométrica.

## Accesibilidad

- Una ilustración informativa lleva `role="img"`, `aria-labelledby` que apunte a `<title>` y, si es compleja, también a `<desc>`.
- Una ilustración decorativa lleva `aria-hidden="true"` y ningún título.
- Si un diagrama transmite datos, pon el mismo contenido también en texto HTML cerca de la ilustración.
- Con `<use>` que apunta a `<defs>`, pon un `<desc>` en cada pieza definida y un `<title>` en el `<use>`.
- Los gráficos que transmiten información deben contrastar al menos 3:1 con su fondo.

## SVGO sin romper la animación

```js
// svgo.config.mjs (SVGO 4; en SVGO 3 añade también removeViewBox: false dentro de overrides)
export default {
  multipass: true,
  plugins: [
    { name: 'preset-default', params: { overrides: {
      cleanupIds: false,          // conserva los id que usan GSAP/CSS
      convertShapeToPath: false,  // mantiene polygon/ellipse legibles
      mergePaths: false,          // no fusiona las caras: cada una se anima por separado
      collapseGroups: false,      // conserva los <g> semánticos
      moveGroupAttrsToElems: false, // no baja la matriz del plano a los hijos (rompería la rotación en plano)
      convertTransform: false,    // mantiene las matrices de plano exactas (iso-check las reconoce)
      removeUnknownsAndDefaults: { keepDataAttrs: true }, // conserva data-iso-part
      inlineStyles: false,        // no convierte clases en style="": las clases son los anclajes de la animación
    } } },
    'removeDimensions',           // deja que el tamaño lo decida el CSS
  ],
};
```

Ejecuta `svgo in.svg -o out.svg --config svgo.config.mjs`. Después pasa `iso-check.py` sobre el resultado y compara visualmente antes y después (`iso-render.py`).

## Rendimiento

- Una ilustración estática va mejor como `<img src="x.svg">` (se cachea). Si se va a animar o tematizar, inclúyela inline en el HTML.
- Anima solo `transform` y `opacity`. Los filtros y las sombras con desenfoque cuestan más de pintar.
- Usa `will-change` solo si hay un problema medido, y quítalo después. Crear capas tiene su propio coste.
- Si hay más de unos cientos de polígonos animados a la vez, pasa a Canvas/WebGL o simplifica la escena.

## Entrega

- Un SVG fuente legible (con comentarios y datos de escena) y uno optimizado para producción.
- Un PNG de vista previa a 2x para revisión.
- La paleta en tokens: JSON o variables CSS con los tres tonos de cada color.
