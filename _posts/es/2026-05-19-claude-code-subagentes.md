---
title: "Claude Code: los subagentes tienen su propia ventana de contexto"
date: 2026-05-19 00:00:00
lang: es
ref: claude-code-subagents
tags: [claude-code, subagents, context, workflow]
read_min: 4
excerpt_text: "Un subagente no es un ayudante que comparte tu sesión: es un Claude con su propia ventana de contexto. Lo usas para absorber la fase de exploración y que tu sesión principal se quede con el resumen, no con el ruido."
video:
  src: /assets/videos/demo_subagents.webm
  thumbnail: /assets/videos/demo_subagents.jpg
  duration: PT2M5S
  width: 1917
  height: 1038
---

El [post anterior sobre la ventana de contexto](/claude-code-ventana-contexto/) nombró a los subagentes como uno de los cuatro hábitos: *"tiene su propia ventana, devuelve un resumen"*. La mayoría de la gente con la que hablo asiente y luego sigue tratándolos como si fueran una segunda pestaña de la misma conversación — la misma memoria, los mismos archivos, los mismos apuntes. No lo son.

Un subagente es una invocación separada de Claude, con su propio contexto, que ejecuta el prompt que le pasas y devuelve **un único mensaje de texto final**. Nada más cruza la frontera.

## Su propia ventana de contexto (esto es lo clave)

El subagente no ve tu conversación. No ve los archivos que inyectaste con `@`. No ve las decisiones que discutiste en el chat. Solo ve el prompt que le pasas.

Y en la vuelta, la sesión principal recibe **un único bloque de texto** — el último mensaje del subagente. Sin transcript de tool calls, sin contenidos de archivos, sin razonamiento intermedio.

```text
Main session                Subagent (Explore)
┌──────────────┐            ┌──────────────────────┐
│ context: 40k │            │ context: 0 → 80k     │
│              │  prompt    │                      │
│              │ ─────────▶ │ greps, reads, finds, │
│              │            │ reads more files…    │
│              │ ◀───────── │                      │
│ context: 41k │  summary   │ context: 80k (gone)  │
└──────────────┘  (~1k)     └──────────────────────┘
                                discarded on exit
```

El main paga el prompt (~1k) y el resumen (~1k). Los 80k gastados grepeando viven y mueren dentro del subagente.

## Built-in vs custom

`Explore` y `general-purpose` vienen con Claude Code. `Explore` es read-only y está afinado para navegar código — al que recurro el 90% del tiempo.

Un subagente **custom** es un archivo markdown en `.claude/agents/<nombre>.md` con frontmatter y un cuerpo que es su system prompt:

```yaml
---
name: auditlib-callsites
description: Finds call sites of an auditlib function across the
  microservices. Returns repo / file:line / snippet.
tools: [Bash, Read, Grep, Glob]
model: claude-sonnet-4-6
---
```

Merece definir uno cuando vas a teclear el mismo prompt de orquestación cada semana, cuando quieres restringir las herramientas a read-only, o cuando quieres un modelo más barato para tareas mecánicas. No para un one-off.

Un subagente custom en `.claude/agents/` **no es** lo mismo que un skill en `.claude/skills/`. Un skill es un bundle de instrucciones reutilizable que carga la sesión principal. Un subagente es un Claude separado con su propia ventana. Un skill puede lanzar un subagente; no son lo mismo.

## Cómo se invocan

Tres modos:

- **Automático** — en mitad de una tarea, el modelo del main decide solo: *"voy a lanzar un Explore subagent para mapear esto"*. Visible en el transcript, interrumpible.
- **Explícito** — tú lo pides: *"usa el subagent `auditlib-callsites` para encontrar todos los callers de X"*. Claude elige el correcto cuando la `description` del agente encaja.
- **Anidado** — un orquestador custom puede lanzar sus propios subagentes en paralelo, útil cuando si no serializarías un fan-out.

El contrato es siempre el mismo: prompt entra, un mensaje de texto final sale. Esa es toda la interfaz.

<video controls playsinline preload="none" poster="/assets/videos/demo_subagents.jpg">
  <source src="/assets/videos/demo_subagents.webm" type="video/webm">
</video>

## Dos ejemplos reales del workspace

Trabajo sobre ~25 microservicios Go en `~/projects/` (el [setup del workspace multi-repo](/claude-code-workspace-multi-repo/)). Dos usos del día a día.

**¿Quién llama a `auditlib.Submit()`?** Tecleo en el main:

> Lanza un Explore subagent en `~/projects/`. Busca todos los call sites de `auditlib.Submit` en los microservicios (orders, billing, notifier, gateway, accounts, reports, etc.). Para cada hit devuelve: repo, file:line, la función que llama, y una línea de contexto.

Lo que aterriza en la sesión principal:

```text
4 call sites:
- orders    internal/order/engine.go:142     func (e *Engine) finalize
- billing   internal/worker/process.go:88    func (w *Worker) onCharge
- notifier  internal/usecase/dispatch.go:201 func (s *Service) send
- gateway   internal/handler/submit.go:55    func (h *Handler) Post

Los cuatro pasan un *auditlib.Event construido localmente;
ninguno reutiliza un builder compartido. orders es el único que
envuelve la llamada en un retry.
```

Orden de magnitud: ~25k tokens quemados grepeando y leyendo dentro del subagente; mi main creció ~200. Sin él, esos 25k contaminarían mi contexto el resto de la sesión.

**Impacto de cambiar la firma de `httpx.Foo()`.** Tecleo:

> Quiero cambiar `httpx.Foo(ctx, id string)` a `httpx.Foo(ctx, id string, opts FooOptions)`. Usa el `Explore` subagent en `~/projects/` para listarme todos los consumidores agrupados por repo. Para cada uno dime si compilaría con `opts = FooOptions{}` por defecto, o si el caller expone la forma actual.

Lo que vuelve:

```text
9 callers en 6 repos. Todas llamadas directas.

Seguros con opts por defecto:
- orders, billing, notifier, gateway, accounts (8 sites en total)

Cuidado:
- reports  internal/legacy/foo_wrapper.go:18
  Envuelve httpx.Foo detrás de un func tipado ReportFoo, expuesto
  en la API pública. Cambiar httpx.Foo cascada a los callers de
  ReportFoo.
```

~40k tokens dentro del subagente recorriendo 6 repos; mi main recibió 400 tokens de veredicto estructurado — y la pista del break en cascada en `reports`. Eso que no pillaría grepeando a mano a las 11 de la noche.

## Por qué importa

El subagente es una herramienta de **presupuesto de contexto**, no un "Claude más inteligente". El mismo modelo detrás (o uno más pequeño). Lo que cambia es qué cruza la frontera de vuelta a ti.

Regla del pulgar: si una sub-tarea va a leer más de cinco archivos o correr más de tres greps, lanza uno. El `Explore` built-in cubre casi todo. Agentes custom solo cuando si no repetirías el mismo prompt de orquestación cada semana.

El coste de lanzar uno es pequeño. El coste de dejar que esa exploración manche el contexto del main dura el resto de la sesión.
