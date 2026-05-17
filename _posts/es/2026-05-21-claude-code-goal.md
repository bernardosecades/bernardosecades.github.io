---
title: "Claude Code: /goal — describe la meta, no cada paso"
date: 2026-05-21 00:00:00
lang: es
ref: claude-code-goal
tags: [claude-code, goal, automation, workflow]
read_min: 4
excerpt_text: "Describes el estado final una vez y te vas. /goal convierte el prompt-a-prompt en una sola condición que Claude persigue hasta cumplirla."
unlisted: true
sitemap: false
---

El [post anterior sobre los subagentes](/claude-code-subagentes/) iba sobre delegar la exploración a otro contexto. Este va sobre algo distinto: cómo dejar de teclear *"sigue"*, *"ahora arregla el siguiente"*, *"corre los tests otra vez"* después de cada turno.

Por defecto Claude termina un turno y se calla. Si tu tarea necesita veinte iteraciones, son veinte mensajes tuyos. `/goal` cambia ese contrato: describes el estado final una vez, y Claude itera solo hasta llegar.

## Lo que hace `/goal` (esto es lo clave)

Escribes `/goal <condición>` y arranca el primer turno. Tras cada turno, un modelo pequeño (Haiku por defecto) lee el output y juzga si la condición se cumplió.

- Si **no**, Claude arranca el siguiente turno automáticamente, con la razón del evaluador como pista (*"no cumplido — TestParseDate sigue rojo"*).
- Si **sí**, el goal se limpia y Claude para.

En el statusline ves `◎ /goal active` con el tiempo transcurrido. Útil para acordarte de que está corriendo cuando vuelves del café.

## El evaluador

```text
Tú ──── /goal "go test ./... termina con exit 0"

  turno 1 ─▶ Claude edita foo.go, corre tests, 3 fallan
             evaluator: "no cumplido — 3 fallos en foo_test.go"
  turno 2 ─▶ Claude arregla asserts, corre tests, 1 falla
             evaluator: "no cumplido — TestParseDate sigue rojo"
  turno 3 ─▶ Claude arregla el parser, corre tests, todos verdes
             evaluator: "cumplido ✓"
             /goal limpio
```

Detalle importante: **el evaluador solo ve lo que Claude muestra en el transcript**. Si Claude no enseña la salida de `go test`, el evaluador no tiene nada que juzgar. Esto cambia cómo escribes el prompt — más adelante.

## `/goal` no es `/loop`, ni un prompt normal

La confusión es constante:

| Comando | El siguiente turno arranca cuando… | Para cuando… |
|---|---|---|
| Prompt normal | Tú lo lanzas | Tú decides |
| `/goal <cond>` | Termina el turno anterior | La condición se cumple |
| `/loop 5m <cmd>` | Pasa el intervalo (cada 5 min) | Tú lo paras |

Resumen: **`/goal` es condición, `/loop` es reloj**. Si quieres que Claude pare cuando algo se cumpla, `/goal`. Si quieres que repita una tarea cada X minutos pase lo que pase, `/loop`.

## Ejemplo 1: el test loop

El caso más simple, cualquier dev lo entiende:

```text
/goal go test ./... termina con exit 0, no toques nada en test/fixtures/
```

Claude edita, corre tests, falla, lee el output, edita otra vez, corre, falla, edita, pasa. Tú miras Slack mientras tanto.

La condición funciona porque tiene un **check verificable en la salida**: `go test ./...` termina con un exit code, el evaluador lo ve. *"Haz que pasen los tests"* serviría también. *"Mejora la calidad del código"* no — no hay nada que el evaluador pueda medir.

## Ejemplo 2: refactor en cascada

Tienes una lib compartida `auditlib` que usan seis microservicios Go: `orders`, `billing`, `notifier`, `gateway`, `accounts`, `reports`. Quieres añadir un parámetro de opciones a su función principal y dejar el build verde en los seis.

```text
/goal cambia la firma de auditlib.Log(ctx, event) a
auditlib.Log(ctx, event, opts LogOptions), actualiza los callers
en orders, billing, notifier, gateway, accounts y reports, y
que `go build ./...` pase en los seis repos. Para tras 15 turnos.
```

Lo que pasa, turno a turno:

- Claude edita `auditlib/log.go` y bumpea la versión.
- Repo a repo: `cd ~/projects/orders && go get auditlib@latest && go build ./...`.
- En `gateway` tropieza con un wrapper tipado que envuelve `auditlib.Log` — el build falla y el evaluador devuelve *"no cumplido — gateway no compila"*. Claude arregla el wrapper y sigue.
- Cuando los seis repos buildean, el evaluador da el ok.

Ojo al `para tras 15 turnos`: es el botón de emergencia. Sin él, una condición imposible (un repo que jamás compila por un motivo de fuera) te consume tokens hasta que lo cortas tú.

## La condición es lo que importa

Errores comunes al principio:

- **Vaga:** *"limpia el código del módulo X"* — el evaluador no puede juzgar. No la uses.
- **No verificable en el transcript:** *"la base de datos está optimizada"* — no aparece en la salida. No la uses.
- **Sin límite de turnos:** condiciones imposibles queman tokens hasta que las paras tú. Pon siempre un `para tras N turnos`.
- **Confundirlo con `/loop`:** ya lo viste arriba, pero pasa todo el rato.

Patrón fiable: *"&lt;comando concreto&gt; termina con &lt;exit / contador / estado&gt;, para tras N turnos"*.

## Por qué importa

`/goal` no hace a Claude más listo. Lo libera de tener que pedirte permiso después de cada turno.

Lo usas cuando **la meta es clara** pero **el camino no**: tests en rojo, builds rotos, refactors en cascada, colas de migraciones, lint con warnings que hay que llevar a cero.

Si la meta no se puede medir en la salida, no uses `/goal`. Usa `/plan` y guíalo tú turno a turno.
