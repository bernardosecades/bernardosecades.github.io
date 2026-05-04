---
title: "Claude Code: nombra una sesión y bifúrcala para probar otro camino"
date: 2026-05-04 00:00:00
lang: es
ref: claude-code-fork-session
tags: [claude-code, workflow]
read_min: 2
excerpt_text: "Renombra una sesión de Claude Code con /rename y luego bifúrcala en una copia independiente para probar otro enfoque sin perder la original."
---

Un pequeño flujo de dos pasos que uso cuando quiero probar otro enfoque en una sesión sin perder el hilo original. La idea es muy parecida a un `git branch`: parto del mismo punto, pero en una rama aparte.

## 1. Pon nombre a la sesión

Dentro de la sesión, dale un nombre fácil de recordar:

```text
/rename main-context
```

A partir de ahora puedes referirte a la sesión por su nombre en vez de por su ID.

## 2. Bifúrcala desde fuera

Desde la terminal, recupera esa sesión por nombre y bifúrcala:

```bash
claude --resume main-context --fork-session
```

Esto copia el historial en una **nueva sesión** y te deja dentro de ella. La sesión original `main-context` queda intacta — puedes volver a ella cuando quieras como tu hilo "estable".

## Por qué lo hago

- Trato una sesión con nombre como mi contexto principal de trabajo, y la bifurco cada vez que quiero explorar un camino lateral.
- Si el camino lateral no cuaja, simplemente recupero la original y vuelvo a intentarlo desde un estado limpio.
