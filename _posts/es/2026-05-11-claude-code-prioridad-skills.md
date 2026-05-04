---
title: "Claude Code: ¿qué skill gana cuando hay nombres duplicados?"
date: 2026-05-11 00:00:00
lang: es
ref: claude-code-skill-priority
tags: [claude-code, skills, workflow]
read_min: 2
excerpt_text: "Clonas un repositorio que incluye sus propios skills y de repente tienes dos con el mismo nombre. Hay un orden de prioridad claro que decide cuál se ejecuta."
---

Clonas un repositorio que incluye sus propios skills y de repente tienes dos con el mismo nombre. ¿Cuál se ejecuta? Claude Code tiene un orden de prioridad fijo:

1. **Enterprise** * — configuración gestionada, máxima prioridad
2. **Personal** — tu directorio home (`~/.claude/skills`)
3. **Project** — el directorio `.claude/skills` dentro del repositorio
4. **Plugins** — plugins instalados, mínima prioridad

Si tu empresa distribuye un skill enterprise llamado `code-review` y tú tienes uno personal con el mismo nombre, siempre gana el de la empresa.

> (*) Los skills Enterprise se aplican cuando usas Claude Code con una **cuenta corporativa** — normalmente cuando tu organización gestiona Claude a través de la oferta enterprise de Anthropic y distribuye la configuración de forma centralizada. Si usas una cuenta personal, no hay ninguna capa enterprise en el stack.

## Ejemplo concreto

Tienes:

```
~/.claude/skills/code-review       ← Personal
repo/.claude/skills/code-review    ← Project
```

Cuando Claude intenta usar `code-review`:

```
Revisa Enterprise → no hay
Revisa Personal   → sí hay ✔ STOP
                    nunca llega a mirar el de Project
```

**Resultado:** ejecuta tu skill personal, y el del repo queda completamente ignorado.

## Qué significa esto en la práctica

Los skills personales tienen prioridad sobre los del proyecto. Esto es útil cuando quieres añadir tus propios atajos encima de lo que ofrece un repositorio sin tocar el repo.

Los skills del proyecto tienen prioridad sobre los plugins. Los repositorios pueden distribuir flujos de trabajo específicos que se imponen sobre cualquier cosa que hayas instalado desde el marketplace.

La capa enterprise existe para que las organizaciones puedan establecer estándares que los individuos no puedan sobrescribir accidentalmente, aunque añadan un skill personal o de proyecto con el mismo nombre.

## Cómo evitar colisiones

Usa nombres descriptivos en lugar de genéricos. `review` es una colisión esperando ocurrir. `frontend-review`, `backend-review` o `api-review` son suficientemente específicos como para que la probabilidad de un nombre duplicado accidental sea casi nula.

El orden de prioridad es un mecanismo de resolución, no una invitación a depender de él. Los nombres claros son mejor solución que los sobreescritos ingeniosos.
