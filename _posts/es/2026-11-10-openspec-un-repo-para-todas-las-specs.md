---
title: "La spec vivía en un repo y el cambio tocaba tres. Así que las specs tienen ahora su propio repo"
date: 2026-11-10 00:00:00
lang: es
ref: openspec-central-repo
tags: [openspec, spec-driven, workflow, claude-code, team]
read_min: 5
excerpt_text: "Hace un mes defendí que la spec va en el repo del servicio dueño del comportamiento. Luego el equipo empezó a usar OpenSpec, casi todos los cambios tocaban dos o tres servicios y los revisores de los otros repos no llegaban a ver los escenarios. Movimos todas las specs a un repo propio. Cuento por qué, cómo trabajamos con él y lo que cuesta."
---

Hace un mes escribí que la spec tenía que vivir en el repo del servicio dueño del comportamiento. En Limitations añadí que los repos consumidores no la verían. Esa limitación acabó siendo el problema principal. Cuando empezamos a usar OpenSpec en el equipo, casi todos los cambios tocaban dos o tres servicios y elegir un "dueño" para la spec era jugar a adivinar. Las PRs de los otros repos las revisaba gente que no había leído los escenarios.

Así que hicimos justo lo que yo había descartado: un repo que solo contiene specs. Aquí va por qué, cómo trabajamos con él y lo que nos cuesta.

## Qué cambió en lo que pensaba

En el [primer post](/openspec-la-spec-en-el-repo/) descarté una spec a nivel de workspace porque la carpeta del workspace no es un repo, así que nada de lo que hay ahí se versiona ni se revisa. Sobre la carpeta sigo pensando lo mismo. Lo que se me escapó es que el sitio compartido podía ser un repo en sí mismo. Así la spec se versiona y se revisa como cualquier otro código, y ningún servicio tiene que hacer como que es dueño de un cambio que cruza tres.

## Por qué un repo central

Hubo cuatro motivos, y aparecieron más o menos en este orden.

Los cambios cruzan servicios. Cuando `accounts` cambia lo que devuelve en un campo, `billing` y `reports` reciben otra cosa. Con una spec en el repo dueño, quien revisaba `billing` trabajaba a partir del ticket de Jira. Ahora todas las PRs de servicio enlazan a la misma carpeta del change.

Puede revisar gente que no lee Go. QA y producto abren la PR de la spec, leen los escenarios y comentan. Cuando la spec vivía dentro del repo de un servicio, al lado del código, nadie fuera de los developers la abría.

Las reglas son las mismas para todos. `openspec/config.yaml` guarda el contexto y las reglas de cada artefacto, y los comandos y skills `/opsx:*` están commiteados en `.claude/` del mismo repo. Todo el equipo genera specs con las mismas instrucciones. Con una configuración por repo, ya habían empezado a divergir. Del config hablo en el siguiente post.

Tenemos un catálogo del sistema. `openspec/specs/` describe cómo se comporta hoy cada capacidad, la implemente el servicio que la implemente. Cuando entra alguien nuevo, o cuando le pregunto a Claude cómo funciona un flujo, es lo primero que se lee.

La estructura queda así:

```
specs/                          # repo git propio
├── .claude/                    # comandos y skills /opsx, compartidos por el equipo
└── openspec/
    ├── config.yaml             # contexto y reglas por artefacto
    ├── specs/                  # cómo se comporta el sistema hoy
    └── changes/
        ├── unify-delegations/
        │   ├── proposal.md
        │   ├── design.md
        │   ├── tasks.md
        │   └── specs/delegations/spec.md
        └── archive/
            └── 2026-10-22-provider-retry-webhook/
```

## Cómo trabajamos con él

El repo de specs está clonado junto a los repos de los servicios, en la misma carpeta padre que ya uso como [workspace multi-repo](/claude-code-workspace-multi-repo/). Claude arranca en esa carpeta padre. Desde ahí, `/opsx:propose` escribe en `specs/openspec/changes/...` y `/opsx:apply` lee el change y edita código en `accounts/`, `billing/` o donde diga `tasks.md`.

La PR de la spec hace de puerta: se aprueba antes de que nadie escriba código. En el primer post decía que la propuesta es el sitio más barato para arreglar un malentendido. Ahora que QA también la lee, es además donde sale el escenario que falta antes de que llegue como incidencia. Después, cada servicio lleva su propia PR y `tasks.md` las lista todas.

## Impact

- **Todos los revisores leen el mismo contrato.** Las PRs de servicio enlazan a la carpeta del change, así que quien revisa `billing` lee los mismos escenarios que quien revisa `accounts`.
- **QA revisa el comportamiento antes de implementar.** Los comentarios sobre escenarios llegan en la PR de la spec, cuando cambiarlos todavía cuesta una línea de Markdown.
- **Una sola configuración para todo el equipo.** Un config y un juego de comandos `/opsx`, versionados junto a las specs.
- **Las dudas sobre un flujo empiezan en `specs/`.** Puedo saber cómo se comporta algo hoy sin leerme tres servicios.

## Decisions

- **Repo central en lugar de una spec por servicio.** Cuando un cambio cruza servicios, una spec por repo acaba duplicada o invisible para la mitad de los revisores.
- **Repo central en lugar de una carpeta de workspace.** Las dos opciones le dan a Claude la foto completa, pero solo el repo se versiona y se revisa con PRs.
- **La PR de la spec es una puerta.** Sin escenarios aprobados no hay código. Llevar las dos PRs en paralelo sería más rápido, pero entonces la spec tiende a describir lo que el código ya hace.
- **Claude arranca en la carpeta padre del workspace.** Es el único sitio desde el que ve el change y los servicios a la vez.

## Limitations

- **La spec y el código van en PRs distintas.** El enlace entre ellas es manual, una URL en la descripción de la PR y en `tasks.md`. Si una decisión cambia durante la implementación, nada obliga a cambiar la spec con ella.
- **Todos necesitan la misma estructura de carpetas.** `/opsx:apply` da por hecho que el repo de specs y los servicios están uno al lado del otro, así que quien llega nuevo tiene que montarlo antes de su primer change.
- **Es pronto.** Llevamos dos changes por el proceso y solo uno archivado. Esto es lo que hemos visto hasta ahora, y algo cambiará.
