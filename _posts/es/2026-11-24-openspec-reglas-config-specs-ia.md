---
title: "La IA escribía specs que parecían código. Lo arregló un config.yaml"
date: 2026-11-24 00:00:00
lang: es
ref: openspec-config-rules
tags: [openspec, spec-driven, workflow, claude-code]
read_min: 5
excerpt_text: "Nuestras primeras specs generadas con IA nombraban funciones, convertían en requisito un bug que manteníamos a propósito y tenían escenarios sin ningún test. Además cada uno le pedía las cosas a Claude a su manera, y cada change salía con una forma distinta. El config.yaml de OpenSpec le da a la IA el mismo contexto y las mismas reglas en cada artefacto. Estas son las nuestras y los fallos que hay detrás."
---

Cuando empezamos a generar specs con `/opsx:propose`, los borradores parecían correctos a primera vista y en la segunda lectura aparecían los problemas. Los escenarios nombraban funciones y paquetes. Un bug que manteníamos a propósito acabó convertido en requisito. Había escenarios sin ningún test, y nadie se daba cuenta hasta la review. Encima, cada uno le pedía las cosas a Claude a su manera, así que cada change salía con una forma distinta.

OpenSpec tiene un sitio para esto, `openspec/config.yaml`. En el [post anterior](/openspec-un-repo-para-todas-las-specs/) uno de los motivos para tener un repo central de specs era compartir unas mismas reglas en todo el equipo. Aquí cuento cómo son esas reglas y de dónde salieron.

## Qué es el config

`openspec init` crea `config.yaml` con dos secciones opcionales comentadas. `context` es texto libre que la IA ve cada vez que crea un artefacto. `rules` lleva instrucciones para cada tipo de artefacto: proposal, specs, design y tasks. El nuestro, recortado:

```yaml
schema: spec-driven

context: |
  Microservicios en Go que se comunican por gRPC; contratos proto en un repo compartido.
  Eventos asíncronos por Kafka y RabbitMQ.
  Merchants en onboarding V1 o V2, con comportamientos distintos en algunos flujos.
  Jira: PROJ-* es nuestro equipo.

rules:
  proposal:
    - Escribir en castellano
    - Cabeceras de la plantilla en inglés (Why, What Changes, Capabilities, Impact), que las parsea OpenSpec
    - En Impact, listar los microservicios impactados
    - Indicar si cambia algún contrato (proto, topic, cola) y qué consumidores lo notan
  specs:
    - 'Cada requirement se titula "Requirement: REQ-NN <título>"'
    - Describir el contrato observable con los flags y campos del proto y los estados de dominio
    - Sin nombres de funciones, paquetes ni ficheros (eso va en design.md)
    - Un scenario por regla observable
    - No documentar comportamientos defectuosos que se preservan a propósito
  design:
    - Cada decisión con su alternativa rechazada
    - Mapeo scenario → test que lo cubre
  tasks:
    - Ticket de Jira y PR por repo
    - Incluir QA en dev y el archivado del change tras el deploy a prod
```

## El antes y el después

Nuestro primer change salió con el config por defecto, vacío. La propuesta quedó en inglés y con cabeceras que se inventó Claude (Context, Proposed Changes, Impacted Microservices) en lugar de las de la plantilla. `tasks.md` era una línea con el número de ticket. Sirvió, pero el siguiente change habría salido distinto, y el de después también.

El segundo change lo escribimos con el config ya relleno. La propuesta sigue Why, What Changes, Capabilities e Impact, y en Impact aparecen los consumidores que van a recibir una salida distinta. Cada decisión de `design.md` lleva la alternativa que descartamos. `tasks.md` tiene tres bloques: implementación, entrega y cierre del change.

## Tres reglas que salieron de errores

Nada de nombres de funciones, paquetes ni ficheros en la spec. Los primeros borradores describían el comportamiento a través del código, con frases tipo "`ListDelegations` devuelve...". Una spec así se rompe con cualquier refactor aunque el comportamiento no cambie, y alguien de QA no la puede revisar. Ahora la spec habla del contrato observable (campos del proto, flags, estados de dominio) y los nombres van a `design.md`, que sí puede cambiar con el código.

No documentar comportamientos defectuosos que mantenemos a propósito. A veces un change deja una rareza como está porque arreglarla toca en otro change. Cuando Claude describía el comportamiento actual, metía esas rarezas como requisitos con su SHALL. A partir de ahí el bug era parte del contrato, y quien lo arreglara más adelante estaría "rompiendo la spec".

Cada escenario, con el test que lo cubre. Los escenarios vivían en el repo de specs, los tests en el del servicio, y nadie cruzaba unos con otros. Ahora `design.md` acaba con una tabla: escenario a la izquierda, test a la derecha. Las filas que me interesan son las que dicen "sin test dedicado". En el segundo change había dos, con seis escenarios entre las dos, y quedaron apuntadas en el diseño en vez de descubrirse después.

## El resto es convención

Las demás reglas no vienen de un error concreto. Sirven para que los changes se parezcan:

- Prosa en castellano, pero las cabeceras de OpenSpec, SHALL/MUST y GIVEN/WHEN/THEN en inglés, porque OpenSpec las parsea.
- Requisitos titulados `Requirement: REQ-NN <título>`, para que en la review baste con decir "REQ-02".
- Un escenario por regla observable.
- El Impact de la propuesta nombra los servicios afectados y dice si cambia algún contrato y quién lo va a notar.
- `tasks.md` lleva ticket y PR por repo, QA en dev y el archivado tras el deploy a producción. Esto último tiene post propio.

## Impact

- **Los changes tienen la misma forma.** Dos changes son poca muestra, pero el segundo, escrito con el config, se revisa con el mismo checklist que cualquiera de los que vengan.
- **Specs que QA puede leer.** Sin nombres de funciones, los escenarios se revisan sin abrir el código.
- **Los huecos de tests quedan apuntados antes del merge.** La tabla escenario → test del segundo change dejó a la vista seis escenarios sin test dedicado, en el propio diseño.

## Decisions

- **Reglas en `config.yaml` y no en CLAUDE.md.** El config vive en el repo de specs y OpenSpec aplica cada bloque al artefacto que le toca. Un CLAUDE.md se aplica a todo lo que hace Claude en esa carpeta, y un prompt suelto depende de quién lo escriba.
- **Empezar vacío y añadir reglas cuando aparece un problema real.** No intentamos escribir el reglamento entero al principio. Las tres reglas de arriba salieron de borradores que lo hacían mal.
- **Dos idiomas a propósito.** Castellano en la prosa, porque es como el equipo escribe y revisa. Inglés en lo que parsea la herramienta.

## Limitations

- **Las reglas orientan pero no obligan.** `openspec validate` comprueba la estructura. No te va a avisar de que un escenario nombra una función o bendice un bug. Eso lo sigue mirando una persona en la PR de la spec.
- **Algunas reglas piden algo que la IA no sabe.** "No documentar comportamientos defectuosos" solo funciona si alguien ha dicho qué comportamiento es defectuoso. Tiene que salir en explore o en la review.
- **Los changes antiguos se quedan como estaban.** El primero, ya archivado, no sigue nada de esto.
