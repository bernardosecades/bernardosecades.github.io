---
title: "Depurar con IA: dale las pruebas, no 'arréglalo'"
date: 2026-08-04 00:00:00
lang: es
ref: debugging-with-ai
tags: [debugging, prompting, distributed-systems, workflow]
read_min: 4
excerpt_text: "Pedirle a la IA que \"arregle el bug\" te devuelve un parche seguro para un bug que no tienes. El modelo no ve tu traza, así que encaja con la causa más común y se inventa una raíz. Dale las pruebas y pídele hipótesis ordenadas."
---

Un servicio `orders` que cobra dos veces a través de `billing` — pero solo bajo carga, quizá una vez cada pocos miles de peticiones. Ese bug que no se reproduce en tu portátil y sobrevive a tres pares de ojos. Mi primer instinto fue el perezoso: pegar la función, *"aquí hay un bug de doble cobro, arréglalo."* Lo que volvió fue un parche limpio y seguro — un mutex alrededor de un bloque que nunca fue el problema.

El modelo no tenía el bug. Tenía una función y una etiqueta, así que hizo lo único que podía: encajó con la forma más común de un bug de doble cobro y parcheó eso. Plausible. Incorrecto.

## Por qué falla "arréglalo"

Sin pruebas, el modelo no puede *encontrar* una causa raíz — solo puede *inventar* la que suena más probable. Y optimiza por una respuesta creíble, no por una verdadera. Esa es la trampa: un parche seguro pero incorrecto es peor que un "no lo sé", porque parece progreso y acabarás mandándolo a producción. Cuanto menos le das al modelo, más rellena el hueco con sus priores en vez de con tu realidad.

## Dale las pruebas, no un veredicto

Depurar es trabajo de pruebas, y el modelo solo es tan bueno como lo que le pones delante. Antes de pedirle nada, entrégale:

- **La señal de verdad** — el stack trace, el error, o las líneas de log *con timestamps* que muestran la anomalía. No tu paráfrasis de ellas.
- **El camino real del código** — las dos o tres funciones por las que pasa de verdad la petición, no solo la que sospechas. El bug suele estar en el traspaso, no en la función.
- **Lo que ya has descartado** — "no son los reintentos del cliente, el log del `gateway` muestra una única llamada entrante." Esto evita que el modelo malgaste una ronda en algo que ya sabes muerto.
- **La petición correcta** — no "arréglalo" sino "dame las causas más probables, ordenadas, y la forma más barata de confirmar la primera."

## Un prompt que funciona

```text
orders cobra dos veces a billing, ~1 de cada 3000 peticiones, solo bajo carga.
Aquí tienes los logs de billing de una petición mala (dos POSTs, misma
idempotency key, con 40ms de diferencia) y los dos handlers del camino.
He descartado reintentos del cliente — el log del gateway muestra una sola llamada entrante.

No arregles nada todavía. Dame las 3 causas raíz más probables ordenadas
por probabilidad, y para la primera la única línea de log o test que
la confirme o la descarte.
```

"Ordenadas por probabilidad" obliga al modelo a razonar sobre *tus* pruebas en vez de tirar de una respuesta de catálogo. "No arregles nada todavía" le impide saltar al parche. Y pedir la *confirmación más barata* lo convierte de un oráculo que adivina la respuesta en un compañero que diseña el siguiente experimento.

## Deja que diseñe el experimento, luego córrelo

La victoria no es el parche — es el bucle. El modelo propone una línea de log discriminante o un test concreto; tú lo corres — un repro inestable puede ir [en background](/claude-code-tareas-en-background/) mientras sigues trabajando — y le pegas el resultado. Cada ronda reordena contra pruebas frescas, no contra el prior del modelo, así que dos o tres pasadas suelen sacar la causa real a la luz. (Aquí un rol se gana el sueldo como *calibración*, no como credencial: "traza qué goroutine escribe la fila primero" gana a "eres un depurador senior" — la misma lección que [descripción del puesto por encima del cargo](/roles-prompts-descripcion-no-cargo/).)

## Impact

- **Llegan menos parches seguros-pero-incorrectos a la revisión.** La salida es una hipótesis ordenada con un paso de confirmación, no un arreglo del que nadie responde.
- **El bug se cierra por una razón que sabes nombrar.** Terminas con pruebas de que *esto* era la causa, no con un cambio que hizo desaparecer el síntoma por ahora.
- **La conversación se vuelve un registro.** Las hipótesis ordenadas y los experimentos que las descartaron son el post-mortem, ya escrito.

## Decisions

- **Retén el veredicto a propósito.** Pide hipótesis, no arreglos. El arreglo es el último paso, no el primero.
- **Da el camino, no el sospechoso.** Entrega todo el flujo de la petición; el bug suele esconderse en el traspaso entre funciones, que no ves si solo pegas la que culpas.
- **Haz barata la confirmación antes de hacerla correcta.** Una línea de log que distingue dos causas vale más que un parche que asume una de ellas.

## Limitations

- **Pruebas basura, hipótesis basura.** Si tus logs no capturan la anomalía, el modelo adivina contigo — primero va la instrumentación.
- **Los heisenbugs siguen mordiendo.** Añadir una línea de log puede mover el timing lo justo para esconder una race. El modelo puede avisarte, pero no puede sentir que pasa.
- **No sustituye a conocer tu sistema.** El modelo ordena lo plausible; tú sigues decidiendo qué es físicamente posible según cómo hablan de verdad tus servicios.
