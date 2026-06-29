---
title: "Claude Code: la memoria persistente que no sabías que tenías"
date: 2026-05-21 00:00:00
lang: es
ref: claude-code-persistent-memory
tags: [claude-code, memory, workflow]
read_min: 4
excerpt_text: "Claude toma notas sobre ti entre sesiones. Dónde viven, qué guarda, las dudas que siempre aparecen, y cómo verlas, podarlas y dirigirlas."
---

El [post anterior sobre /goal](/claude-code-goal/) iba de estado **dentro** de una sesión — Claude iterando hasta que una condición se cumple. Este va de estado **entre** sesiones: el pequeño conjunto de notas que Claude escribe por su cuenta y recarga la próxima vez que [continúas o recuperas el proyecto](/claude-code-sesiones-rewind/).

A la mayoría le sorprende que exista. Y luego les sorprende otra vez ver que son simples ficheros markdown en su propio disco.

## Qué es la memoria persistente — la clave

Es una carpeta bajo `~/.claude/projects/<slug-de-tu-cwd>/memory/` con:

- Un **índice** `MEMORY.md` — una línea por memoria, siempre cargado al inicio de cada sesión.
- Ficheros `.md` individuales — uno por memoria, con frontmatter y un cuerpo corto. Se cargan **bajo demanda** cuando una memoria parece relevante.

Claude escribe ahí solo, sin que nadie le pida nada, cuando aprende algo que merece la pena recordar. Tú también puedes decirle *"recuerda que…"* o *"olvida X"*.

Esto **no** es `CLAUDE.md`. `CLAUDE.md` es documentación que **tú** escribes para Claude (arquitectura, convenciones, comandos). La memoria persistente son notas que **Claude escribe para sí mismo** sobre ti, tus preferencias y el trabajo en curso.

## Dónde vive

```text
~/.claude/projects/
└── -home-you-projects-orders/        ← una carpeta por cwd
    └── memory/
        ├── MEMORY.md                 ← el índice, siempre cargado
        ├── user_role.md
        ├── feedback_respuestas_cortas.md
        ├── project_auditlib_rewrite.md
        └── reference_linear_ingest.md
```

El nombre de la carpeta es la ruta absoluta del cwd, con `/` convertido en `-`. Un proyecto, una carpeta. Abre Claude en otro directorio y tienes otro conjunto de memorias — no se mezclan.

## Los cuatro tipos

Cada fichero declara un `type:` en su frontmatter:

- **user** — quién eres, tu rol, lo que sabes. *"Dev de Go senior, primera vez tocando la parte de React del repo."*
- **feedback** — cómo quieres que se trabaje contigo. *"Prefiere respuestas concisas, sin resumen al final de cada turno."*
- **project** — qué está pasando ahora mismo en el trabajo. *"El rewrite de `auditlib` lo motiva un requerimiento legal/compliance, no limpieza de tech-debt. Why: un review legal flageó el almacenamiento de session tokens. How to apply: priorizar corrección de compliance sobre ergonomía."*
- **reference** — dónde mirar en sistemas externos. *"Los bugs del pipeline se trackean en el proyecto de Linear `INGEST`."*

Los cuatro tipos mapean a las cuatro preguntas que Claude tiende a hacerse al retomar una sesión: *con quién trabajo, cómo quieren que trabaje, qué hay en curso, dónde busco más.*

## Las dudas que siempre aparecen

**¿Se sincroniza entre máquinas?** No. Ficheros planos bajo `~/.claude/`. Si los quieres en otro sitio, `rsync` y listo.

**¿Se comparte entre proyectos?** No. Una carpeta por cwd. Si abres Claude en `~/projects/orders` y luego en `~/projects/billing`, las dos memorias son independientes.

**¿Claude lee todo en cada turno?** No. Solo `MEMORY.md` (el índice, truncado más allá de 200 líneas) se carga automáticamente. Los ficheros individuales se cargan cuando la descripción del índice sugiere que son relevantes.

**¿Y si guarda algo embarazoso o secreto?** Son markdown plano. `cat`, `vim`, `rm` — como cualquier fichero. Nada está cifrado, y el subsistema de memoria no sube nada a ningún sitio por su cuenta.

**¿Caduca?** No por sí solo. Las memorias `project` envejecen rápido (pasa una deadline, aterriza un refactor), pero el fichero se queda hasta que alguien — tú o Claude — lo borre. Vale la pena darles un vistazo cada pocas semanas.

## Trucos para verla, manejarla y optimizarla

**Verla.** Dos formas:

```bash
ls ~/.claude/projects/-home-you-projects-orders/memory/
cat ~/.claude/projects/-home-you-projects-orders/memory/MEMORY.md
```

O pregúntale: *"¿qué recuerdas de mí en este proyecto?"*. Claude lee el índice y te lo cuenta.

**Forzar que guarde.** *"Recuerda que prefiero Postgres sobre MySQL para servicios nuevos por el JSONB y los índices parciales."* Claude elige tipo, escribe el fichero, actualiza el índice.

**Forzar que olvide.** *"Olvida la memoria sobre el rewrite de `auditlib` motivado por legal — ya no es cierto."* Claude busca el fichero, lo borra, quita la línea del índice. Si no encuentra a qué te refieres, abre la carpeta y bórralo a mano — son ficheros.

**Limpia las memorias de tipo `project` cada cierto tiempo.** Por diseño llevan una línea `Why:` con el motivo. Échales un ojo: si ese motivo ya no se sostiene (deadline pasada, decisión revertida, persona que se fue), bórralas. Una memoria `project` desactualizada mentirá con confianza en la siguiente sesión.

**Mantén el índice ligero.** `MEMORY.md` se trunca pasadas las 200 líneas. Una memoria = una línea, bajo ~150 caracteres: `- [Título](fichero.md) — gancho de una línea`. Si crece más, consolida duplicados y descarta lo muerto.

**Edita a mano cuando vaya más rápido.** Los ficheros son markdown con frontmatter. Si ves algo mal, arréglalo directo. Dos cosas a tener en cuenta: que `name:` del fichero case con la entrada del índice, y que `type:` sea válido (`user`, `feedback`, `project`, `reference`).

**¿Trabajas entre repos hermanos?** La memoria se indexa por cwd. Si abres Claude en `~/projects/orders` y en `~/projects/billing` por separado, ninguno sabe lo que aprendió el otro. El [setup de workspace multi-repo](/claude-code-workspace-multi-repo/) lo esquiva: abres Claude un nivel más arriba y la carpeta de memoria se comparte para todos los repos por debajo.

## Por qué esto importa

La memoria no hace a Claude más listo. Te ahorra explicarte cada lunes — las rarezas del proyecto, tus preferencias, qué hay en curso, dónde están las cosas.

Pero es estado mutable que tú no mantienes activamente. Lo que Claude escribió hace dos meses puede engañarte hoy. Léela de vez en cuando y limpia lo que ya no aplica, como harías con un `.bashrc` que olvidaste que existía. La [ventana de contexto](/claude-code-ventana-contexto/) es el presupuesto de una sesión; la memoria persistente es el presupuesto entre todas — las dos merecen el mismo cuidado.
