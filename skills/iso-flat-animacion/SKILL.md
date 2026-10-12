---
name: iso-flat-animacion
description: "Anima ilustraciones flat isométricas para web y motion: construcción por capas, crecimiento de bloques, movimiento sobre ejes isométricos, trazados, scroll, hover y exportación a GSAP, CSS, Lottie o Rive, respetando prefers-reduced-motion. Úsala para animar o planificar el motion de escenas isométricas. Para dibujar la escena estática usa iso-flat-svg-web."
metadata:
  author: Jose Luis Pizarro Feo
  version: '1.0'
  family: iso-flat
---

# Animación flat isométrica

Parte de una escena que cumpla `iso-flat-fundamentos`, con un `<g id data-iso-part>` por objeto y caras `.face-*`. Esa estructura la generan `iso.mjs` (`iso-flat-svg-web`) y los exportadores de Blender (`iso-flat-blender`).

## Estructura: geometría en atributos, movimiento en CSS

Nunca animes `transform` sobre un elemento que ya tiene un atributo `transform`, porque lo sustituyes. Monta cada pieza en tres capas:

1. un envoltorio `data-iso-part` que se mueve en el mundo;
2. un grupo con la matriz del plano;
3. un grupo interior que gira o escala *en plano*.

Una rotación en la capa 3 se ve como rotación isométrica real. Detalle y ejemplo en `iso-flat-svg-web` (sección «Estructura preparada para animar») y `iso-flat-svg-web/assets/ejemplo-animable.svg`.

## Principio central: animar en el espacio del mundo

La proyección es paralela, así que el movimiento también tiene que serlo:

- Para desplazarte a lo largo de un eje del mundo, usa su vector proyectado (isométrica real, 1 unidad = `u` px). En CSS puro, registra `--iso-u/v/w` con `@property` y anímalas directamente (ver `iso-flat-svg-web`). Los manifiestos de Blender traen estos vectores ya calculados en `ejes_px`:
  - eje x: `(+0.866u, +0.5u)`
  - eje y: `(−0.866u, +0.5u)`
  - eje z: `(0, −u)`
  - En la dimétrica 2:1, sustituye 0.866 por 1.
- La altura nunca se anima con `scaleY`, porque deforma las aristas inclinadas. Anima un objeto de datos (`{h}`) y vuelve a calcular los puntos en `onUpdate`. También puedes usar `translateY` en la cara superior más un `clip-path` en los laterales.
- No añadas perspectiva ni escalas para simular profundidad.
- Si un objeto se mueve de modo que cambia quién tapa a quién, reordena el DOM en ese momento o divide el recorrido en tramos.

## Elige el destino

| Destino | Úsalo para | Lee |
|---|---|---|
| GSAP sobre SVG o DOM | webs, scroll, interacción y control fino | `references/recetas-gsap.md` |
| CSS keyframes/transitions | hover, bucles simples y sin JS | `references/recetas-gsap.md` (sección CSS) |
| Lottie | entregar a apps o a equipos que trabajan con After Effects | `references/lottie-rive.md` |
| Rive | estados interactivos (hover, clic, datos) en web y apps | `references/lottie-rive.md` |
| Animación hecha en Blender | movimientos con curvas reales, mecanismos, coreografías largas | `iso-flat-blender`: `iso_export_svg.py --animacion` convierte las traslaciones en `@keyframes` y las rotaciones en flipbook (sombras incluidas) |

## Flujo de trabajo

1. Escribe el guion de movimiento: para cada objeto, qué hace, cuándo (en segundos), con qué curva y si es decorativo o funcional.
2. Define la versión con movimiento reducido antes de animar nada. Normalmente es solo opacidad o el estado final, sin desplazamientos grandes.
3. Implementa con una timeline por escena y escalona por profundidad (de atrás hacia delante, o de abajo hacia arriba).
4. Revisa en 60 fps: que no haya saltos de orden de profundidad, que las sombras sigan al objeto y que los bucles no tengan cortes. Sin navegador a mano, fija un instante con `iso-render.py --fijar 'selector=transform'` y míralo. Con navegador, pausa las animaciones (`document.getAnimations()`) en varios tiempos y captura.
5. Comprueba que, con `prefers-reduced-motion: reduce`, se ve la alternativa.

La tarea está terminada cuando la animación corre fluida, respeta el movimiento reducido y conserva intactas la proyección y la luz en cada fotograma.

## Tiempos y curvas de referencia

| Efecto | Mecanismo | Curva y duración |
|---|---|---|
| Construcción por capas | opacidad y caída en z, escalonada | `back.out(1.4)`, escalonado 0.12 s |
| Torre o pila | capas de abajo arriba | `back.out(1.5)`, escalonado 0.15 s |
| Explosionar o montar | `--gap` compartido o z por capa | `power3.inOut`, 0.7–1 s |
| Elevación en hover | +12 px en z y sombra más amplia | `power2.out`, 0.25 s |
| Deriva de cámara | unos pocos px o grados, en vaivén | `sine.inOut`, 6–12 s |

Son valores de partida, no reglas. Ajústalos a la marca.

## Accesibilidad del movimiento

- Lo decorativo se elimina con movimiento reducido. Lo funcional (progreso, feedback) se simplifica a opacidad.
- Evita parpadeos con cortes de color duros y barridos grandes en x/y. Las microinteracciones pequeñas suelen estar bien.
- Si la animación dura más de 5 s o va en bucle, ofrece un botón de pausa.
