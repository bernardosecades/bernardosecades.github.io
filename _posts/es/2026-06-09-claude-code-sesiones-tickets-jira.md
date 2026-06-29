---
title: "Claude Code: tres tareas en vuelo, cero contexto perdido — nombra las sesiones con el ticket"
date: 2026-06-09 00:00:00
lang: es
ref: claude-code-sessions-jira-tickets
tags: [claude-code, sessions, workflow, jira]
read_min: 4
excerpt_text: "Los conflictos de merge llegan tres días después de pasar a otro ticket. La sesión que conocía cada decisión de esa PR está enterrada bajo títulos autogenerados. Nombra cada sesión con su ticket de Jira y claude --resume se convierte en un índice de todo lo que tienes en vuelo."
---

Miércoles: termino ORD-2143, una política de retries para el webhook de orders, abro la PR y salto directo a BIL-987 porque el sprint no espera. Viernes: un compañero mergea antes y mi PR amanece con conflictos. Esa misma tarde, QA pregunta por qué un feature flag que terminé el lunes — mergeado, en staging, aún sin llegar a producción — se comporta raro.

Las dos respuestas viven en sesiones de Claude Code que cerré hace días. Y esto es lo que me enseña `claude --resume`:

```text
❯ refactoriza el handler para que los retri…   hace 2 días · 47 mensajes
  arregla el test del webhook que falla en CI  hace 2 días · 12 mensajes
  añade un feature flag para el flujo nuevo    hace 4 días · 31 mensajes
  refactoriza el handler y extrae la lógica…   hace 5 días · 58 mensajes
  investiga eventos duplicados en staging      hace 6 días · 23 mensajes
```

Cinco sesiones, todas plausibles, ninguna se llama ORD-2143. Los títulos autogenerados describen lo primero que escribí, no la tarea. Así que abro dos, compruebo que no son, me rindo y reconstruyo el contexto a mano: releo el ticket, el diff, mi propia descripción de la PR. Quince minutos para recuperar lo que una sesión ya sabía.

El arreglo cuesta tres segundos por tarea: nombrar la sesión con el ticket.

## Nombra la sesión al empezar la tarea

Dos formas, mismo resultado:

```bash
# al arrancar
claude -n ORD-2143
```

```text
# o dentro de la sesión, en cuanto te des cuenta de que se te olvidó
/rename ORD-2143
```

La convención: **una sesión por ticket, y el nombre es exactamente el ID del ticket**. La misma clave que ya nombra la rama (`feature/ORD-2143-webhook-retries`), el título de la PR y la tarjeta de Jira ahora nombra también la sesión. Un identificador, cuatro sistemas.

## --resume como índice de tareas

Cuando las sesiones llevan el ID del ticket, `claude --resume` deja de ser una lista y se convierte en un índice:

```bash
claude --resume        # picker: teclea ORD-2143, un match, Enter
claude -r ORD-2143     # match exacto del nombre — entra directo, sin picker
```

Los detalles que lo hacen funcionar:

- La búsqueda del picker matchea nombres de sesión, resúmenes de conversación y primeros prompts. Teclear un ID de ticket es determinista como nunca lo será "webhook" — solo una sesión de tu historial contiene `ORD-2143`.
- Si el valor que pasas no es un match exacto del nombre, el picker se abre prefiltrado con él como término de búsqueda. En cualquier caso estás a una tecla de la sesión correcta.
- Cada entrada muestra la rama git, el tiempo desde la última actividad y el número de mensajes — la rama te confirma que vas a retomar lo que toca.
- `Ctrl+A` amplía la búsqueda a todos los proyectos, `Ctrl+B` filtra por la rama git actual. Y `/resume` también funciona desde dentro de una sesión.

¿Nuevo con estos comandos? Los [básicos de continuar, recuperar y rebobinar sesiones](/claude-code-sesiones-rewind/) cubren qué hacen `--continue`, `--resume` y `/rewind`; este post es la convención de nombres montada encima.

## Workflow 1: mergeado pero sin deployar

Los deploys van por lotes; las tareas están "hechas" días antes de llegar a producción. Los bugs y las preguntas de QA llegan en asíncrono — siempre sobre la tarea en la que ya no estás.

`claude -r ORD-2143` trae de vuelta la sesión que hizo el cambio: los edge cases que discutimos, la alternativa que descartamos, por qué el handler comprueba el timestamp antes que el estado. Responder "¿por qué staging se comporta así?" cuesta una pregunta dentro de la sesión correcta, en vez de una excavación arqueológica por el diff.

## Workflow 2: conflictos de PR resueltos con el contexto original

Una sesión nueva resolviendo conflictos tiene exactamente una fuente de verdad: el diff. Re-deduce la intención a partir del código, y la intención es justo lo que los conflictos destruyen. La sesión original no tiene que adivinar — sabe qué lado del conflicto es estructural, porque lo escribió ella.

Así que cuando llegan los conflictos:

```bash
claude -r ORD-2143 --fork-session
```

`--fork-session` copia la conversación a una sesión nueva y deja la original intacta — el mismo patrón que [bifurcar una sesión para probar una alternativa](/claude-code-bifurcar-sesion/). La sesión del ticket se mantiene limpia como hilo estable; la resolución de conflictos ocurre en una rama de ella. Un matiz: los permisos aprobados con "allow for this session" no se heredan en el fork.

Un atajo relacionado: si la PR la creó Claude desde esa sesión, `claude --from-pr 4812` (número o URL de la PR) retoma directamente la sesión vinculada — el naming por ticket cubre todo lo que nunca llegó a la fase de PR.

## Impacto

- Recuperar el contexto de una tarea pasó de ~15 minutos releyendo ticket, diff y descripción de la PR a menos de un minuto: teclea el ID del ticket, Enter, pregunta.
- Las resoluciones de conflictos se mantienen coherentes con la intención original — la sesión que escribió el código resuelve sus propios conflictos.
- Escala con el trabajo en paralelo: tres tickets en vuelo u ocho, el índice funciona igual.
- Coste: tres segundos por tarea, cero infraestructura. Ni plugin, ni hook, ni script — una convención de nombres.

## Decisiones técnicas

- **El ID del ticket es el nombre completo.** No `ORD-2143-webhook-retries` — el picker ya muestra la rama y el resumen de la conversación al lado; un sufijo añade tecleo sin añadir precisión.
- **Una sesión por ticket.** La documentación no define qué hace el resume por nombre exacto cuando dos sesiones comparten nombre, así que no creo esa situación. Si un ticket necesita un segundo hilo, lo bifurco — el fork recibe su propio ID y queda agrupado bajo el original en el picker.
- **Nombrar al arrancar, no al final.** `claude -n ORD-2143` es memoria muscular justo después de abrir el ticket. Renombrar al final depende de acordarte — y las sesiones que se te olvidan son exactamente las que vas a necesitar.
- **Sube `cleanupPeriodDays`.** Las sesiones se borran al arrancar pasados 30 días por defecto. Una tarea esperando deploy puede vivir más que eso. En `~/.claude/settings.json`: `{"cleanupPeriodDays": 90}`.

## Limitaciones reales

- **Es disciplina, no automatización.** Nada fuerza la convención — no hay forma integrada de derivar el nombre de la sesión desde la rama. Sobrevive porque es barata, no porque esté garantizada.
- **Las sesiones son locales a la máquina.** `~/.claude/projects/` no se sincroniza. Cambia de portátil y el índice se queda atrás.
- **Encontrar la sesión ≠ que la sesión lo recuerde todo.** Si la conversación fue lo bastante larga para entrar en compactación, el contexto antiguo vuelve [resumido, no literal](/claude-code-ventana-contexto/). La convención encuentra la sesión correcta; no la hace infinita.
- **Los nombres duplicados son comportamiento indefinido.** Nada impide que dos sesiones compartan nombre, y qué hace entonces `claude -r <nombre>` no está documentado. La regla de una-sesión-por-ticket existe para no descubrirlo.
