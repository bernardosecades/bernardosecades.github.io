---
title: "Escribimos la spec de un código que ya funcionaba. Fallaron dos tests"
date: 2026-12-08 00:00:00
lang: es
ref: openspec-refactor-spec
tags: [openspec, spec-driven, refactoring, testing, claude-code]
read_min: 6
excerpt_text: "Queríamos unificar dos formas de leer los mismos datos en un servicio sin cambiar lo que reciben los consumidores. Antes de tocar código escribimos con OpenSpec la spec del comportamiento actual y pasamos sus escenarios a tests contra una base de datos real. Sobre develop fallaron dos. Ninguno de los dos bugs estaba reportado."
---

Uno de nuestros servicios tenía dos formas de leer el mismo árbol de datos, y no daban lo mismo. Queríamos dejar una sola sin cambiar lo que recibe ningún consumidor. En un refactor así, el riesgo es romper un comportamiento que no está escrito en ningún sitio, así que antes de tocar el código lo escribimos. Hicimos con OpenSpec la spec de cómo se comporta hoy el servicio, convertimos cada escenario en un test de integración y lo lanzamos contra el código actual. Fallaron dos. Eran dos bugs que nadie había reportado.

Es el tercer post sobre cómo usamos OpenSpec en el equipo. El [anterior](/openspec-reglas-config-specs-ia/) iba de las reglas que le damos a la IA. Este va de usar una spec cuando se supone que no cambia nada.

## El código

`accounts` guarda delegaciones: una cuenta puede gestionar otras cuentas, y esas a su vez otras, así que sale un árbol. Varias RPCs y eventos exponen ese árbol. Había dos formas de leerlo:

- Una iba por el ORM, leía un solo nivel, tenía filtros opcionales y paginaba con un máximo de 100 por página.
- La otra era SQL a mano con filtros fijos, hacía la recursión dentro del repositorio y solo devolvía IDs.

Cada caller usaba una u otra, y ninguna tenía tests contra una base de datos real. La idea era quedarnos con una única lectura de un nivel con filtros combinables, y subir el recorrido del árbol y la paginación a la capa de servicio.

## Primero la spec

Creamos el change como una capacidad nueva, `account-delegations`. La spec describe el árbol tal como lo exponen las RPCs y los eventos: campos del proto, flags y estados de dominio. Siguiendo las [reglas del config](/openspec-reglas-config-specs-ia/), no lleva nombres de funciones, así que sobrevive al refactor que tiene que proteger.

Uno de los requisitos, resumido:

```markdown
### Requirement: REQ-02 Cuentas gestionadas

Cuando se piden las delegaciones de una cuenta, el sistema MUST devolver en
`delegations.managed` las cuentas que gestiona directamente y, además, las
que gestiona cada una de ellas. Siempre dos niveles.

#### Scenario: Segundo nivel de varias cuentas gestionadas
- **GIVEN** una cuenta X que gestiona A y B
- **AND** A gestiona A1
- **AND** B gestiona B1, todas válidas
- **WHEN** se pide GetAccount de X con `with_delegations` = true
- **THEN** `delegations.managed` **SHALL** contener A, B, A1 y B1
```

La primera tarea de `tasks.md` era escribir los tests de integración a partir de esos escenarios, contra una base de datos real en un contenedor, y pasarlos sobre develop antes de cambiar nada. La segunda era el refactor.

## Los dos tests en rojo

Sobre develop pasaron todos menos dos.

El escenario de arriba devolvía A, B y B1. Con más de una cuenta gestionada directamente, del segundo nivel solo quedaban las cuentas de la última.

El otro era un escenario con 101 gestores. La RPC que los lista devolvía 100. Pasaba por la lectura paginada, que se queda en 100 por página, y nadie pedía la siguiente.

Ninguno estaba reportado. Con árboles pequeños y menos de 100 gestores, que es casi cualquier cuenta, los dos se comportaban bien.

## Qué hicimos con ellos

Los arreglamos en el mismo change y lo dijimos. La propuesta los lista en What Changes. En Impact aparecen los consumidores que van a empezar a recibir más IDs (`billing`, `reports` y una caché de `gateway`) y queda claro que el proto no cambia. El resto del comportamiento lo fija la spec, y el refactor tenía que mantenerlo en verde.

`design.md` relaciona cada escenario con su test. Dos de los escenarios tienen test propio por culpa de los bugs: uno para el segundo nivel con varias cuentas gestionadas y otro para más de una página de gestores. La tabla también muestra dos filas sin test dedicado, y ahí quedan apuntadas.

## Impact

- **Dos bugs encontrados antes de que nadie los reportara.** Los dos habrían sobrevivido al refactor, porque el refactor estaba pensado para dejar el comportamiento como estaba.
- **El refactor tuvo red.** La nueva lectura y el código del árbol entraron con el comportamiento anterior fijado por tests contra base de datos real, algo que el servicio no tenía.
- **Los consumidores lo supieron antes.** El cambio de comportamiento está en la propuesta, con la lista de servicios que reciben más IDs, antes de hacer merge de nada.

## Decisions

- **Spec y tests antes del refactor.** La spec de un refactor describe el comportamiento actual, así que se puede comprobar contra el código actual. Esa comprobación es la que encontró los bugs.
- **Base de datos real en lugar de mocks.** Una de las dos lecturas era SQL a mano. Un mock habría devuelto lo que le dijéramos, y los dos bugs estaban en cómo se combinaban las consultas.
- **Arreglar los bugs en este change, a la vista.** La otra opción era una spec que describiera los bugs, tests que los afirmaran y un segundo change para darles la vuelta. Nuestra regla es no documentar comportamientos defectuosos, así que los arreglos entraron aquí, listados en la propuesta.

## Limitations

- **La spec solo cubre las lecturas.** Dos operaciones de escritura también recorren el árbol y se quedaron fuera de este change. Su comportamiento sigue sin estar escrito.
- **Escribir la spec de algo que ya existe lleva tiempo.** Cada escenario necesita a alguien que sepa qué comportamientos son intencionados, y ese conocimiento suele estar repartido entre varias personas.
- **No todos los escenarios tienen test.** La tabla muestra seis escenarios sin test dedicado. Están a la vista, pero no cubiertos.
- **Más IDs sigue siendo un cambio.** Es el comportamiento correcto, pero algún consumidor podría depender de la salida anterior. Lo apuntamos como riesgo en el diseño en lugar de comprobar consumidor por consumidor.
