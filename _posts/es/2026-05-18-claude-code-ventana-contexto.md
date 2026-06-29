---
title: "Claude Code: cómo no quedarme sin ventana de contexto"
date: 2026-05-18 00:00:00
lang: es
ref: claude-code-context-window
tags: [claude-code, context, workflow]
read_min: 4
excerpt_text: "La ventana de contexto es un recurso limitado. Cuatro hábitos que uso a diario para mantenerla sana: /clear y /compact, subagentes, @path y plan mode."
---

La ventana de contexto de Claude Code es finita. Cada mensaje, cada tool call, cada fichero leído ocupa un trozo de ella. Cuando se llena demasiado pasan dos cosas: el modelo empieza a olvidar lo del principio de la sesión, y los costes por turno se disparan porque procesar 200k tokens cada vez no sale gratis.

Cuatro hábitos que uso a diario para mantenerla sana.

## /clear, /compact y auto-compact

Tres herramientas para liberar contexto, pensadas para situaciones distintas.

`/clear` borra todo y empieza una sesión nueva. Lo uso cuando cambio de tarea: terminé el bug de pagos, ahora voy a tocar logs, no necesito que la conversación arrastre lo anterior.

`/compact` pide al modelo que resuma el contexto y conserva sólo ese resumen. Útil cuando llevo una hora trabajando, los 12 ficheros que leí al principio ya no aportan, pero sí quiero conservar las decisiones que tomamos por el camino.

El auto-compact se dispara solo cuando rozas el ~95% de la ventana. Funciona, pero te puede pillar a mitad de tarea: el resumen automático puede perder matices que estabas a punto de usar. Si veo el indicador subiendo del 70% y la tarea aún no está cerrada, lanzo `/compact` antes de que el modelo decida por mí.

## Subagentes para no contaminar la sesión principal

Cuando una tarea empieza con *"necesito entender cómo X habla con Y"*, esa fase de exploración se come 30-50k tokens fácilmente: greps, finds, lectura de varios ficheros para reconstruir el flujo.

En lugar de hacer eso en mi sesión principal, lanzo un subagente `Explore`. El subagente tiene **su propia ventana**: gasta sus tokens explorando y devuelve un resumen de medio kilobyte. Mi sesión sólo recibe ese resumen, no los doce ficheros completos.

Empiezo la tarea con el contexto principal casi vacío, ya con el mapa mental que necesitaba, y la ventana queda libre para el trabajo de verdad — escribir, revisar, iterar.

## @path: dale el archivo, no le pidas que lo busque

Hay dos formas de meter un fichero en el contexto: pedirle a Claude *"léete los tests para entender qué hace X"* (Claude lista, abre, a veces se equivoca de fichero), o escribirlo tú: `@path/al/test.go`.

La segunda es directa. El fichero se inyecta y listo. Sin búsquedas, sin tool calls intermedios, sin ida y vuelta.

Lo mismo aplica al [atajo `!`](/claude-code-atajo-bang/): `!cat config.yaml` mete la salida del comando en el contexto sin pasar por el modelo.

Que Claude **busque y lea diez ficheros** para encontrar el correcto cuesta más que darle los dos relevantes desde el principio.

## Plan mode y cuándo abrir sesión nueva

Plan mode (Shift+Tab dos veces) hace que Claude diseñe sin ejecutar. No hay tool calls, no hay outputs largos, no hay 800 líneas de log volcadas en la conversación. Sólo razonamiento y un plan al final. El diseño cuesta poco contexto, y al ejecutar ya tengo claro qué tocar.

Para decidir si seguir o arrancar de cero:

- **Tarea nueva, mismo proyecto** → `/clear`. Conservo CLAUDE.md y el setup local, suelto la conversación.
- **Quiero explorar una alternativa sin perder lo actual** → [bifurcar sesión](/claude-code-bifurcar-sesion/). Dos ramas en paralelo.
- **Claude se ha ido por el camino equivocado en esta sesión** → [rebobinar a un punto anterior](/claude-code-sesiones-rewind/). Deshace los mensajes *y* las ediciones de ficheros desde ahí, sin limpiar a mano.
- **Tarea completamente distinta** → sesión nueva desde cero, sin más.

La fricción de empezar limpio es mucho menor que la de pelear con una sesión sobrecargada.

## Por qué importa

La ventana es un **presupuesto**, no un infinito. Cada lectura, cada tool call, cada subagente tiene coste, y la calidad de las respuestas baja cuando se acerca al techo.

Si te das cuenta tarde, `/compact` te salva. Si te das cuenta a tiempo, ni siquiera lo necesitas.
