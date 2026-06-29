---
title: "Economía de tokens: mide antes de optimizar"
date: 2026-09-22 00:00:00
lang: es
ref: token-economics
tags: [cost, tokens, performance, workflow]
read_min: 5
excerpt_text: "La factura sube y el instinto es acortar el prompt. Pero recortar palabras es optimizar la línea equivocada — el gasto real es estructural: contexto reenviado en cada turno, sin caché, el tier equivocado. Mide primero, y la línea cara casi nunca es la que adivinarías."
---

La factura de tokens del pipeline de extracción de `orders` fue subiendo, y mi primer instinto fue el obvio: recortar el prompt — apretar la redacción, cortar los ejemplos, dejarlo escueto. Antes de hacerlo, saqué el desglose real de uso. La prosa del prompt era un 2% del gasto. El ochenta por ciento era reenviar el mismo contexto grande en cada turno, sin caché. Una tarde puliendo palabras del prompt habría sido una tarde gastada en la línea equivocada por completo.

El coste de tokens es un problema de ingeniería con la misma trampa que cualquier otro problema de rendimiento: el cuello de botella casi nunca está donde apunta tu intuición, y optimizar antes de medir es solo adivinar con pasos extra.

## Mide antes de recortar

No puedes optimizar una factura que no has desglosado. Consigue las cuentas *exactas* — la API reporta tokens por llamada, así que no hay excusa para calcular a ojo o tirar de un tokenizador genérico que está mal para el modelo. Luego mira el desglose por los ejes que importan: input vs output, cacheado vs no cacheado, por-llamada vs por-bucle. La línea cara se revela al instante, y suele ser estructural — un contexto reenviado, un prefijo sin cachear, un tier de modelo demasiado alto para la tarea — no la redacción que estabas a punto de agonizar.

## Las palancas grandes, en orden

Una vez has medido, gasta esfuerzo de arriba abajo:

- **Caché.** Un prefijo estable enviado en cada petición — system prompt, instrucciones, el documento grande compartido — debería estar cacheado; una lectura de caché cuesta una fracción del input fresco. Y vigila los fallos *silenciosos*: un timestamp o un request ID cerca del principio del prefijo invalida la caché en cada llamada y duplica tu factura en silencio mientras parece todo bien.
- **El contexto que reenvías.** Los bucles agénticos largos reenvían toda la historia en cada turno, así que un transcript inflado es un impuesto que pagas en cada paso. Poda o compacta la salida de herramientas obsoleta antes de que componga — esto es la [ventana de contexto como presupuesto](/claude-code-ventana-contexto/), vista desde el lado del coste.
- **El tier.** Una tarea acotada y de alto volumen corriendo sobre el buque insignia es el sobrecoste más común — [empareja el tier con la tarea](/elegir-el-modelo-segun-la-tarea/).
- **Fan-out.** N agentes son N× los tokens; [despliega solo cuando se paga solo](/orquestacion-multi-agente-cuando-merece-la-pena/).

Los micro-ahorros — como meter la salida de shell directa con el [atajo `!`](/claude-code-atajo-bang/) en vez de dar la vuelta por el modelo — son reales, pero son el paso cinco, no el uno. No empieces ahí.

## Cuándo el ahorro no compensa

Aquí está la parte fácil de olvidar a media optimización: los tokens son baratos, y tu tiempo no. Recortar un 10% a una llamada que cuesta céntimos y corre dos veces al día es una pérdida neta en cuanto te cuesta una tarde. Optimiza las llamadas que son **calientes** (alto volumen) o **gordas** (contexto enorme); para todo lo demás, el movimiento más barato es dejarlo en paz. El token-golf prematuro es el mismo error que el tuning de rendimiento prematuro — esfuerzo vertido en una línea que nunca fue el problema, pagado con el tiempo que no gastaste en una que sí.

## Un hábito barato que se paga

Loguea el uso en cada llamada — input, output, cacheado, no cacheado — para que el desglose esté siempre ahí cuando la factura se mueva. Cuesta casi nada añadirlo y cambia el modo de fallo: "la factura subió" deja de ser una investigación que corres bajo presión y se vuelve un número que ya tienes. La tarde que *no* gasté puliendo palabras del prompt la recuperé porque el log de uso me dijo dónde estaba de verdad el 80%.

## Impact

- **La optimización aterriza en la línea cara.** Medir primero redirige el esfuerzo de la redacción del prompt (2%) al contexto reenviado (80%) — la misma hora de trabajo, un orden de magnitud más ahorrado.
- **Se cazan los fallos de caché silenciosos.** Vigilar cacheado-vs-no-cacheado convierte una factura duplicada en silencio en un arreglo de una línea en vez de una subida lenta e inexplicada.
- **Las llamadas baratas se dejan en paz.** Saber qué llamadas son calientes o gordas significa que el resto no absorbe tiempo de ingeniería que cuesta más que los tokens.

## Decisions

- **Desglosa la factura antes de cambiar nada.** Las cuentas exactas por llamada deciden dónde gastar esfuerzo; la intuición apunta de forma fiable a la línea equivocada.
- **Trabaja las palancas de arriba abajo.** Caché y contexto reenviado antes que tier antes que fan-out antes que micro-ahorros — el orden es más o menos el orden de impacto.
- **Para cuando el tiempo de ingeniería supera la factura.** Una llamada barata y rara no vale una tarde; optimiza lo caliente o lo gordo, deja el resto.

## Limitations

- **La caché tiene sus propias reglas.** Un prefijo cacheado solo ayuda si es de verdad estable y lo bastante largo como para calificar; mal usado, pagas primas de escritura por lecturas que nunca llegan.
- **Tokens más baratos pueden costar precisión.** Recortar contexto o bajar a un tier menor ahorra dinero justo hasta que cambia la respuesta — mide la calidad, no solo la factura.
- **Precios y ratios se mueven.** El *orden* de las palancas es duradero; el break-even exacto entre "optimiza" y "déjalo" cambia según el precio y tu volumen de llamadas — re-mide, no memorices.
