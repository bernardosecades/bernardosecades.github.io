---
title: "Claude Code: deja de mirar el terminal — ejecuta comandos largos en background"
date: 2026-07-21 00:00:00
lang: es
ref: claude-code-background-tasks
tags: [claude-code, background, workflow, productivity]
read_min: 4
excerpt_text: "Una suite de integración que tarda cuatro minutos se convierte en cuatro minutos mirando un terminal congelado, bloqueado por un resultado sobre el que aún no puedes actuar. Lánzala en segundo plano: Claude dispara el comando, sigue trabajando en otra cosa, y se reactiva solo en cuanto termina — con el resultado ya en la mano."
---

Un comando largo es un pequeño impuesto que pagas una y otra vez. `make test-integration` tarda cuatro minutos; un build de imagen Docker, más. Pulsas Enter y a esperar — el terminal está ocupado, Claude está ocupado, y te quedas mirando una barra de progreso terminar un trabajo sobre el que no puedes actuar hasta que acabe. Multiplícalo por cada ejecución de tests del día y la espera suma tiempo real, gastado en no hacer nada.

El arreglo es dejar de tratar "ejecutar un comando" como algo que tiene que bloquear. Lánzalo en segundo plano.

## La idea: lánzalo en background y sigue

En vez de dejar que un comando largo tenga la sesión secuestrada, le pides a Claude que lo ejecute en segundo plano. El comando se desacopla y sigue corriendo a través de los turnos, y Claude no se sienta a esperarlo — pasa directo a lo siguiente: editar otro fichero, redactar el próximo cambio, responder una pregunta. Los dos trabajáis mientras la suite va girando.

La parte que hace esto algo más que una comodidad: **cuando el comando termina, Claude es re-invocado automáticamente con el resultado.** No es un "lánzalo y olvídate" donde tienes que acordarte de volver a comprobar. En cuanto los tests acaban — pasen o fallen — Claude se despierta, lee la salida y la integra en lo que venga después. Si fallaron, empieza con el arreglo; si pasaron, sigue con el siguiente paso. Nunca tuviste que vigilar la ejecución.

Así que el bucle cambia de forma. Sin tareas en background es *ejecutar → esperar → reaccionar*. Con ellas es *ejecutar → seguir trabajando → que te traigan de vuelta cuando importa*.

## Cómo se ve en la práctica

```text
> ejecuta la suite de integración y empieza a montar el nuevo endpoint

Lanzo la suite en segundo plano y empiezo con el endpoint mientras corre.

  ⏵ make test-integration  (background)

[Claude edita handler.go, añade la ruta, escribe el struct de request…]

  ✓ tarea en background terminada: make test-integration — exit 0
    142 passed, 0 failed (3m51s)

Tests en verde. Los cambios del endpoint también están — esto es lo que añadí…
```

Mientras la suite corre puedes seguir dirigiendo. `/tasks` muestra lo que sigue en marcha:

```text
> /tasks

  ⏵ make test-integration   running   2m10s
```

Y no tienes que esperar al final para echar un vistazo — puedes pedirle a Claude que consulte la salida de una tarea aún en curso si sospechas que se ha quedado colgada en un test lento. La clave es que la ejecución es *visible y direccionable*, no una caja negra de la que estás excluido hasta que devuelve.

## Background ≠ programado

Conviene trazar una línea aquí, porque es fácil confundirlos. Una tarea en background vive y muere con tu sesión: es un trabajo largo puntual que quieres solapar con otro trabajo *ahora mismo*. No es algo recurrente ni duradero. Si lo que quieres es "ejecuta esto cada 15 minutos" o "lanza esto mañana a las 9", eso es una tarea programada (`/loop`, `/schedule`) — otra herramienta con otras reglas de vida. Background es para *esta* ejecución de tests, *este* build, mientras tú avanzas con el resto del cambio.

## Impact

- **Encadenas trabajo en vez de esperarlo.** Los minutos que costaba un comando largo se gastan ahora en la siguiente edición, no en una barra de progreso.
- **El terminal deja de ser un cuello de botella.** Una suite lenta ya no serializa todo lo que tiene detrás — la ejecución y el trabajo pasan en paralelo.
- **Sin vigilancia.** Como Claude vuelve solo al terminar, no tienes que acordarte de regresar a comprobar. El resultado te encuentra a ti.

## Decisions

- **Dirígelo desde Claude, no fire-and-forget.** La razón de pedirle a Claude que ejecute el comando en background — en vez de desacoplarlo tú a mano — es la re-invocación automática al terminar. Eso es lo que convierte "un trabajo corriendo en algún sitio" en "un resultado que vuelve y sobre el que se actúa".
- **Background para lo largo y puntual; foreground para lo rápido.** Una suite de cuatro minutos o un build es el caso ideal. Un comando que devuelve en un segundo no gana nada en background — la sobrecarga no compensa.
- **Mantenlo visible.** `/tasks` y la opción de inspeccionar una tarea en curso significan que nunca adivinas si algo sigue vivo. Úsalos cuando una ejecución parezca atascada.

## Limitations

- **No sobrevive a la sesión.** Las tareas en background están atadas a la sesión viva — no las restauran `claude --resume` ni `--continue`. Si cierras la sesión, la ejecución se va con ella.
- **El resultado sigue costando contexto.** Cuando Claude vuelve con la salida, esa salida entra en la ventana de contexto como cualquier otro resultado de herramienta. Un comando que imprime miles de líneas al terminar no es gratis solo por haber corrido en background.
- **No es para trabajo recurrente.** Si te ves queriendo el mismo comando en un temporizador, te has quedado pequeño con las tareas en background — usa `/loop` o un agente programado.
- **No mandes a background comandos triviales.** Para cualquier cosa que devuelve casi al instante, el foreground normal es más simple y claro. Reserva esto para ejecuciones lo bastante largas como para que la espera duela de verdad.
