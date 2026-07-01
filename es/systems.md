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
{%- assign _now = site.time | date: '%s' | plus: 0 -%}
{%- assign _pub = "" -%}
{%- for _p in site.posts -%}{%- if _p.lang == page.lang -%}{%- assign _pts = _p.date | date: '%s' | plus: 0 -%}{%- if _pts <= _now -%}{%- assign _pub = _pub | append: _p.url | append: "|" -%}{%- endif -%}{%- endif -%}{%- endfor -%}

# Sistemas que he construido

Soy ingeniero backend con la IA entretejida en el flujo diario — debugging, revisión de código, decisiones de modelo y diseño, orquestación agentiva — y construyo los sistemas que hacen eso repetible en un equipo: marketplaces de plugins, workspaces multi-repo, pipelines de review. Todo empezó dentro de una organización de ingeniería real con un problema real, no como exploración de una feature.

Abajo está el set de trabajo — las prácticas en las que me apoyo y los sistemas que he puesto en producción — agrupado por lo que es. Cada enlace lleva a un post que nombra el problema resuelto, las decisiones tomadas y las limitaciones reales.

## Sistemas de coding con IA en práctica

- **Marketplace de plugins por equipos** — un repo Git, una carpeta de plugin por equipo, más un plugin `common` compartido y promovido de abajo arriba. Sustituye tres formas incompatibles de distribuir skills de Claude entre equipos. → [Cómo un marketplace por equipos acabó con la duplicación de skills en Claude](/claude-code-marketplace-equipos/)

- **Workspace multi-repo con Claude** — abre Claude un nivel por encima de tus repos para que un único contexto, una única carpeta de memoria y un único alcance de permisos cubran ~25 servicios a la vez. → [Workspace multi-repo](/claude-code-workspace-multi-repo/)

- **Depurar un bug distribuido con IA** — dale al modelo la traza, el camino real del código y lo que has descartado, y pídele hipótesis ordenadas y la confirmación más barata — no "arréglalo".{% if _pub contains "/depurar-con-ia-dale-las-pruebas/" %} → [Depurar con IA](/depurar-con-ia-dale-las-pruebas/){% endif %}

- **Diseñar un servicio con la IA como interlocutor** — trae tu propio diseño y haz que el modelo lo ataque (por dónde se filtra el límite, qué se rompe bajo fallo parcial), que defienda los dos lados de un trade-off, y quédate el lápiz.{% if _pub contains "/disenar-un-servicio-con-ia-como-interlocutor/" %} → [Diseñar un servicio](/disenar-un-servicio-con-ia-como-interlocutor/){% endif %}

- **Elegir modelo como decisión de ingeniería** — empareja el tier y el dial de esfuerzo con la tarea en vez de tirar del modelo más grande; decide sobre un set de evaluación real, no por instinto.{% if _pub contains "/elegir-el-modelo-segun-la-tarea/" %} → [Elegir modelo](/elegir-el-modelo-segun-la-tarea/){% endif %}

## Claude Code para automatización real

- **Patrón de bifurcar sesión** — ramificar la conversación actual de Claude para probar una alternativa sin perder el estado en curso de la original. → [Bifurcar sesión](/claude-code-bifurcar-sesion/)

- **El atajo `!` como entrada determinista** — meter la salida de un comando en la conversación sin pasar por el modelo cuando necesitas los bytes exactos, no una paráfrasis. → [Atajo bang](/claude-code-atajo-bang/)

- **Alcance vs. permisos, separados** — el `cwd` decide lo que Claude *ve*, `.claude/settings.json` decide lo que Claude *hace*. Trátalos como dos palancas independientes en cada proyecto. → [Alcance vs. permisos](/claude-code-alcance-vs-permisos/)

## Workflows agentivos en producción

- **Exploración con subagentes para mantener el contexto principal limpio** — mover el trabajo pesado de grep/read a un subagente `Explore` para que la sesión principal pague ~1k por un resumen en vez de ~40k por un transcript. → [Los subagentes tienen su propia ventana de contexto](/claude-code-subagentes/)

- **Bucle `/goal` para iteración autónoma** — dejar de teclear "sigue" entre turnos; declarar una condición y dejar que Claude itere hasta cumplirla. → [El bucle `/goal`](/claude-code-goal/)

- **Memoria persistente como estado entre sesiones** — ficheros markdown que Claude escribe solo y recarga la próxima vez, indexados por `cwd`. Úsala con intención, púdala como un `.bashrc`. → [Memoria persistente](/claude-code-memoria-persistente/)

- **Fan-out multi-agente, con el coste honesto** — cuándo N agentes paralelos ganan a uno (trabajo independiente y ancho) y cuándo solo multiplican la factura; descompón por independencia real y presupuesta la fusión.{% if _pub contains "/orquestacion-multi-agente-cuando-merece-la-pena/" %} → [Cuándo el fan-out se paga solo](/orquestacion-multi-agente-cuando-merece-la-pena/){% endif %}

## Casos de fallo y lecciones aprendidas

- **Prioridad de skills y sobrescrituras silenciosas** — una skill personal gana en silencio sobre una skill de proyecto o plugin con el mismo nombre. Nombres distintivos, no sobrescrituras ingeniosas. → [¿Qué skill gana cuando los nombres colisionan?](/claude-code-prioridad-skills/)

- **La ventana de contexto como presupuesto, no como capacidad** — una vez empieza la compactación, el modelo razona sobre un resumen con pérdida. Planifica la sesión para no llegar ahí por accidente. → [Ventana de contexto](/claude-code-ventana-contexto/)

- **Rewind de sesión para recuperación barata** — unidad pequeña de undo sobre una sesión de Claude para que un turno malo no envenene el resto. → [Sesiones rewind](/claude-code-sesiones-rewind/)

- **Revisar el código de IA por sus modos de fallo** — el código de IA es *plausiblemente* incorrecto, no obviamente; una checklist para los traspasos que no ve — caminos de error, radio de impacto fuera del diff, concurrencia, scope creep.{% if _pub contains "/revisar-codigo-generado-por-ia-antes-de-confiar/" %} → [Revisar código generado por IA](/revisar-codigo-generado-por-ia-antes-de-confiar/){% endif %}

- **Tests en verde que no prueban nada** — los tests escritos por IA pasan comprobando el mock o fijando el bug como esperado; escribe el test que falla primero y lee cada aserción con el criterio ¿puede fallar?{% if _pub contains "/testing-con-ia-verde-no-es-probado/" %} → [Testing con IA](/testing-con-ia-verde-no-es-probado/){% endif %}

- **El día que la IA casi mete un bug en prod** — un diff limpio, una suite en verde y una explicación segura son una sola fuente coincidiendo consigo misma; la comprobación independiente tiene que venir de fuera del bucle del modelo.{% if _pub contains "/el-dia-que-la-ia-casi-mete-un-bug-en-prod/" %} → [Qué cambié](/el-dia-que-la-ia-casi-mete-un-bug-en-prod/){% endif %}

</article>
</div>
