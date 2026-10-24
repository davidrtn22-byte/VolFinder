# VolFinder

Aplicación web para el laboratorio de Análisis Numérico (UPB). Estima el
volumen de un envase a partir de **una sola fotografía**, modelándolo como un
sólido de revolución V = π∫r(z)²dz e integrando con cuatro métodos escritos a
mano (trapecio, Simpson 1/3, Newton-Cotes 3/8 y Gauss-Legendre) sobre un
spline cúbico natural también escrito a mano.

**100 % automática:** la persona solo toma o sube la foto (envase + una moneda
colombiana). La app detecta sola la escala, la silueta, el eje y el perfil. No
hay clics ni medidas manuales sobre la imagen.

## Cómo ejecutarla

Dos archivos en la misma carpeta: `index.html` y `opencv-wasm.b64gz.txt`
(OpenCV.js empaquetado). Debe servirse por http(s) (GitHub Pages sirve);
abrir con doble clic (`file://`) puede bloquear la carga de OpenCV.

## Protocolo de la foto

- Envase completo en el encuadre y, a su lado, una moneda colombiana ($100,
  $200, $500 o $1.000) **a la misma distancia de la cámara** que el envase,
  sin tocarlo ni taparlo, grande en la foto (≥ 5 % del ancho).
- **Envase de pie:** moneda de pie a su lado (o acostada sobre la mesa,
  eligiéndolo en "Posición de la moneda"); cámara de frente o algo elevada
  (hasta ≈ 45°).
- **Envase acostado:** moneda acostada sobre la mesa junto a él; cámara desde
  arriba o en diagonal, con el envase atravesado frente a la cámara.
- Cualquier fondo sirve si el envase contrasta con él; sin reflejos fuertes;
  usar la foto original de la cámara (conserva la distancia focal en el EXIF).

Una sola foto solo da proporciones en píxeles: para obtener centímetros hace
falta un objeto de tamaño conocido (la moneda). No es una limitación del
software sino de la geometría de una vista.

## Cómo funciona la detección (`detectScene`)

OpenCV.js se usa **solo para procesar la imagen**; nada de interpolar ni
integrar usa librerías.

1. **Segmentación** con GrabCut (aprende los colores de fondo y objeto, no
   exige fondo uniforme ni se confunde con etiquetas blancas). Una segunda
   máscara por diferencia de color con la mesa ayuda a no perder monedas
   bimetálicas (centro plateado).
2. **Moneda como elipse:** ajuste de elipse sobre el contorno y refinado con
   bordes Canny a resolución completa. Eje mayor → escala en píxeles; relación
   menor/mayor q → inclinación del plano de la moneda → ángulo de la cámara.
3. **Eje del envase:** eje principal de la silueta (momentos de imagen) y
   refinado por mínimos cuadrados escritos a mano; funciona con el envase de
   pie, acostado o en diagonal. El perfil r(z) se mide en el marco del propio
   eje.
4. **Reconstrucción con cámara estenopeica:** distancia focal desde EXIF
   (`FocalLengthIn35mmFilm`; si no hay, se asume un celular típico). Cada punto
   del eje del envase tiene su propia profundidad y escala:
   - acostado: el eje es una recta a altura = radio máximo sobre la mesa;
   - de pie con moneda de pie: las profundidades de los puntos de contacto con
     la mesa (base del envase y base de la moneda) salen de sus filas en la
     imagen y el ángulo de cámara;
   - de pie con moneda acostada: la moneda define el plano de la mesa.
5. **Validaciones:** si largo, radio o proporciones no son las de un envase, o
   la silueta ocupa casi toda la foto, la app lo dice en lugar de mostrar un
   número absurdo. Avisos por moneda muy pequeña, envase cortado, cámara muy
   rasante o sin EXIF.

## Integración numérica

`n = 60` (par y múltiplo de 3) para trapecio/Simpson/Newton-Cotes 3/8 sobre
malla uniforme; Gauss-Legendre de 4 puntos en 15 subintervalos. El spline
cúbico natural se resuelve con el algoritmo de Thomas.

## Las tres medidas de volumen (paso 6)

- **Volumen geométrico calculado** (los 4 métodos).
- **Capacidad nominal de etiqueta** (solo referencia, nunca entra al error).
- **Volumen experimental medido** (p. ej. pesando el agua: 1 g ≈ 1 mL). Es el
  único contra el que se calcula el error.

## Exactitud esperada (simulación con imágenes sintéticas)

| Montaje | Error típico |
|---|---|
| Envase de pie, moneda de pie, cámara 5–45° | ≈ 1–8 % |
| Envase de pie, moneda acostada, cámara 20–50° | ≈ 1–8 % |
| Envase acostado, moneda acostada, envase de lado a la cámara | ≈ 1–8 % |
| Cámara casi cenital (> 60°) o envase acostado en diagonal respecto a la cámara | 8–15 % |
| Moneda a distinta distancia que el envase (≈ 2 cm de diferencia) | ≈ ±10 % |
| Foto sin EXIF (distancia focal supuesta) | hasta ≈ ±10–20 % |

Estas cifras salen de simulaciones, no de fotos reales: hay que validar con
las botellas de la entrega.
