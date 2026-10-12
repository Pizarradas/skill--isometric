---
name: iso-flat-prompts-imagen
description: "Redacta y revisa prompts para generar ilustraciones flat isométricas con modelos de imagen (Midjourney, GPT Image, Flux, Imagen y similares): proyección sin perspectiva, tres tonos, paleta fija, coherencia entre series y revisión del resultado para web o animación. Úsala cuando la imagen la vaya a generar una IA ráster. Para SVG o código usa iso-flat-svg-web."
metadata:
  author: Jose Luis Pizarro Feo
  version: '1.0'
  family: iso-flat
---

# Prompts de imagen para flat isométrico

Los modelos de imagen tienden a meter perspectiva, sombras suaves y texturas. La palabra "isometric" sola no basta: hay que describir la proyección, la luz y lo que no queremos.

## Estructura del prompt (en inglés, que es lo que mejor entienden los modelos)

Usa este orden y rellena `templates/prompt-iso.md`:

1. **Tipo y estilo**: `flat isometric vector illustration, clean geometric shapes, solid colors`.
2. **Proyección**: `true isometric projection, 30-degree axonometric angles, parallel projection, no vanishing point, no perspective distortion, vertical lines perfectly vertical`.
3. **Contenedor primero**: `isometric floating platform / tile, rectangular, ...`.
4. **Objetos relativos al contenedor**: posición (front-left, rear-right...), tamaño relativo y color. Lo importante va delante.
5. **Luz y sombreado**: `single light source from upper left, three distinct face values: top face lightest, left face mid-tone, right face darkest, hard-edged flat shadows, no gradients`.
6. **Paleta**: 4–6 colores con hex o nombres precisos, más el fondo.
7. **Encuadre para web**: espacio vacío (`generous empty space on the right for headline`) y fondo liso o transparente.
8. **Exclusiones**: en Midjourney van con `--no` (separadas por comas y siempre al final). En los demás modelos, como frase `Avoid: ...`.

## Reglas por herramienta

- **Midjourney**: los parámetros van al final, separados del texto por un espacio y sin puntuación (`--ar 16:9 --raw --no perspective, gradient, texture, photorealistic`). Para series usa `--sref` con una imagen de referencia y `--seed` fijo. `--raw` reduce el estilo propio del modelo. `--no` lee cada palabra por separado, así que evita combinaciones que se malinterpreten.
- **Modelos conversacionales (GPT Image, Gemini/Imagen)**: escribe frases completas. Pide "transparent background" si lo admiten y adjunta una imagen de referencia de estilo cuando haya serie.
- **Flux, SDXL y similares**: los prompts negativos funcionan mejor. Usa una LoRA isométrica si hay. Fija el seed y la resolución.

## Coherencia de serie (iconos o escenas de una misma web)

- Mantén igual en todas: el mismo bloque de proyección, luz y paleta (cópialo tal cual), la misma referencia de estilo y el mismo encuadre o proporción.
- Cambia solo el bloque de objetos.
- Genera primero una pieza "maestra" y úsala como referencia para las demás.

## Blender como estructura (proyección garantizada)

Los modelos de imagen no garantizan la geometría. Para escenas exigentes, invierte el orden:

1. Bloquea la escena en Blender con formas simples y ejecuta `iso_setup.py` (`iso-flat-blender`). El render da la proyección, el encuadre y los tres tonos exactos.
2. Usa ese render como guía estructural del modelo de imagen:
   - imagen de referencia o img2img con fuerza media en los modelos conversacionales;
   - imagen de partida en Midjourney;
   - ControlNet (depth, canny o lineart) en Flux/SDXL.
   El prompt aporta el estilo, los materiales y el detalle; la imagen aporta la geometría.
3. Revisa con los cinco puntos de abajo. Si la pieza se va a animar, monta la versión final con `iso_export_svg.py` o `iso_export_capas.py` a partir del mismo `.blend`, no desde la imagen generada.

## Revisión obligatoria del resultado

1. Comprueba que las verticales son verticales y que las aristas paralelas no convergen. Si convergen, regenera reforzando "no vanishing point".
2. Comprueba que las tres caras siguen la misma luz en todos los objetos.
3. Busca texto basura, manos o detalles fundidos, y bórralos o regenera.
4. Comprueba el fondo y los márgenes para el layout web.
5. Si se va a animar, conviértelo a vector (trazado manual o vectorizador). Luego reconstruye las piezas por capas con `iso-flat-svg-web`, porque una imagen ráster plana no se puede animar por partes.

La pieza está lista cuando pasa estos cinco puntos y el control de calidad de `iso-flat-fundamentos`.

## Personajes

Desde arriba se ven más la coronilla, los hombros y los sombreros que la cara. Pide `slightly oversized head, readable silhouette from above` y una sombra de contacto plana con forma de rombo.
