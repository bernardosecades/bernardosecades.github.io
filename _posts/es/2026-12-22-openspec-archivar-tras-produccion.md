---
title: "Un change no termina con el merge. Lo archivamos después de producción"
date: 2026-12-22 00:00:00
lang: es
ref: openspec-change-lifecycle
tags: [openspec, spec-driven, workflow, claude-code, team]
read_min: 5
excerpt_text: "El archive de OpenSpec pasa los escenarios de un change a las specs que describen el sistema. Si lo lanzas al hacer merge, esas specs describen develop, y quien las lee da por hecho que es producción. Este es el ciclo completo de un change en nuestro equipo, desde el primer comando hasta el archive tras el deploy, y el error que nos hizo dejar el archive para el final."
---

OpenSpec tiene un paso que convierte un change en documentación: `openspec archive` mueve los escenarios del change a `openspec/specs/`, la carpeta que describe cómo se comporta el sistema. Lo natural sería lanzarlo al hacer merge del código. El problema es que el código mergeado está en develop, y quien lee `specs/` da por hecho que está leyendo lo que corre en producción. En el equipo el archive es el último paso, después del deploy a producción, y así lo dice `tasks.md`.

Con este post cierro la serie sobre cómo usamos OpenSpec en el equipo. Los anteriores fueron sobre [el repo central de specs](/openspec-un-repo-para-todas-las-specs/), [las reglas que le damos a la IA](/openspec-reglas-config-specs-ia/) y [una spec escrita antes de un refactor](/openspec-spec-antes-del-refactor/). Aquí pongo todos los pasos en orden.

## El ciclo

Todo esto se lanza desde la carpeta padre del workspace, donde están clonados uno al lado del otro el repo de specs y los servicios.

1. Crear el change. `openspec new change unify-delegations` crea la carpeta en `openspec/changes/`.
2. Escribir los artefactos. `/opsx:propose` redacta `proposal.md`, `design.md`, `tasks.md` y la delta de la spec siguiendo las reglas del config. Yo suelo pasar antes por [grill-me](/claude-code-machacame-modo-plan/), para que los casos borde salgan de preguntas y no del borrador.
3. Validar. `openspec validate unify-delegations` revisa la estructura: requisitos con SHALL o MUST y escenarios con las cabeceras correctas.
4. Abrir la PR de la spec. Va al repo de specs y hace de puerta. QA y el resto del equipo revisan los escenarios, y no se escribe código hasta que está aprobada.
5. Implementar. `/opsx:apply` lee el change y edita los servicios. Cada servicio lleva su PR, y cada tarea terminada recibe su `[x]` en `tasks.md`.
6. QA en dev y después deploy a producción.
7. Archivar. `openspec archive unify-delegations` integra la delta en `specs/` y mueve el change a `changes/archive/` con la fecha delante.

## tasks.md también cubre la entrega

Nuestro primer change tenía un `tasks.md` de una línea, el número de ticket. El segundo tiene tres bloques. Así estaba mientras la PR del servicio esperaba review:

```markdown
## 1. Implementación (PROJ-787)
- [x] 1.1 accounts: tests de integración del árbol contra base de datos real,
      en verde sobre develop salvo los dos fixes (PR #33)
- [x] 1.2 accounts: lectura única de un nivel, recorrido del árbol en la capa de servicio (PR #33)

## 2. Entrega
- [ ] 2.1 accounts: review y merge de la PR #33 a develop
- [ ] 2.2 QA en dev: GetAccount con `with_delegations`, y ListManagers
- [ ] 2.3 Deploy de accounts a producción

## 3. Cierre
- [ ] 3.1 `openspec archive unify-delegations`
```

Quien abre el change sabe en qué punto está sin preguntar a nadie. Código hecho, esperando review, archive pendiente.

## Qué hace el archive

Con todas las tareas marcadas, el archive se ve así:

```
Specs to update:
  provider-retry-webhook: create
✔ Proceed with spec updates? Yes
Applying changes to openspec/specs/provider-retry-webhook/spec.md:
  + 1 added
Totals: + 1, ~ 0, - 0, → 0
Specs updated successfully.
Change 'provider-retry-webhook' archived as '2026-10-22-provider-retry-webhook'.
```

Si quedan tareas abiertas, antes pregunta:

```
Task status: 0/1 tasks
? Warning: 1 incomplete task(s) found. Continue? (y/N)
```

Ese aviso es el único control, y basta una "y" para saltarlo. Nuestro primer change se archivó justo así, con su única tarea sin marcar. El archive siguió adelante y `specs/` se actualizó igual. Con los bloques de entrega y cierre en `tasks.md`, una tarea abierta ahora significa algo concreto (falta QA, falta el deploy), y contestar "y" ahí pasa a ser una decisión que alguien toma sabiendo lo que hace.

## Impact

- **`specs/` describe producción.** Un change solo llega a `specs/` después del deploy, así que lo que hay ahí es lo que está corriendo.
- **Cualquiera ve en qué punto está un change.** `tasks.md` enseña código, review, QA, deploy y archive, con su PR o ticket al lado.
- **Una misma secuencia para todo el equipo.** Los mismos siete pasos en cada change, desde la misma carpeta y con los mismos comandos.

## Decisions

- **Archivar después de producción y no después del merge.** OpenSpec también tiene `/opsx:sync`, que actualiza `specs/` sin archivar el change. No lo usamos al hacer merge por el mismo motivo: describiría develop.
- **El archive es una tarea más de `tasks.md`.** Si está en la lista, no depende de que alguien se acuerde después del deploy.
- **Pasos de entrega en `tasks.md`, no solo código.** QA y deploy forman parte del change. Sin ellos, una tarea abierta no dice nada de dónde está el change, y así fue como el primero se archivó con su única tarea sin marcar.

## Limitations

- **El archive es manual.** Nada conecta el deploy a producción con el repo de specs. Si nadie lanza el archive, el change se queda en `changes/` y `specs/` se queda por detrás de producción.
- **Marcar tareas también es manual.** `/opsx:apply` marca las de implementación, pero QA y deploy se marcan a mano, y el aviso del archive es fácil de saltar.
- **El proceso es joven.** Han pasado dos changes por él. Con los siguientes algo cambiará, y probablemente parte de lo que cuento en esta serie acabe necesitando un post de seguimiento.
