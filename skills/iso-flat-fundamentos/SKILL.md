---
name: iso-flat-fundamentos
description: "Reglas base de la ilustración flat isométrica: elegir proyección (isométrica real 30° o dimétrica 2:1), rejilla, sombreado de tres tonos, sombras, orden de profundidad y control de calidad. Úsala antes de diseñar, generar o revisar cualquier ilustración, icono o escena isométrica plana. No sirve para perspectiva cónica ni para render 3D realista."
metadata:
  author: Jose Luis Pizarro Feo
  version: '1.0'
  family: iso-flat
---

# Fundamentos de flat isométrico

Esta skill es la base de la familia `iso-flat-*`. Las demás dan por hecho que sigues estas reglas:
`iso-flat-svg-web`, `iso-flat-animacion`, `iso-flat-blender` e `iso-flat-prompts-imagen`.

Convención de ejes común a todo el kit (código, Blender y documentación): x hacia abajo-derecha, y hacia abajo-izquierda y z hacia arriba. El observador mira desde +x +y +z y ve las caras superior (z máx), derecha (x máx) e izquierda (y máx). En Blender, con la cámara estándar, x = +X, y = −Y y z = +Z.

## 1. Fija la especificación antes de dibujar

Escribe estas cinco decisiones al principio de cualquier tarea y no las cambies a mitad:

| Decisión | Opciones | Por defecto |
|---|---|---|
| Proyección | `iso` (30°, ejes a 120°) / `dimetric` (2:1, 26.565°) | `iso` para vector y web; `dimetric` para pixel art y tiles |
| Unidad de rejilla | p. ej. 8, 16, 40 px por unidad de mundo | 40 px en SVG |
| Luz | una sola, direccional | arriba-izquierda: top claro, cara izquierda media, cara derecha oscura |
| Paleta | 4–6 colores base, cada uno con 3 tonos | saturación y luminosidad parecidas entre colores |
| Estilo de contorno | sin trazo / trazo fino uniforme | sin trazo (el volumen lo dan los tonos) |

No mezcles proyecciones en una misma pieza o serie. Muchas fuentes llaman "isométrico" a la 2:1; nombra siempre cuál usas.

## 2. Reglas geométricas que no se negocian

- Las verticales siempre son verticales. Las aristas del suelo van a ±30° (iso) o con pendiente 1:2 (dimétrica).
- No hay punto de fuga: las aristas paralelas se mantienen paralelas y el tamaño no cambia con la distancia.
- La altura se expresa desplazando en z, nunca escalando el objeto.
- Construye todo con primitivas (prisma, cilindro, cuña, esfera), luego suma o resta. Ancla los vértices a la rejilla.
- Los círculos del suelo son elipses con proporción 1:√3 (iso) o 1:2 (dimétrica). Inscríbelas siempre en un rombo de construcción.
- Para la matemática exacta (matrices, transformaciones SSR, ángulos de cámara, conversión pantalla↔mundo) lee `references/matematicas.md`.

## 3. Color, luz y sombra

- Cada material lleva tres tonos fijos: superior, lateral iluminado y lateral en sombra. Ningún objeto puede romper esa asignación.
- Para mover tonos, las reglas de matiz, sombras proyectadas, oclusión y degradados permitidos, lee `references/color-luz.md`.

## 4. Composición

- Apoya la escena en una plataforma o losa flotante y describe primero el contenedor y luego los objetos sobre él.
- Pon los objetos importantes delante (abajo en pantalla). Evita meter detalle denso en la parte trasera, porque se solapa.
- Deja aire: los objetos flotantes y las alineaciones precisas dan ligereza.
- El texto va plano a pantalla o proyectado exactamente sobre un plano (top, left o right). Nunca a medias.
- Dibuja de atrás hacia delante (algoritmo del pintor). Ordena por `x + y` y luego por `z`. Si hay cajas que se cruzan o tamaños muy distintos, construye un grafo de "está detrás de" y ordénalo topológicamente.

## 5. Control de calidad (pásalo siempre antes de entregar)

1. Todas las verticales son verticales y todas las aristas de suelo tienen el ángulo de la proyección elegida.
2. Los tres tonos por cara son coherentes en todos los objetos, y todas las sombras caen hacia el mismo lado.
3. Ninguna arista paralela converge y nada se ha escalado para simular distancia.
4. Las elipses están inscritas en rombos y no son círculos.
5. El orden de dibujo es correcto: ningún objeto trasero tapa a uno delantero.
6. La paleta y el contorno coinciden con la especificación de la sección 1.

7. Si el entregable es SVG: `iso-flat-svg-web/scripts/iso-check.py` da 0 errores. Además, la has mirado renderizada (`iso-render.py`), no solo leído el código.

Si falla un punto, corrige y vuelve a revisar. No entregues con fallos conocidos.

## Errores frecuentes en las fuentes

- Hay tutoriales que escriben "86.062 %" para la escala SSR. El valor correcto es 86.602 % (cos 30°).
- `rotateX(60deg) rotateZ(45deg)` en CSS da una vista dimétrica 2:1, no isométrica real. La isométrica real necesita `rotateX(54.7356deg)`.
- En Blender, X = 60° da la vista "de juego" 2:1. La isométrica real necesita X = 54.736°.

Las fuentes y la verificación numérica están en `references/fuentes.md`.
