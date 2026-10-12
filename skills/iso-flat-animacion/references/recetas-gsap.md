# Recetas GSAP y CSS para escenas isométricas

Lee este archivo cuando implementes la animación en GSAP o en CSS.

## Base: movimiento reducido con matchMedia

```js
const mm = gsap.matchMedia();
mm.add("(prefers-reduced-motion: no-preference)", () => { buildFull(); });
mm.add("(prefers-reduced-motion: reduce)", () => {
  gsap.from(".iso-obj", { opacity: 0, duration: 0.4, stagger: 0.05 });
});
```

## Vectores de eje (isométrica real)

```js
const U = 40, C = 0.8660254;
const iso = { x: [C*U, .5*U], y: [-C*U, .5*U], z: [0, -U] };
const along = (axis, n) => ({ x: `+=${iso[axis][0]*n}`, y: `+=${iso[axis][1]*n}` });
gsap.to("#carro", { ...along("x", 3), duration: 1.2, ease: "power2.inOut" });
```

## Construcción por capas (aparecer cayendo en z)

```js
gsap.from(".iso-obj", {          // los elementos ya están ordenados por profundidad en el DOM
  y: -60, opacity: 0, duration: .6,
  ease: "back.out(1.4)", stagger: .12
});
gsap.from(".iso-shadow", { opacity: 0, duration: .4, stagger: .12, delay: .2 });
```

## Crecimiento de un bloque (altura desde datos)

```js
import { prism } from "./iso.mjs";
const el = document.querySelector("#barra-1");
const s = { h: 0 };
gsap.to(s, { h: 3, duration: 1, ease: "power3.out",
  onUpdate: () => { el.outerHTML = prism({ id: "barra-1", x: 0, y: 0, z: 0, w: 1, d: 1, h: s.h, color: "#5b8def" }); }
});
// Para producción, actualiza solo el atributo points de cada cara en lugar de reemplazar outerHTML.
```

## Trazado de líneas (conexiones, flujos de datos)

```html
<path class="flow" pathLength="1" d="..." fill="none" stroke="#5b8def" stroke-width="3"/>
```
```css
.flow { stroke-dasharray: 1; stroke-dashoffset: 1; }
```
```js
gsap.to(".flow", { strokeDashoffset: 0, duration: 1.2, ease: "none" });
```

Con `pathLength="1"` no hace falta medir con `getTotalLength()`. Dibuja la ruta sobre las aristas o planos isométricos.

## Recorrido por un camino isométrico

```js
gsap.registerPlugin(MotionPathPlugin);
gsap.to("#paquete", { duration: 3, ease: "none", repeat: -1,
  motionPath: { path: "#ruta", align: "#ruta", alignOrigin: [0.5, 0.5], autoRotate: false } });
```

Pon `autoRotate: false`: un objeto isométrico no gira con la curva, mantiene sus caras. `align` se calcula una sola vez, así que si la página cambia de tamaño tendrás que recrear el tween.

## Scroll

```js
gsap.registerPlugin(ScrollTrigger);
const tl = gsap.timeline({ scrollTrigger: { trigger: "#hero-iso", start: "top 70%", end: "bottom 30%", scrub: true } });
tl.from(".capa", { y: -40, opacity: 0, stagger: .1 });
```

- Pon el ScrollTrigger en la timeline padre, no en los tweens que contiene.
- Haz funcionar la animación sin scroll primero y conéctala al final.
- Evita `scroll-behavior: smooth` en `html`, porque descuadra los cálculos.
- Si cargan imágenes sin tamaño, llama a `ScrollTrigger.refresh()` después.

## Reglas GSAP con SVG

- Las transformaciones iniciales se ponen con GSAP (`gsap.set`), no en CSS, para evitar lecturas erróneas del navegador.
- `transformOrigin` es relativo al propio elemento. `svgOrigin` usa coordenadas globales del SVG y solo admite px.
- Usa valores relativos (`"+=40"`) para poder reutilizar la animación si cambia la geometría.
- Si Chrome pinta mal un elemento, prueba con `rotation: 0.01`.

## Solo CSS

```css
@keyframes flotar { 50% { transform: translateY(-6px); } }
.iso-float { animation: flotar 4s ease-in-out infinite; }
.iso-float + .iso-shadow { animation: sombra 4s ease-in-out infinite; }
@keyframes sombra { 50% { transform: scale(.9); opacity: .7; } }   /* la sombra en el suelo sí puede escalar: es una figura del plano */
@media (prefers-reduced-motion: reduce) { .iso-float, .iso-float + .iso-shadow { animation: none; } }
```

Anima solo `transform` y `opacity`. Mover con `top` o `left` es más lento.
