# Color, luz y sombra en flat isométrico

Lee este archivo cuando tengas que definir una paleta, sombrear caras, añadir sombras proyectadas o decidir si usar degradados.

## Tres tonos por material

| Cara | Valor | Ajuste desde el color base (HSL) |
|---|---|---|
| Superior | el más claro: recibe la luz del cielo | L +10–14, matiz unos 4–6° hacia cálido (amarillo) |
| Lateral iluminado | medio | color base |
| Lateral en sombra | el más oscuro | L −14–18, saturación ×0.85, matiz unos 4–6° hacia frío (azul) |

- Con luz arriba-izquierda (la convención por defecto), la cara izquierda es la media y la derecha la oscura. Con luz arriba-derecha se invierten.
- Enfriar la cara oscura y calentar la superior hace el resultado más vivo que oscurecer con negro. Nunca uses un `multiply` negro para sombrear.
- Una paleta flat funciona cuando sus colores base tienen saturación y luminosidad parecidas. Limítate a 4–6 colores base.
- Usa transparencias solo cuando sea necesario. Evita los modos de fusión (screen, multiply, overlay): se ven sucios en flat y además Lottie no los exporta.
- El script `iso-flat-svg-web/scripts/iso.mjs` implementa esta tabla en la función `shade()`.

## Una sola fuente de luz

- Fija la dirección de la luz al principio y aplícala a todos los objetos y a toda la serie.
- No mezcles ángulos de sombra: sugieren varias fuentes de luz y rompen la lectura.

## Sombras proyectadas

- La luz direccional (sol) produce rayos paralelos. La sombra de un plano sobre un plano paralelo tiene la misma forma, solo desplazada.
- Para construirla, une cada esquina del contorno superior con el suelo siguiendo la dirección de la luz, y cierra el polígono con la base. La función `flatShadow()` del script lo hace.
- Si el objeto flota, proyecta la esquina en vertical hasta el suelo y corta con la dirección de la luz.
- Hay dos estilos: sombra plana (bordes duros, opacidad 8–15 % en negro o en el color frío del suelo) o sombra con degradado (se desvanece). Usa un solo estilo por serie.
- Las sombras largas a 45° son un recurso estilístico de iconos, no una sombra física. No las mezcles con sombras isométricas construidas.

## Degradados

- En flat estricto no hay degradados. En flat "moderno" se permite un degradado lineal suave dentro de una cara, siempre en la misma dirección que la luz.
- Si la pieza se va a animar en Lottie, prefiere colores planos y degradados lineales simples.

## Contornos

- Sin contorno: el volumen se apoya solo en los tres tonos. Asegúrate de que la diferencia entre tonos sea clara.
- Con contorno: usa un grosor uniforme, uniones redondeadas y un color oscuro del mismo matiz (no negro puro).
- Aclarar o quitar el contorno donde un objeto toca el suelo hace que se vea asentado. Reforzarlo lo hace parecer más sólido o interactivo.

## Contraste para la web

- Si la ilustración transmite información (diagramas, pasos), los elementos que la explican deben contrastar al menos 3:1 con el fondo (criterio WCAG 1.4.11 para gráficos).
- Comprueba la paleta en modo oscuro si la web lo tiene. Lo normal es mantener los tres tonos y cambiar solo el fondo y la losa base.
