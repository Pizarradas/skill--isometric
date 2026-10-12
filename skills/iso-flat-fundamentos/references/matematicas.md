# Matemática isométrica (referencia)

Lee este archivo cuando necesites valores exactos: código, transformaciones, cámaras o conversión de coordenadas.
Todos los números se han verificado numéricamente (ver `fuentes.md`).

## Constantes

| Magnitud | Isométrica real | Dimétrica 2:1 |
|---|---|---|
| Ángulo de las aristas de suelo respecto a la horizontal | 30° | 26.565° = atan(1/2) |
| Ángulos entre ejes en pantalla | 120°, 120°, 120° | 116.565°, 116.565°, 126.870° |
| Elevación de cámara sobre el suelo | 35.264° = asin(tan 30°) = atan(1/√2) | 30° |
| Rotación X de cámara en Blender (Z = 45°, Y = 0) | 54.736° = atan(√2) | 60° |
| Rotación X en CSS (con rotateZ ±45°) | 54.7356deg | 60deg |
| Proporción alto/ancho del rombo de suelo | 1 : √3 ≈ 0.577 | 1 : 2 = 0.5 |
| Escorzo de cada eje en la isométrica real | √(2/3) ≈ 0.8165 | — |
| Píxel art: escalón de línea | — | 2 px horizontales por 1 px vertical |

## Proyección mundo → pantalla (y de pantalla hacia abajo)

Ejes del mundo: x hacia abajo-derecha, y hacia abajo-izquierda, z hacia arriba.

```js
// Isométrica real
sx = (x - y) * Math.cos(Math.PI / 6) * unit;   // 0.8660254
sy = ((x + y) * 0.5 - z) * unit;
// Dimétrica 2:1 (tiles de ancho W y alto W/2)
sx = (x - y) * (W / 2);
sy = (x + y) * (W / 4) - z * zUnit;
```

Pantalla → suelo (z = 0), isométrica real:

```js
x = (sx / (unit * 0.8660254) + sy / (unit * 0.5)) / 2;
y = (sy / (unit * 0.5) - sx / (unit * 0.8660254)) / 2;
```

## Matrices SVG por plano (isométrica real)

Dibuja la forma en plano (sin deformar) dentro de un `<g>` con la matriz de su cara. `offset` desplaza el plano a lo largo de su normal, en unidades del mundo (multiplica por `unit` si trabajas en px). En las caras laterales, el eje y local apunta hacia abajo (−z), igual que en SVG.

| Plano | `matrix(a, b, c, d, e, f)` |
|---|---|
| Suelo/techo (XY) | `matrix(0.866, 0.5, -0.866, 0.5, 0, -offset)` |
| Cara izquierda (XZ, normal +y) | `matrix(0.866, 0.5, 0, 1, -0.866*offset, 0.5*offset)` |
| Cara derecha (YZ, normal +x) | `matrix(0.866, -0.5, 0, 1, 0.866*offset, 0.5*offset)` |

La cara derecha admite también la forma espejada `matrix(-0.866, 0.5, 0, 1, …)`, que dibuja desde la arista contraria. Es válida para formas simétricas, pero invierte el texto. Para texto usa la de la tabla. `iso-check.py` acepta ambas.

Equivalente con DOMMatrix para el plano de suelo: `new DOMMatrix().rotate(30).skewX(-30).scale(1, 0.86602)`.
Con 0.8602 (valor que aparece en un artículo) el eje y queda un 0.7 % corto. Usa 0.86602.

Dimétrica 2:1, plano de suelo: `matrix(1, 0.5, -1, 0.5, 0, 0)`. Multiplícala por la escala que quieras.

## SSR en Illustrator (escalar, sesgar, rotar)

Parte de un cuadrado. Aplica en este orden: escalar en vertical, sesgar en horizontal y rotar.

| Cara | Escala vertical | Sesgado | Rotación |
|---|---|---|---|
| Superior | 86.602 % | 30° | −30° |
| Izquierda | 86.602 % | −30° | −30° |
| Derecha | 86.602 % | 30° | 30° |

Comprobación: las aristas que en la vista ortográfica forman 90° o 180° deben caer a 30° o 150° en la isométrica.

## CSS 3D

```css
.iso-true  { transform: rotateX(54.7356deg) rotateZ(-45deg); transform-style: preserve-3d; }
.iso-2to1  { transform: rotateX(60deg)      rotateZ(-45deg); transform-style: preserve-3d; }
/* No pongas perspective, o pon un valor muy grande: la proyección es paralela. */
```

## Orientaciones

Hay 8 orientaciones isométricas posibles. En cámaras, girar Z ±90° (45°, 135°, 225°, 315°) mantiene la isométrica vista desde otro lado.
Si giras la cámara, tienes que reasignar qué cara es "izquierda" y cuál es "derecha" para conservar la luz.
