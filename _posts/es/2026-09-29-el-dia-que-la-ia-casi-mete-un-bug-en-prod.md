---
title: "El día que la IA casi mete un bug en prod — y qué cambié"
date: 2026-09-29 00:00:00
lang: es
ref: ai-nearly-shipped-a-bug
tags: [failure-cases, code-review, reliability, workflow]
read_min: 5
excerpt_text: "Un diff de IA limpio, una suite en verde, una explicación segura — tres luces verdes, y casi aprobé un bug de doble cobro. La lección no fue 'no uses IA'. Fue que el código, los tests y la explicación venían de un mismo modelo, así que nunca se corroboraron entre sí."
---

Viernes por la tarde, con prisa. El modelo había producido un cambio prolijo en el manejo de idempotencia del camino de cobro `orders` → `billing` — del tipo de arreglo que he pedido cien veces. El diff se leía bien. La suite estaba en verde. Cuando le pedí que explicara el cambio, la explicación fue nítida y convincente. Tenía el cursor sobre aprobar. La única razón por la que no lo hice es un runrún vago e inmerecido de que ya había visto esta forma de "todo está bien" antes — y ese runrún, no mi proceso, es lo único que lo cazó.

El cambio estaba mal. No obviamente mal — *plausiblemente* mal, que es peor.

## Qué pasó de verdad

Al apretar la comprobación de idempotencia, el modelo había quitado en silencio un guard que solo importaba bajo reintentos concurrentes. En el mundo de petición única que ejercitaban los tests, era impecable. Bajo dos reintentos casi simultáneos del mismo cobro — el escenario exacto para el que el guard existía — cobraría dos veces. Cada señal delante de mí decía *mándalo*: el diff limpio, la suite en verde, el autor seguro. El bug vivía justo en el hueco que ninguna de esas señales cubría.

## La lección de fondo: los fallos de la IA están correlacionados

Esto es lo que tardé en ver. Había tratado tres cosas como confirmación independiente — el código se ve bien, los tests pasan, la explicación tiene sentido — cuando las tres venían del *mismo modelo*. El diff que engañó a mi revisión también escribió los tests que salieron verdes (comprobaban el comportamiento de petición única, nunca el concurrente) y también produjo la explicación que me convenció pasando por encima de mi duda. Eso no son tres testigos coincidiendo. Es una sola fuente coincidiendo consigo misma, tres veces, en tres formatos.

Esta es la trampa debajo de cada modo de fallo individual sobre el que había escrito: [código plausiblemente incorrecto](/revisar-codigo-generado-por-ia-antes-de-confiar/), [tests en verde que no prueban nada](/testing-con-ia-verde-no-es-probado/), una voz segura. Cada uno es sobrevivible a solas. Apilados, fabrican un consenso que se siente como seguridad y no lo es — porque el consenso tiene un único punto de origen.

## Qué cambié

El arreglo no es más vigilancia en el momento. Es asegurar que al menos una comprobación venga de *fuera del propio bucle del modelo*:

- **Leo por comportamiento, no por pulcritud.** No "¿está limpio este diff?" sino "¿qué hace esto cuando a `billing` le llegan dos reintentos con 40ms de diferencia?". La pulcritud es la fortaleza del modelo y justo lo que no debe tranquilizarte.
- **El test decisivo es uno mío.** Escrito contra la especificación — *el guard debe aguantar bajo reintentos concurrentes* — no generado contra el código que se supone que comprueba. A un test spec-first no se le convence de no fallar.
- **"Parece terminado" es el freno, no la luz verde.** Cuanto más suave se siente todo, más deliberadamente miro ahora la costura — porque suave es lo que la confianza correlacionada se siente por dentro.

Y cuando quiero una segunda opinión, viene de un [revisor sin nada en juego](/claude-code-escritor-revisor/) leyendo el diff en frío, o de un humano — algo que no escribió el código, así que su acuerdo de verdad cuenta como una segunda fuente.

## Qué no cambié

Sigo usando IA para todo, cada día — el código, los tests, el primer borrador de la explicación. El apalancamiento es real y no lo voy a devolver. El cambio fue un hábito, no una retirada: dejar de que el modelo califique su propio trabajo, y aportar exactamente una comprobación que él no pueda haber escrito. Eso es seguro barato, no una pérdida de velocidad.

## Impact

- **Un casi-fallo no se convirtió en incidente.** El doble cobro se cazó en revisión en vez de en producción — y la caza es ahora un proceso, no un runrún con suerte.
- **La comprobación independiente es barata y portante.** Un test spec-first más una lectura centrada en el comportamiento cuestan minutos y rompen la trampa de la confianza correlacionada que ninguna cantidad de mirar un diff limpio habría roto.
- **Mi confianza se calibró, no se retiró.** Me apoyo en la IA exactamente igual de fuerte que antes, pero dejé de contar su auto-corroboración como evidencia.

## Decisions

- **Trata el código, los tests y la explicación de la IA como una sola fuente.** No se confirman independientemente entre sí; exige al menos una comprobación de fuera de ese bucle.
- **Posee el test que importa.** Los tests generados valen para cobertura; el que guarda el comportamiento que te pone nervioso, lo escribes tú contra la especificación.
- **Frena donde parece terminado.** La señal de "se ve bien" más fuerte es justo donde se esconde el fallo correlacionado — haz que la suavidad dispare escrutinio, no alivio.

## Limitations

- **Un runrún no es un proceso — y casi no lo tengo.** La versión honesta de esta historia es que lo cazó la suerte; los cambios son lo que convierte "tuve suerte una vez" en "lo cazaría la próxima", pero solo funcionan si de verdad los corro bajo presión de deadline.
- **Las comprobaciones de fuera del bucle tienen un coste.** Un test spec-first y una lectura cuidadosa por comportamiento llevan tiempo; en cambios de verdad de bajo riesgo, esa ceremonia no siempre compensa — sigue haciendo falta criterio.
- **Esto generaliza solo hasta cierto punto.** El fallo correlacionado es la lente para el trabajo *escrito por IA*; no es una teoría universal de los bugs, y sobre-aplicarla haría que cada diff limpio se sintiera como una trampa cuando la mayoría no lo son.
