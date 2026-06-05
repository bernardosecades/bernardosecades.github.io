---
title: "Claude Code: dos tareas en el mismo servicio, dos sesiones, cero stash — git worktree"
date: 2026-06-16 00:00:00
lang: es
ref: claude-code-git-worktree-parallel
tags: [claude-code, git, worktree, workflow]
read_min: 5
excerpt_text: "Estás a mitad de un refactor y entra un hotfix urgente en el mismo servicio. El ritual de stash + checkout tiene ahora una víctima nueva: la sesión de Claude que conocía tu código a medias. git worktree — en git desde 2015, no es cosa de Claude Code — te da un segundo directorio de trabajo con su propia sesión, sin tocar la primera."
---

Martes, 11:40. Llevo media mañana dentro de ORD-2210, un refactor del handler de pagos del servicio de orders: quince ficheros tocados, la mitad sin compilar, y una sesión de Claude que conoce cada decisión a medias. Suena el incidente: ORD-2231, el webhook de cancelaciones devuelve 500 en producción. Mismo servicio. Para ya.

El ritual clásico: `git stash`, checkout a main, arreglar, PR, volver, `git stash pop`, rezar. Pero el ritual tiene ahora una víctima que antes no existía: la sesión de Claude. La conversación entera — el plan, los quince ficheros, el porqué de cada cambio — describe un working tree que acaba de dejar de existir. Si le sigues hablando después del checkout, razona sobre código que ya no está en disco. Y la alternativa, cerrarla y abrir otra, tira el contexto que llevabas media mañana construyendo.

El problema no es git: es que un clone tiene un solo directorio de trabajo y tú tienes dos tareas. La solución también es git, y no es nueva. `git worktree` no es una feature de Claude Code — me he cruzado con más de un developer convencido de que sí — sino de git 2.5, julio de 2015. Lo que ha cambiado es que ahora cada directorio de trabajo puede llevar su propia sesión de IA encima, y de repente hay una razón de peso para usarlo.

## Un .git, varios directorios de trabajo

Un worktree es un segundo directorio de trabajo colgando del **mismo** `.git`. No es un clone: no hay segundo fetch, no hay segundo historial. Objetos, ramas, stash, remotes y config son los mismos; un commit hecho en un worktree es visible en el otro al instante.

```bash
git worktree add -b hotfix/ORD-2231 ../orders-hotfix origin/main
```

```text
$ git worktree list
~/work/orders          3f2a91c [feature/ORD-2210-payment-handler]
~/work/orders-hotfix   8c17d04 [hotfix/ORD-2231]
```

El flujo paralelo sale solo:

```bash
cd ../orders-hotfix
claude -n ORD-2231
```

Segundo terminal, segunda sesión, [nombrada con su ticket](/claude-code-sesiones-tickets-jira/) como siempre. La sesión de la feature sigue abierta en el primer terminal, con su working tree intacto debajo. Cambiar de tarea es cambiar de ventana — sin stash, sin checkout, sin contexto que reconstruir en ninguna de las dos direcciones.

## El atajo integrado

Claude Code empaqueta exactamente ese flujo en un flag:

```bash
claude -w ORD-2231
```

Crea el worktree en `.claude/worktrees/ORD-2231/`, le crea la rama `worktree-ORD-2231` y arranca la sesión dentro. Tres detalles con intención:

- **La rama nace de `origin/HEAD`** — la default remota — y no de tu rama actual. Justo lo que un hotfix necesita: partir de main limpio, no de tu feature a medias. (Si quieres partir de tu HEAD local: `"worktree": {"baseRef": "head"}` en settings.)
- **Se recoge solo.** Al cerrar la sesión, si el worktree no tiene cambios ni commits nuevos, se elimina junto con su rama. Si los tiene, Claude pregunta si lo conservas.
- **`--resume` vuelve al worktree.** La sesión queda vinculada a él y se reabre dentro. En el picker, `Ctrl+W` muestra las sesiones de todos los worktrees del repo.

El nombre del worktree y el de la sesión son cosas distintas; para mantener la convención de una sesión por ticket, los dos flags conviven: `claude -w ORD-2231 -n ORD-2231`. Y una variante que merece línea propia: `claude -w "#4812"` hace fetch de esa PR y monta el worktree sobre ella — probar la PR de un compañero sin tocar tu rama.

## Gotcha 1: qué se comparte y qué no

La confusión número uno. El `.git` se comparte entero — ramas, stash, objetos, remotes, hooks, config. Lo que **no** viaja es todo lo que git no versiona:

- `.env`, `.env.local`, `docker-compose.override.yml`, certificados locales
- `vendor/`, `node_modules/`, caches de build

El worktree nuevo nace "limpio": solo lo versionado. El servicio que arrancaba a la primera en tu directorio de siempre aquí muere pidiendo un `.env` que no existe. Es la fuente clásica del "¿por qué no arranca?" — y de la conclusión errónea de que los worktrees "no funcionan".

Claude Code tiene arreglo de serie: un fichero `.worktreeinclude` en la raíz del repo, con patrones estilo `.gitignore`:

```text
# .worktreeinclude
.env
.env.local
docker-compose.override.yml
```

Todo fichero que matchee (y esté ignorado por git) se copia automáticamente a cada worktree nuevo. Versiónalo y el equipo entero hereda el arreglo. Para los worktrees manuales, el equivalente es un `cp` en tu script de bootstrap — o saber que la primera vez toca copiar el `.env` a mano.

## Gotcha 2: el código está aislado; el runtime no

Dos worktrees del mismo microservicio son dos copias del código y **cero** copias del runtime. Los dos quieren el puerto 8080, los dos quieren levantar el mismo docker-compose, los dos apuntan a la misma base de datos local. El primer `make run` en el worktree del hotfix puede pisar — o corromper — lo que tenías a medias en el de la feature.

Lo que a mí me funciona:

- **El runtime tiene un solo dueño.** El worktree de siempre levanta el stack completo; el del hotfix vive de tests — `make test-unit`, integración con testcontainers o puertos efímeros. Para un hotfix casi siempre basta, y es la opción cero-conflictos.
- **Si de verdad necesitas dos stacks:** compose ya separa proyectos por nombre de directorio (o fuerza el tuyo con `docker compose -p ord-2231`), pero los puertos *publicados* siguen chocando — un `PORT=8081` de override o un `.env` distinto por worktree. Que el `.env` no se comparta automáticamente deja de ser un bug y pasa a ser la feature.

## Impacto

- El hotfix no toca la feature: cero stash, cero checkout, y ninguna sesión razonando sobre código que ya no está en disco.
- Cambiar de tarea pasa de un ritual con estado (stash, checkout, pop, conflictos) a cambiar de terminal.
- Con `-w`, la segunda tarea arranca en segundos desde `origin/HEAD` limpio — y se recoge sola al terminar si no dejó nada.
- Encaja con [las sesiones nombradas por ticket](/claude-code-sesiones-tickets-jira/): el mismo ID nombra rama, PR, sesión y ahora worktree.

## Decisiones técnicas

- **Manual para entender, `-w` para el día a día.** `claude -w` no es magia: es `git worktree add` más convenciones (ruta bajo `.claude/worktrees/`, rama `worktree-<nombre>`, base `origin/HEAD`) más ciclo de vida. Haberlo hecho una vez a mano es lo que te salva cuando algo se tuerce.
- **Un worktree por ticket, mismo nombre que la sesión.** `claude -w ORD-2231 -n ORD-2231`. El ID del ticket ya nombraba rama, PR y sesión; el worktree no iba a ser menos.
- **`.worktreeinclude` versionado en el repo.** La lista de "lo que el servicio necesita y git ignora" es conocimiento de equipo, no folklore personal.
- **Decidir el dueño del runtime por adelantado.** Evita el debug de "¿quién ha matado mi contenedor?" — que siempre llega en el peor momento de las dos tareas.

## Limitaciones reales

- **Disco y builds duplicados.** El `.git` se comparte; `vendor/`, `node_modules/` y las caches de build, no. Cada worktree compila desde frío la primera vez. En Go el module cache global amortigua; en Node duele entero.
- **Una rama, un worktree.** Git se niega a hacer checkout de una rama que ya está en otro worktree: `fatal: 'feature/ORD-2210' is already used by worktree...`. Razonable cuando sabes qué es un worktree; críptico cuando no. Suele ser el primer síntoma de un worktree olvidado.
- **El runtime sigue siendo uno.** Los worktrees aíslan ficheros, no puertos ni bases de datos. El gotcha 2 es mitigación, no solución.
- **La limpieza automática tiene letra pequeña.** Aplica a sesiones interactivas; con `claude -p` el worktree se queda. Y los manuales no se recogen solos: `git worktree list` de vez en cuando, `git worktree remove <ruta>` al terminar.
