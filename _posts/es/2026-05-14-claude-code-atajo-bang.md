---
title: "Claude Code: el atajo ! para ejecutar comandos de shell"
date: 2026-05-14 00:00:00
lang: es
ref: claude-code-bang-shortcut
tags: [claude-code, shell, workflow]
read_min: 2
excerpt_text: "Escribir ! antes de un comando lo ejecuta directamente en tu shell, sin pasar por el modelo. Más rápido, y sin gastar tokens en una vuelta que no necesitas."
---

El `!` es un atajo para ejecutar comandos de shell directamente desde el prompt de Claude Code, sin tener que pedirle a Claude que lo haga por ti.

## Cómo se usa

Escribes `!` al inicio de la línea seguido del comando:

```
!go test ./... --tags unit,integration
!go build ./...
!git status
!ls internal/
```

<video controls playsinline>
  <source src="/assets/videos/demo-shell-command.webm" type="video/webm">
</video>

El comando se ejecuta en tu shell y la salida queda visible en la conversación. Claude puede leerla y usarla como contexto para los pasos siguientes.

## Por qué ahorra tokens

Cuando escribes `!go test ./... --tags unit,integration` directamente, el comando se ejecuta sin pasar por el modelo. No hay tokens de input, ni de razonamiento, ni de tool call.

Compara con: *"Claude, corre los tests por favor"*. En ese caso el modelo:

1. Procesa tu mensaje (tokens de input)
2. Razona qué herramienta usar (tokens de output)
3. Hace la llamada al Bash tool (tokens de tool use)
4. Recibe el resultado (tokens de input)
5. Te responde algo tipo *"los tests pasaron"* (tokens de output)

Con `!go test ./... --tags unit,integration` te saltas todo eso. La salida del comando se inyecta en el contexto, pero solo eso, sin la vuelta de procesamiento del modelo.

## Cuándo usarlo

Si ya sabes exactamente qué comando quieres correr, `!` es la opción correcta. Es más rápido que describir lo que quieres y esperar a que Claude decida qué tool llamar.

Si no sabes el comando exacto, o necesitas que Claude interprete el resultado y actúe en consecuencia, tiene más sentido pedírselo directamente.
