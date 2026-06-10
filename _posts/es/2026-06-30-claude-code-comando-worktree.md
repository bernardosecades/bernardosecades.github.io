---
title: "Claude Code: un comando /worktree que monta un worktree de Go y abre GoLand"
date: 2026-06-30 00:00:00
lang: es
ref: claude-code-worktree-command
tags: [claude-code, slash-commands, worktree, golang]
read_min: 4
excerpt_text: "El `claude -w` integrado te deja en un terminal dentro de un worktree recién creado — perfecto, hasta que tu día transcurre de verdad en GoLand y acabas repitiendo a mano el mismo bootstrap de Go. Un comando /worktree propio codifica ese setup una vez: un ID de ticket de entrada, una rama, un .env copiado y GoLand abierto encima."
---

En el [post anterior](/claude-code-git-worktree-paralelo/) me quedé con `claude -w ORD-2231` como la forma del día a día de llevar dos tareas en el mismo servicio sin el ritual de stash + checkout. Crea el worktree, parte de `origin/HEAD` limpio, arranca una sesión dentro y hasta se recoge solo. Para un arreglo rápido es justo lo que quieres.

Pero mi día no transcurre en un terminal — transcurre en GoLand. Y `-w` me deja en una sesión de terminal, no en mi IDE. Así que cada vez que monto un worktree el flujo real lleva tres pasos manuales grapados al final: copiar el `.env` que el servicio necesita, asegurarme de que los módulos están, y abrir aquello en GoLand. Tres pasos que hago de memoria, ligeramente distintos cada vez, y de los que de vez en cuando olvido el primero (el servicio no arranca, y ahí van cinco minutos de confusión — exactamente el Gotcha 1 de la última vez).

Esa es la forma del trabajo que merece convertirse en comando: corto, repetitivo, y lo sigues haciendo a mano.

## Lo que el integrado deja sobre la mesa

`claude -w` es `git worktree add` más convenciones (ruta bajo `.claude/worktrees/`, rama `worktree-<nombre>`, base `origin/HEAD`) más un ciclo de vida, más una sesión de Claude en un terminal. Es el default correcto y lo sigo usando primero.

Lo que no hace — a propósito, porque no puede conocer tu stack — es el bootstrap específico de tu equipo:

- **Abrir tu IDE en el worktree.** Una sesión de terminal no es donde leo código Go; GoLand sí.
- **Tu propia convención de rama y ruta.** Quiero `feature/ORD-2231` y una carpeta `.trees/`, no `worktree-ORD-2231` bajo `.claude/`.
- **Copiar la config ignorada por git que el servicio necesita para arrancar.** El `.env` que no viaja con el worktree.

Nada de eso es difícil. Son tres comandos que prefiero no reteclear — así que los codifico una vez.

## Un comando no es más que un prompt

Un slash command de Claude Code es un fichero markdown en `.claude/commands/`. El nombre del fichero es el nombre del comando; el cuerpo es el prompt que Claude ejecuta; `$ARGUMENTS` (o `$1`, `$2`, …) es donde aterrizan los argumentos. Un poco de frontmatter lo hace visible y seguro:

```markdown
---
description: Crea un worktree de Go para un ticket de Jira y lo abre en GoLand
argument-hint: <TICKET-ID>
allowed-tools: Bash(git worktree:*), Bash(cp:*), Bash(goland:*), Bash(test:*)
---

Crea un git worktree para el ticket `$1`, prepáralo para Go y ábrelo en GoLand.

Pasos:
1. Si `.trees/$1` ya existe, para y dímelo — no lo recrees.
2. Crea el worktree en una rama nueva `feature/$1` desde `origin/main`:
   git worktree add -b feature/$1 .trees/$1 origin/main
3. Copia la config local ignorada por git que el servicio necesita (`.env`, `.env.local`)
   desde la raíz del repo a `.trees/$1`. El module cache de Go es global, así que no hay
   nada más que enlazar.
4. Abre el worktree en una ventana nueva de GoLand: goland .trees/$1
5. Imprime el nombre de la rama y la ruta, y recuérdame que el runtime (puertos, DB) es compartido.
```

Guárdalo como `.claude/commands/worktree.md` y se convierte en `/worktree <TICKET-ID>`. El `argument-hint` aparece en el autocompletado; `allowed-tools` acota lo que el comando puede ejecutar sin preguntar, así que un `/worktree` no es un cheque en blanco sobre tu shell.

Fíjate en el paso 1: esto es un *prompt*, no un script de shell, así que basta con decirle "si el worktree existe, para". Claude lo comprueba y aborta con un mensaje claro en vez de pelearse con el `fatal: ... already used by worktree` de git.

## Adaptar la receta de Python a Go

La versión de este comando que circula usa Python y VS Code, y su paso clave es **hacer symlink de `.venv`** dentro del worktree nuevo para no reinstalar cada dependencia, y luego lanzar `code`. Tradúcelo literal a Go y te pondrías a buscar el equivalente de `.venv` que enlazar.

No lo hay — y ahí está la gracia. El module cache de Go es **global** (`$GOPATH/pkg/mod`, con los builds cacheados en `$GOCACHE`). Todos los worktrees de la máquina ya lo comparten; el primer `go build` en un worktree nuevo resuelve desde la caché, no desde la red. Así que el paso del symlink no se traduce — simplemente *desaparece*. El comando de Go es más corto que el de Python.

Lo que sí hay que tratar es lo único que el post anterior marcaba como Gotcha 1: los ficheros que git ignora pero el servicio necesita para arrancar. En Go eso es casi siempre el `.env`. Así que el paso 3 no es un symlink, es una copia — el único paso manual que, olvidado, hace que el servicio muera al arrancar. Y `code` pasa a ser `goland`: JetBrains trae un launcher de línea de comandos, así que `goland .trees/ORD-2231` abre ese directorio en una ventana nueva.

## En marcha

Desde cualquier sesión en el repo:

```text
> /worktree ORD-2287

✓ .trees/ORD-2287 todavía no existe
✓ git worktree add -b feature/ORD-2287 .trees/ORD-2287 origin/main
✓ copiado .env → .trees/ORD-2287/.env
✓ goland .trees/ORD-2287   (abriendo…)

Rama feature/ORD-2287 lista en .trees/ORD-2287.
Ojo: los puertos y la DB local siguen compartidos con tus otros worktrees.
```

Una línea de entrada, una rama nombrada con el ticket, un checkout que arranca y GoLand abriéndose encima. El ID del ticket que ya nombraba [la sesión y la rama](/claude-code-sesiones-tickets-jira/) nombra ahora también la ruta del worktree.

## Impacto

- Una línea — `/worktree ORD-2287` — va del ID del ticket a un worktree de Go que arranca, con GoLand abierto encima.
- El bootstrap está **codificado, no recordado**: la copia del `.env` que antes olvidaba es ahora el paso 3, siempre.
- El mismo ID nombra la rama, la ruta y el worktree — sin vocabulario nuevo que inventar por tarea.

## Decisiones

- **`-w` para el terminal rápido, `/worktree` para el flujo de IDE.** El integrado sigue siendo el default; el comando es el 20% específico del equipo que no puede cubrir.
- **Rama `feature/<ticket>`, ruta `.trees/`.** Mis convenciones, no el `worktree-<nombre>` bajo `.claude/` del integrado. `.trees/` va al `.gitignore`.
- **Copiar `.env`, no hacer symlink de nada.** El module cache global de Go significa que no hay análogo de `.venv` que enlazar — lo único que arrastrar es la config ignorada por git.

## Limitaciones

- **Es un wrapper de conveniencia que mantienes tú.** A diferencia de `-w`, no lo mantiene nadie más — si tu bootstrap crece, el comando crece con él.
- **`goland` tiene que estar en tu PATH.** Instala el launcher desde JetBrains Toolbox (o *Tools → Create Command-line Launcher*) o el paso 4 simplemente falla.
- **El runtime sigue siendo uno.** Esto copia ficheros; no aísla puertos ni bases de datos. Todo el Gotcha 2 del [post de worktrees](/claude-code-git-worktree-paralelo/) sigue aplicando.
- **Un prompt tiene criterio, un script tiene garantías.** Claude ejecuta los pasos razonando, que es lo que permite al paso 1 abortar con elegancia — pero si necesitas exactamente los mismos bytes cada vez, un script de shell detrás de `claude -w` es la herramienta más determinista.
