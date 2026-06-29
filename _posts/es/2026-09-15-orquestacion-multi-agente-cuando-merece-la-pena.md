---
title: "Orquestación multi-agente: cuándo el fan-out se paga solo"
date: 2026-09-15 00:00:00
lang: es
ref: multi-agent-when-worth-it
tags: [multi-agent, agents, cost, workflow]
read_min: 5
excerpt_text: "Descubres los subagentes y el instinto es desplegarlo todo — diez agentes, uno por preocupación, todos en paralelo, rápido y concienzudo. La factura vuelve 8 veces una sola pasada y la mitad reencontraron los mismos problemas. El fan-out no es paralelismo gratis; son N ventanas de contexto que no se ven entre sí."
---

Lo primero que haces al descubrir que puedes lanzar agentes es lanzar demasiados. Tenía una tarea de revisión y la desplegué: diez agentes, uno por preocupación, todos en paralelo — *se sentía* rápido y concienzudo. La factura volvió unas ocho veces una sola pasada, y la mitad de los agentes habían reencontrado de forma independiente los mismos tres problemas. Paralelo, sí. Concienzudo, no tanto. Sobre todo caro.

El fan-out parece paralelismo gratis. No lo es. Son N ventanas de contexto independientes que no saben que las otras existen, y esa propiedad es la historia entera — es lo que hace al fan-out potente en la tarea correcta y un derroche en la equivocada.

## Lo que cuesta el fan-out de verdad

Tres costes son fáciles de saltarse cuando los agentes corren en paralelo y todo se siente eficiente:

- **Los tokens no se dividen — se multiplican.** Cada agente paga el precio completo de su propio contexto. Diez agentes son unas diez veces la factura de tokens, no una décima de la latencia gratis. Reloj de pared en paralelo, coste multiplicado.
- **La coordinación es trabajo de verdad.** Alguien tiene que partir la tarea y *fusionar los resultados* — y la fusión es la parte difícil: dedup, resolver conflictos, sintetizar. Ese paso no desaparece porque paralelizaras la parte fácil.
- **Cada agente es ciego a los demás.** Se solapan (los problemas reencontrados), se pierden la costura *entre* sus rebanadas que ningún agente poseía, y no pueden construir sobre los hallazgos del otro. La independencia es la virtud y la limitación a la vez.

## Cuándo se paga solo

El fan-out gana cuando el trabajo de verdad se descompone en rebanadas que son **independientes** (ningún agente necesita ver el trabajo de otro), **sustanciales** (cada rebanada empequeñece el coste de coordinación) y mejor servidas por **amplitud que por profundidad** (cobertura o perspectivas diversas, no un solo hilo profundo y coherente). Revisar veinte ficheros que no se referencian entre sí; buscar en un espacio grande de varias formas a la vez; generar N intentos independientes para poder juzgar entre ellos. La prueba honesta: *¿se contaminaría o reventaría su presupuesto el contexto de un solo agente haciendo esto en serie?* Si sí — si el trabajo es demasiado ancho o ruidoso para una ventana — despliega, y que cada agente devuelva un [resumen escueto en vez de un transcript](/claude-code-subagentes/). Si no, estás pagando el multiplicador para nada.

## Cuándo gana un solo agente

El trabajo secuencial y dependiente — donde cada paso necesita el resultado del anterior — no es un fan-out, y forzarlo a agentes paralelos solo fabrica un problema de fusión que no tenías. Una cadena de ediciones dependientes, una tarea lo bastante pequeña como para que partirla cueste más que hacerla, cualquier cosa que necesite una sola línea de razonamiento coherente sostenida en un contexto: un agente. El default debería ser un agente; el fan-out es lo que coges cuando un agente demostrablemente no puede sostener el trabajo, no al revés.

## La forma que de verdad funciona

Cuando el fan-out *es* lo correcto, la forma que se paga no es "diez agentes, ¡vamos!". Es un pequeño pipeline:

- **Descompón por independencia real**, no por cuántas preocupaciones se te ocurra nombrar — rebanadas que de verdad no se necesitan entre sí.
- **Escalona el trabajo** — un modelo barato hace la pasada ancha, el tier caro se reserva para síntesis y juicio, la misma lógica de [emparejar el tier con la tarea](/elegir-el-modelo-segun-la-tarea/) aplicada entre agentes.
- **Presupuesta la fusión.** Haz del dedup y la síntesis un paso explícito con un responsable (a menudo un agente final), porque el coste de coordinación es real lo planearas o no.
- **Verifica, no creas al más ruidoso.** Que los hallazgos se comprueben en vez de creer al agente que enunció su conclusión con más seguridad.

## Impact

- **La factura cuadra con el trabajo.** El fan-out se gasta solo donde las rebanadas son de verdad independientes y anchas; el trabajo dependiente o pequeño se queda en un agente, así que dejas de pagar el multiplicador N× por tareas en serie.
- **La cobertura mejora donde la amplitud ayuda.** Veinte ficheros independientes o varios ángulos de búsqueda reciben cobertura paralela de verdad — el caso para el que el fan-out se construyó.
- **La fusión deja de ser una ocurrencia tardía.** Tratar el dedup y la síntesis como un paso explícito es lo que convierte diez salidas solapadas en un resultado coherente.

## Decisions

- **Por defecto un agente; despliega ante necesidad probada.** El disparador es "un contexto no puede sostener esto sin contaminarse o desbordar", no "podría paralelizar esto".
- **Despliega solo rebanadas de verdad independientes.** Si los agentes necesitan ver el trabajo del otro, no deberían ser agentes separados — lo pagarás en solape y costuras perdidas.
- **Haz de la síntesis un paso con nombre.** Presupuesta tokens y un responsable para la fusión; sin planear, es donde se esconden el coste y la inconsistencia.

## Limitations

- **El solape no desaparece del todo.** Aun con rebanadas bien partidas, se redescubre algo de contexto compartido; reduces el derroche, no lo eliminas.
- **La costura entre rebanadas no es trabajo de nadie.** Un problema que vive *entre* los ámbitos de dos agentes es el clásico fallo del fan-out — el paso de síntesis tiene que buscarlo a propósito.
- **Es fácil desplegar para sentirse productivo.** Diez agentes corriendo es una imagen satisfactoria que puede tapar una tarea que un agente habría hecho más barata y coherente — el número de agentes no es el premio.
