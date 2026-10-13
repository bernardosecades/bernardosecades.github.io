---
title: "El plan de la IA muere con la sesión. Mete la spec en el repo"
date: 2026-10-13 00:00:00
lang: es
ref: openspec-spec-in-repo
tags: [openspec, spec-driven, workflow, claude-code]
read_min: 5
excerpt_text: "Te pasas media hora acordando un plan con Claude, cierras la sesión y el plan desaparece. La siguiente sesión lo reconstruye un poco distinto, y el revisor valida el PR contra tu memoria. OpenSpec mueve el acuerdo al repo, donde se versiona, se revisa y se usa como contrato."
---

Te pasas media hora acordando un plan con Claude: los casos borde, las transiciones de estado, qué hace cada servicio. Luego la sesión termina y el plan se va con ella. La siguiente sesión lo vuelve a sacar, un poco distinto. El revisor valida el PR contra lo que *recuerdas* haber acordado. En un cambio que cruza tres repos, "un poco distinto" es como los bugs llegan a producción.

La solución no es un plan mejor. Es dejar el plan en un sitio que sobreviva a la sesión. Para eso llevo un tiempo usando [OpenSpec](https://github.com/Fission-AI/OpenSpec), con Claude Code haciendo el trabajo.

## Qué aporta OpenSpec

OpenSpec es una capa ligera de specs que vive en tu repo. `openspec init` crea una carpeta e instala unos cuantos slash commands `/opsx:*` en Claude Code:

```
openspec/
├── specs/                  # fuente de verdad: cómo se comporta el sistema hoy
└── changes/
    └── add-manual-review/
        ├── proposal.md     # por qué, y qué cambia
        ├── design.md       # enfoque técnico
        ├── tasks.md        # checklist de implementación
        └── specs/          # deltas: requisitos ADDED / MODIFIED / REMOVED
```

Cada requisito de una delta lleva escenarios (*CUANDO pasa esto, ENTONCES aquello*). Cuando el cambio sale, `/opsx:archive` fusiona las deltas en `specs/`, así que la fuente de verdad se mantiene al día. No tiene más: ficheros Markdown, en git, junto al código que describen.

## El cambio

El caso real: añadir un **estado de revisión manual** a un flujo de onboarding. `accounts` es el dueño del flujo. `billing` tiene que retener la activación mientras la cuenta está en revisión, y el SDK cliente tiene que exponer el nuevo estado. Son dos servicios y un SDK. Sin spec, un cambio de este tamaño es donde le reexplico la feature a Claude tres veces y aun así encuentro un desajuste en la review.

## Mi flujo, paso a paso

**1. Explorar, y luego machacar.** `/opsx:explore` para dar forma a la idea, con [grill-me](/claude-code-machacame-modo-plan/) encima para que Claude me entreviste decisión a decisión en lugar de redactar algo para que yo lo apruebe sin mirar. Aquí salieron los casos borde: qué pasa si la cuenta *ya* está en revisión, quién puede liberarla, qué hace `billing` con un cobro que está en vuelo.

**2. Proponer.** `/opsx:propose add-manual-review` deja las decisiones por escrito en `proposal.md`, `design.md`, `tasks.md` y la delta de la spec. Las respuestas de grill-me dejan de ser historial de chat y pasan a ser ficheros.

**3. Revisar los artefactos antes de escribir código.** Leo los escenarios de la delta como leería un PR. Es el punto más barato para pillar un malentendido: arreglar una línea de Markdown sale mejor que reescribir un handler. Aquí reescribí un par de escenarios.

**4. Aplicar en un contexto limpio.** `/opsx:apply` en una sesión nueva. No necesita nada de la sesión anterior, porque los ficheros guardan el acuerdo entero. Aquí es donde la tesis se cumple: el cambio duró más de una sesión, y cada vez lo retomé con "lee `openspec/changes/add-manual-review`", sin reexplicar nada.

**5. Revisar contra la spec, no contra la memoria.** Antes de archivar, mis [agentes revisores](/claude-code-escritor-revisor/) reciben el diff *y* la delta. Su pregunta ya no es "¿esto tiene buena pinta?", sino "¿cada escenario tiene código que lo cumpla, y hay código que ningún escenario pidió?". Esa pregunta es mucho más fácil de responder bien.

**6. Archivar.** `/opsx:archive` integra la delta en `openspec/specs/`. El siguiente cambio en este flujo parte de una descripción de cómo se comporta de verdad.

## Multi-repo: ¿dónde vive la spec?

OpenSpec funciona por repo, y mi cambio no. Puse la spec **solo en el repo dueño del comportamiento**, `accounts/openspec/`. `billing` y el SDK cambian según `tasks.md`, sin `openspec/` propio.

Las alternativas no se sostenían. Un `openspec/` por repo triplica el mantenimiento y deja tres verdades parciales. Una spec a nivel de workspace, en la carpeta padre de mi [workspace multi-repo](/claude-code-workspace-multi-repo/), es tentadora porque desde ahí Claude ya lo ve todo. Pero esa carpeta no es un repo, así que la spec no se versiona ni se revisa. Eso tira por la borda todo el sentido.

## "¿No es esto plan mode con más pasos?"

No, aunque funcionan bien juntos. Plan mode resuelve **cómo** implementar algo en *esta* sesión, y desaparece cuando la sesión termina. OpenSpec recoge **qué** tiene que hacer el sistema, en ficheros que se versionan, se revisan en el PR y se trasladan a `specs/`. Sigo usando plan mode dentro de `apply` para el cómo. OpenSpec es lo que evita que el qué se desvíe.

## Impact

- **Casos borde antes del código.** Tres decisiones que habrían aparecido como comentarios de review (o como incidencias) quedaron resueltas en explore y en la revisión de la propuesta.
- **Sin reexplicar entre sesiones.** El cambio duró varias sesiones, y cada una arrancó desde la carpeta del change en lugar de desde mi memoria.
- **Review con referencia.** Los agentes revisores contrastaron el diff con escenarios concretos, y eso produjo hallazgos específicos ("el escenario X no tiene código") en lugar de opiniones de gusto.

## Decisions

- **OpenSpec para el qué, plan mode para el cómo.** Se complementan. Ninguno sustituye al otro.
- **Una spec, en el servicio dueño.** Una única fuente de verdad versionada es mejor que copias por repo o que una carpeta de workspace sin versionar.
- **Revisión humana de los artefactos antes de `apply`.** La propuesta es el sitio más barato para arreglar un malentendido, así que recibe una lectura de verdad, no un vistazo.
- **Los revisores reciben la delta, no solo el diff.** Eso convierte la review en un checklist contra escenarios.

## Limitations

- **La spec se desvía si nadie la actualiza.** En este mismo cambio, una decisión (el timeout de la revisión manual) cambió durante `apply`, y ni `design.md` ni la delta se actualizaron. El revisor lo pilló al comparar diff contra delta antes de archivar. Ahora ese check es obligatorio, porque una spec desactualizada es peor que ninguna: miente con autoridad.
- **Excesivo para cambios pequeños.** Una propuesta, un diseño, tareas y una delta para un fix de 10 líneas es burocracia. Reservo OpenSpec para cambios que cruzan servicios o que alteran un comportamiento del que dependen otros equipos.
- **Los repos consumidores no ven la spec.** Los PRs de `billing` y del SDK no llevan la delta de `accounts`, así que sus revisores trabajan sin ella salvo que la enlace a mano.
