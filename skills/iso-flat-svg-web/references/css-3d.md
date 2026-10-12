# Isométrico con CSS 3D

Lee este archivo cuando el contenido isométrico tenga que ser HTML real: tarjetas, rejillas de producto, mockups de UI o capas con hover.

## Plano base

```css
.iso-scene { transform-style: preserve-3d; }          /* sin perspective */
.iso-plane {
  transform: rotateX(54.7356deg) rotateZ(-45deg);    /* isométrica real */
  /* transform: rotateX(60deg) rotateZ(-45deg);         dimétrica 2:1 */
  transform-style: preserve-3d;
}
.iso-plane * { transform-style: preserve-3d; }        /* en todos los descendientes que tengan que vivir en 3D */
```

- `rotateX(60deg)` aplasta el plano al 50 % y da un rombo 2:1, que es lo que publica la mayoría de tutoriales. Elige con intención y no mezcles las dos.
- No uses `perspective`, o ponle un valor enorme. Con perspectiva corta la escena parece una tarjeta inclinada y deja de ser isométrica.
- `transform-style: preserve-3d` hay que declararlo en cada nivel intermedio. Si no, Firefox y otros navegadores aplanan los hijos.

## Alturas y capas

```css
.layer { transform: translateZ(calc(var(--z) * 1px)); }
```

- La altura se consigue con `translateZ`, nunca con `scale`.
- Para una vista explosionada, usa una variable compartida `--gap` y calcula `translateZ(calc(var(--i) * var(--gap)))`.

## Bloques sólidos

Un bloque son tres caras: la superior es el propio elemento y los laterales se hacen con hijos o pseudo-elementos rotados 90°.

```css
.block { position: relative; width: 100px; height: 100px; background: var(--top); }
.block .left  { position: absolute; inset: 0; background: var(--left);  transform-origin: bottom; transform: rotateX(-90deg); height: var(--h); top: calc(100% - var(--h)); }
.block .right { position: absolute; inset: 0; background: var(--right); transform-origin: right;  transform: rotateY(90deg);  width: var(--h);  left: calc(100% - var(--h)); }
```

Ajusta los signos según la orientación del plano. Comprueba siempre en el navegador que las caras queden unidas sin huecos.

## Hover de elevación

Sigue el patrón de la rejilla de Envato Tuts+:

```css
.card img    { transform: translate3d(0,0,20px); transition: transform .3s cubic-bezier(.165,.84,.44,1); }
.card .shadow{ background: rgba(0,0,0,.1); box-shadow: 0 0 5px 1px rgba(0,0,0,.1); opacity:.9; transition: all .3s cubic-bezier(.165,.84,.44,1); }
.card:hover img     { transform: translate3d(0,0,50px); }
.card:hover .shadow { opacity:.6; box-shadow: 0 0 10px 6px rgba(0,0,0,.1); }
```

Cuanto más se aleja el objeto del suelo, más ancha y difusa debe ser su sombra.

## Límites

- No construyas una interfaz funcional entera en isométrico: cuesta interactuar con ella. Resérvalo para heros, infografías y diagramas.
- El texto rotado en 3D pierde nitidez. Mantén el texto importante plano a pantalla, por encima de la escena.
- Las zonas de clic de los elementos transformados no coinciden con su caja original. Pruébalas con el ratón y con el teclado.
