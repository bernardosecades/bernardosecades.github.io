---
title: "Claude Code: continuar sesiones y deshacer cambios"
date: 2026-04-27 11:00:00
lang: es
ref: claude-code-sessions-rewind
tags: [claude-code, workflow]
read_min: 4
excerpt_text: "¿Claude se fue por mal camino? Con /rewind deshaces las ediciones y la conversación a la vez, y reanudas cualquier sesión anterior donde la dejaste."
---

Tres pequeños trucos de Claude Code que uso todos los días.

## Continuar la última sesión

Retoma justo donde lo dejaste — mismo contexto, mismo historial.

```bash
claude --continue
# o la forma corta
claude -c
```

## Recuperar una sesión de otro día

Abre un selector con tus sesiones anteriores (de días o semanas atrás) para que puedas volver a cualquiera de ellas. El selector también busca por el nombre de la sesión, así que si [nombras las sesiones con el ticket de Jira](/claude-code-sesiones-tickets-jira/) que tienes entre manos, la correcta queda a una tecla.

```bash
claude --resume
# o
claude -r
```

## Deshacer cambios (rewind)

Si Claude se ha ido por el camino equivocado, pulsa **Esc dos veces** (`Esc Esc`) para rebobinar la conversación a un punto anterior. También puedes usar el slash command `/rewind`. Esto deshace los mensajes *y* las ediciones de ficheros hechas desde ese momento, así que no tienes que limpiar a mano. El rewind te lleva *hacia atrás* a un punto anterior; cuando prefieras conservar el hilo actual intacto y probar una alternativa en paralelo, [bifurca la sesión](/claude-code-bifurcar-sesion/) en su lugar.

Y ya está — tres atajos que ahorran mucho tiempo.
