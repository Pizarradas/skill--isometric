# Lottie y Rive para escenas isométricas

Lee este archivo cuando el entregable sea un JSON de Lottie o un archivo .riv.

## Lottie (After Effects y Bodymovin)

Flujo:

1. Agrupa bien la ilustración: un grupo por objeto y caras separadas.
2. Exporta a SVG, ábrelo en Illustrator y guárdalo como .ai. El SVG da una conversión más limpia que el PDF o el EPS.
3. Importa el .ai en After Effects y aplica Layer › Create shapes from vector layer. Después borra el original.
4. Anima y exporta con Bodymovin a tamaño 1x.
5. Valida arrastrando el JSON a LottieFiles. Si se ve bien allí, usa funciones soportadas.

Lo que no exporta, o funciona mal: expresiones, efectos, modos de fusión (multiply, screen, add), mates de luminancia, estilos de capa (sombra paralela, trazo) y, en parte, capas 3D. Para isométrico esto significa:

- Las sombras se hacen con formas planas con opacidad, nunca con el estilo "drop shadow".
- El sombreado de caras va en colores planos por cara, sin modos de fusión.
- No uses capas 3D de After Effects con cámara para fingir la isometría. Anima en 2D con los vectores de eje proyectados.

Para que el archivo pese poco:

- Usa parenting para movimientos compartidos.
- Pon pocos keyframes de trazado y quita vértices sobrantes.
- No uses wiggle ni autotrace, que generan un keyframe por fotograma.
- Las nulls deben quedar visibles con opacidad 0 %, o no se exportan.
- Usa mates pequeños.

## Rive (interacción por estados)

- Úsalo cuando la escena reaccione a hover, clic o datos (por ejemplo, encender o apagar servidores en un diagrama).
- Monta una state machine por artboard con estados como `idle`, `hover` y `active`. Contrólala por data binding o por inputs.
- Las state machines se "asientan" cuando no hay cambios y dejan de calcular, lo que ahorra CPU y batería.

```js
const r = new rive.Rive({
  src: "/escena-iso.riv",
  canvas: document.getElementById("iso"),
  autoplay: true,
  stateMachine: "Main",
  onLoad: () => r.resizeDrawingSurfaceToCanvas(),
});
```

- Respeta el movimiento reducido. Si `matchMedia('(prefers-reduced-motion: reduce)').matches`, no reproduzcas automáticamente y muestra el estado final.

## Cuándo elegir cada uno

- Si ya tienes el SVG y es para web: GSAP.
- Si hay que entregar a apps nativas o a un equipo de After Effects: Lottie.
- Si es interactiva con estados: Rive.
- Si es un bucle de fondo sencillo: CSS.
