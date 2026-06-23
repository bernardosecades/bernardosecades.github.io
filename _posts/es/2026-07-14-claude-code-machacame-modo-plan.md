---
title: "Claude Code: machaca tu plan en plan mode antes de tocar código"
date: 2026-07-14 00:00:00
lang: es
ref: claude-code-grill-me
tags: [claude-code, plan-mode, skills, workflow]
read_min: 4
excerpt_text: "Un plan se lee impecable hasta que empiezas a implementarlo — entonces las decisiones que nadie resolvió van apareciendo una a una, en el peor momento para cambiarlas. La skill grill-me las saca antes: te interroga sin tregua, una pregunta cada vez, recomendando una respuesta en cada una, y explora el código en vez de preguntarte cuando puede responder sola."
---

Un plan se lee impecable. Los pasos están en orden, los ficheros nombrados, parece listo. Entonces empiezo a implementarlo y empiezan las preguntas: ¿qué pasa en este edge case? ¿Esta dependencia o aquella? ¿Este formato o el otro? Ninguna estaba en el plan — no porque el plan fuera chapucero, sino porque escribir en prosa no te obliga a resolver cada rama. Las que quedaron sin resolver las descubres a mitad del código, que es justo cuando cambiar de idea sale más caro.

La skill a la que recurro para que eso no pase es `grill-me`, y se lleva casi de maravilla con plan mode.

## La idea: que te interroguen antes de escribir código

`grill-me` es una skill diminuta cuyo único trabajo es convertir a Claude en un entrevistador implacable sobre un plan o un diseño. Recorre el árbol de decisiones rama a rama, resolviendo las dependencias entre decisiones una a una y — esta es la parte que la hace usable — **recomienda una respuesta en cada pregunta que hace.** Así no te quedas mirando un "¿qué quieres?" abierto; reaccionas a una propuesta concreta, que es mucho más rápido.

Dos reglas de sus instrucciones marcan la diferencia:

- **Una pregunta cada vez.** Nada de un muro de quince viñetas que triar. Cada respuesta puede cambiar lo que pregunta a continuación, así que las preguntas siguen siendo relevantes y las dependencias te quedan secuenciadas.
- **Explorar el código en vez de preguntar, cuando puede.** Si una pregunta se contesta leyendo el repo, va y lee el repo en lugar de convertirte a ti en su servicio de consultas.

Esa es la skill entera. Sin script, sin magia — solo un prompt afilado.

## Por qué encaja con plan mode

Plan mode ya pone a Claude en modo solo-lectura y exploración: puede leer ficheros y buscar en el código, pero no puede editar nada hasta que apruebas. Esa es justo la postura que `grill-me` quiere. En vez de que Claude redacte un plan en silencio y te lo pase para que lo firmes, machaca el diseño *contigo* — y cuando una pregunta se zanja haciendo grep en el código, lo hace sin más, porque ya tiene acceso de lectura y nada que escribir.

El resultado es un plan que llega con sus ramas ya resueltas, no uno que parece terminado y se deshilacha en el primer commit.

## Cómo se instala

Necesitas Claude Code con plan mode (cambias de modo con **Shift+Tab**). A partir de ahí hay dos formas de conseguir la skill.

### La vía rápida: el instalador

`grill-me` nació en la colección de skills de Matt Pocock — es la que [se hizo viral](https://www.aihero.dev/my-grill-me-skill-has-gone-viral). La forma más rápida de tener la original es su instalador de comunidad:

```bash
npx skills@latest add mattpocock/skills
```

Abre un selector: eliges `grill-me` (y su hermana `grill-with-docs` si la quieres) y en qué agente instalarla. La skill acaba en `~/.claude/skills/`. Ojo: esto trae la versión completa y original de la skill, más rica que la versión recortada de abajo. El código está en [github.com/mattpocock/skills](https://github.com/mattpocock/skills).

### O constrúyela tú

Si prefieres un único fichero autocontenido y sin dependencias — o solo quieres ver lo poco que hace falta — es una skill personal que creas una vez y usas en todos los proyectos:

```bash
mkdir -p ~/.claude/skills/grill-me
```

`~/.claude/skills/grill-me/SKILL.md`:

```markdown
---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.
```

Esa versión mínima es mi recorte propio — lo justo para reproducir el comportamiento de este post. Ponla bajo el `.claude/skills/` de un proyecto si prefieres acotarla a un repo. Para dispararla, de cualquiera de las dos formas: entra en plan mode, esboza lo que quieres construir y di **"grill me."** Es el `description` lo que permite a Claude reconocerla a partir de esa frase.

## En la práctica

Una sesión real tiene esta pinta — primero el plan, luego le cedes el volante:

```text
> plan: añadir reintentos automáticos para notificaciones salientes fallidas
> grill me

Q1 — ¿Dónde debería vivir el estado de los reintentos? ¿En memoria, o
persistido en una tabla `notification_attempts`?
Recomendado: persistir. Una cola en memoria pierde todo en un deploy, y
una notificación perdida es justo lo que no te puedes permitir aquí.

> de acuerdo, persístelo

Q2 — ¿Cuántos intentos antes de rendirse, y con qué backoff?
Recomendado: 5 intentos, backoff exponencial con tope en 1h — suficiente
para aguantar un bache pasajero sin martillear un servicio que está caído
todo el día.

> ponlo en 6, tope en 30m

Q3 — ¿Los reintentos tienen que conservar el orden por destinatario, o
cada notificación es independiente?
(mirando el código…) El notifier ya envía cada notificación en su propia
goroutine sin garantía de orden, así que los reintentos no añaden ninguna
restricción nueva — trataré cada intento como independiente.

> correcto, independiente
```

Tres preguntas, tres ramas cerradas — y la Q3 la respondió leyendo el código, no haciéndome recitar cómo funciona el emisor. Para cuando el plan está escrito, la tabla, la política de backoff y la suposición de orden están todas decididas, por escrito, antes de que exista una sola línea.

## Impacto

- **Las decisiones implícitas afloran antes de implementar, no durante.** Los edge cases que antes descubría a mitad del código aparecen como preguntas cuando cambiar la respuesta todavía no cuesta nada.
- **Menos retrabajo.** El plan que se aprueba ya tiene sus ramas resueltas, así que el primer commit no lo contradice de inmediato.
- **Una entrevista más rápida que planificar en abierto.** Reaccionar a una respuesta recomendada es más ágil que redactarla de cero — la mayoría de las veces digo "sí" y de vez en cuando "no, haz X."

## Decisiones

- **Una pregunta cada vez, a propósito.** Un lote de quince preguntas es una tarea de triaje; una secuencia deja que cada respuesta guíe la siguiente y fuerza las dependencias a ordenarse.
- **Recomienda él, decido yo.** La respuesta recomendada me deja avanzar a la velocidad del acuerdo y gastar esfuerzo solo donde no coincido — sin renunciar a la última palabra.
- **Explorar antes que preguntar.** Todo lo que el código puede contestar, lo contesta desde el código. El acceso de lectura de plan mode lo hace gratis, y deja las preguntas para lo que solo sé yo.
- **Machacar en plan mode, no después.** El momento más barato para resolver una rama es antes de que cualquier código se comprometa con ella.

## Limitaciones

- **Pone a prueba un plan; no lo inventa.** Sigues necesitando una idea aproximada de lo que construyes — `grill-me` afina la dirección, no te la da.
- **La calidad escala con el contexto.** Cuanto más código relevante haya que explorar, mejores son las preguntas. En un repo desde cero con poco que leer, se apoya más en ti.
- **Alarga la planificación.** Ese es el trato: más minutos por delante para recomprar las reescrituras a mitad de implementación. Para un arreglo de una línea es matar moscas a cañonazos.
- **No sale código.** Es una conversación de diseño. La salida es un plan resuelto — construirlo sigue tocándote a ti (o a la siguiente fase).
