---
layout: default
title: "Sistemas que he construido"
description: "Automatizaciones, herramientas y workflows de desarrollo aumentados con IA que he puesto en producción — no exploraciones de features."
permalink: /es/systems/
lang: es
nav: systems
systems_ref: true
---
<div class="container">
<article class="prose" markdown="1">

# Sistemas que he construido

Construyo sistemas de desarrollo aumentados con IA: marketplaces de plugins, workspaces multi-repo, pipelines de review, workflows agentivos. Cada uno empezó dentro de una organización de ingeniería real con un problema real, no como exploración de una feature.

Abajo está el set actual, agrupado por tipo de sistema. Cada enlace lleva a un post que nombra el problema resuelto, las decisiones técnicas tomadas y las limitaciones reales.

## Sistemas de coding con IA en práctica

- **Marketplace de plugins por equipos** — un repo Git, una carpeta de plugin por equipo, más un plugin `common` compartido y promovido de abajo arriba. Sustituye tres formas incompatibles de distribuir skills de Claude entre equipos. → [Cómo un marketplace por equipos acabó con la duplicación de skills en Claude](/claude-code-marketplace-equipos/)

- **Workspace multi-repo con Claude** — abre Claude un nivel por encima de tus repos para que un único contexto, una única carpeta de memoria y un único alcance de permisos cubran ~25 servicios a la vez. → [Workspace multi-repo](/claude-code-workspace-multi-repo/)

## Claude Code para automatización real

- **Patrón de bifurcar sesión** — ramificar la conversación actual de Claude para probar una alternativa sin perder el estado en curso de la original. → [Bifurcar sesión](/claude-code-bifurcar-sesion/)

- **El atajo `!` como entrada determinista** — meter la salida de un comando en la conversación sin pasar por el modelo cuando necesitas los bytes exactos, no una paráfrasis. → [Atajo bang](/claude-code-atajo-bang/)

- **Alcance vs. permisos, separados** — el `cwd` decide lo que Claude *ve*, `.claude/settings.json` decide lo que Claude *hace*. Trátalos como dos palancas independientes en cada proyecto. → [Alcance vs. permisos](/claude-code-alcance-vs-permisos/)

## Workflows agentivos en producción

- **Exploración con subagentes para mantener el contexto principal limpio** — mover el trabajo pesado de grep/read a un subagente `Explore` para que la sesión principal pague ~1k por un resumen en vez de ~40k por un transcript. → [Los subagentes tienen su propia ventana de contexto](/claude-code-subagentes/)

- **Bucle `/goal` para iteración autónoma** — dejar de teclear "sigue" entre turnos; declarar una condición y dejar que Claude itere hasta cumplirla. → [El bucle `/goal`](/claude-code-goal/)

- **Memoria persistente como estado entre sesiones** — ficheros markdown que Claude escribe solo y recarga la próxima vez, indexados por `cwd`. Úsala con intención, púdala como un `.bashrc`. → [Memoria persistente](/claude-code-memoria-persistente/)

## Casos de fallo y lecciones aprendidas

- **Prioridad de skills y sobrescrituras silenciosas** — una skill personal gana en silencio sobre una skill de proyecto o plugin con el mismo nombre. Nombres distintivos, no sobrescrituras ingeniosas. → [¿Qué skill gana cuando los nombres colisionan?](/claude-code-prioridad-skills/)

- **La ventana de contexto como presupuesto, no como capacidad** — una vez empieza la compactación, el modelo razona sobre un resumen con pérdida. Planifica la sesión para no llegar ahí por accidente. → [Ventana de contexto](/claude-code-ventana-contexto/)

- **Rewind de sesión para recuperación barata** — unidad pequeña de undo sobre una sesión de Claude para que un turno malo no envenene el resto. → [Sesiones rewind](/claude-code-sesiones-rewind/)

</article>
</div>
