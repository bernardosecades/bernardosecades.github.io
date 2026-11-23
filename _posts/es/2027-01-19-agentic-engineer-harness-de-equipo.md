---
title: "Ir detrás de la IA no escala: el trabajo de un Agentic Engineer es el harness del equipo"
date: 2027-01-19 00:00:00
lang: es
ref: agentic-engineer-team-harness
tags: [harness, spec-driven, agentic-engineering, workflow]
read_min: 15
excerpt_text: "Muchos equipos generan el doble de código con IA y lo siguen revisando línea a línea. El senior acaba bloqueado leyendo PRs de miles de líneas y el resto espera. Lo que ha cambiado es que specs, contratos y tests, que antes eran lo primero que se recortaba, ahora salen baratos. Cuento cómo estamos montando un harness de equipo y por qué creo que saber diseñarlo es la habilidad que define a un Agentic Engineer."
---

Cuando hablo con gente de otras empresas sobre cómo usan la IA para programar, escucho casi siempre la misma historia. El equipo genera mucho más código que hace un año. Las PRs han crecido. Y la revisión sigue funcionando igual que antes: alguien senior abre el diff y lo lee de arriba abajo.

Con PRs de 200 líneas eso funcionaba. Con PRs de 2.000 líneas, varias a la vez y todas con buena pinta, el senior se convierte en el cuello de botella. O se pasa el día revisando y el resto del equipo espera, o empieza a aprobar por encima y los fallos llegan a producción. Me han contado las dos versiones, y en las dos la conclusión que se saca suele ser la equivocada: que la IA genera demasiado código, o que hay que revisar con más cuidado.

Yo creo que el problema está en otro sitio. Esos equipos están usando la IA para escribir más rápido y han dejado el resto del sistema como estaba. Van detrás de la IA, persiguiendo lo que produce. En mi equipo llevamos unos meses intentando lo contrario: usar la IA para construir el sistema en el que trabaja todo el equipo, y que la revisión de miles de líneas deje de ser el punto donde se decide si algo está bien.

Este post es más largo de lo habitual porque junta piezas que he ido contando por separado. Es también un piloto, y lo digo desde el principio: lo estamos probando y cambiando sobre la marcha.

## El cuello de botella se ha movido

Durante años el coste de un cambio estaba en escribirlo. Pensar el diseño, escribir el código, pelearte con los tests. La revisión era una fracción pequeña del total.

Ahora escribir el código cuesta poco. Un agente te saca un handler nuevo con sus tests en minutos. Lo caro ha pasado a ser saber si eso que ha salido es correcto, y esa pregunta la estamos intentando responder con la misma herramienta de siempre: una persona leyendo el diff.

Creo que por eso muchas empresas no ven la productividad que esperaban. Se desarrolla más rápido, eso es real. Pero si la forma de trabajar sigue siendo la misma, el tiempo que se ahorra escribiendo no desaparece. Se va a otras partes del proceso, sobre todo a la cola de revisión y a investigar los bugs que se cuelan a producción. Si lo mides del ticket a producción, la mejora es bastante más pequeña de lo que prometía la demo. Sin una estrategia de harness para todo el equipo, la IA acelera la parte de escribir código y el resto del camino sigue igual de lento.

Leer un diff generado por IA es además más difícil que leer uno escrito por un compañero. Lo conté en [el post sobre revisar código de IA](/revisar-codigo-generado-por-ia-antes-de-confiar/): el código de un junior suele estar mal de forma evidente, y el de la IA está mal de forma plausible. Se lee limpio, los tests pasan, y el fallo está en un traspaso entre servicios que el modelo no podía ver. Para cazar eso hay que leer con mucha atención, y nadie mantiene esa atención durante 2.000 líneas.

Así que el senior que revisa no escala. Tampoco escala pedir a cada developer que revise mejor. Lo que sí puede escalar es mover la verificación a sitios donde sea más barata y no dependa de que alguien lea cada línea.

## Lo que antes no nos podíamos permitir

En todos los equipos en los que he trabajado había cosas que sabíamos que había que hacer y que nunca hacíamos del todo. Escribir la spec antes del código. Definir bien el contrato entre servicios antes de que el consumidor empezara a depender de él. Tener un test por cada comportamiento que importa, y no solo por las funciones que eran fáciles de testear.

No se hacían porque costaban tiempo, y el tiempo se lo llevaba escribir el código. Cuando apretaba una fecha, era lo primero que se recortaba. La spec se quedaba en la descripción del ticket, el contrato se iba ajustando a base de incidencias y los tests cubrían lo que daba tiempo.

Eso es justo lo que ha cambiado. Hoy un primer borrador de una spec con sus escenarios sale en minutos. Lo mismo con un cambio de contrato en un proto o con una batería de tests a partir de una lista de comportamientos. El borrador hay que revisarlo, claro. Pero revisar una spec de 60 líneas de Markdown es mucho más barato que revisar 2.000 líneas de Go, y se hace antes, cuando cambiar algo cuesta una línea de texto.

Mi forma de verlo ahora es esta. El tiempo que la IA nos ahorra escribiendo código no lo quiero gastar leyendo más código. Lo quiero invertir en todo eso que antes no nos podíamos permitir, porque es lo que luego hace que el código se pueda verificar sin leerlo entero.

Para ponerle nombre a esto me ayudó el curso [Agentic Engineer Starter de LIDR](https://lidr.notion.site/curso-agentic-engineer-starter). Lo que cuento a partir de aquí es cómo lo estamos aplicando en mi equipo.

## El harness del equipo

En [el post sobre el harness](/arregla-el-harness-no-el-prompt/) lo definí como todo lo que rodea al modelo: las instrucciones que carga, las herramientas, los permisos, los hooks y lo que ve cuando algo falla. Allí hablaba de mi harness, el de mis sesiones. Aquí hablo del de todo el equipo, que es lo que de verdad cambia la revisión.

Estas son las piezas que tenemos hoy, en el orden en que aparecen durante un cambio. Los nombres de servicios son inventados.

### La spec, revisada antes del código

Usamos OpenSpec con [un repo central solo para specs](/openspec-un-repo-para-todas-las-specs/). Cada cambio empieza con una propuesta y unos escenarios, que se generan con Claude a partir del ticket y luego se revisan en una PR. Esa PR la abren los developers de los servicios afectados y también producto. Producto no lee Go, pero sí lee escenarios escritos como comportamiento observable.

La PR de la spec hace de puerta. Hasta que no está aprobada, no se escribe código. Es el sitio más barato para descubrir que alguien había entendido otra cosa, o que falta un caso, porque arreglarlo ahí cuesta editar un párrafo.

Para que las specs que genera la IA tengan siempre la misma forma, todo el equipo usa el mismo `config.yaml` con reglas por tipo de artefacto. Lo conté en detalle en [el post del config](/openspec-reglas-config-specs-ia/). Varias de esas reglas salieron de fallos reales, y vuelvo a ellas más abajo.

### Escenario y test, uno a uno

Una spec aprobada vale poco si nadie comprueba que el código la cumple. Una de las reglas del config obliga a que `design.md` acabe con una tabla que asocia cada escenario con el test que lo cubre.

Un ejemplo inventado, pero con el formato que usamos. El escenario:

```markdown
### Requirement: REQ-03 Reembolso idempotente

#### Scenario: Reintento del mismo reembolso
- **GIVEN** un pedido cobrado de 50 € en `orders`
- **AND** un reembolso ya aceptado con `idempotency_key` = "r-1"
- **WHEN** llega otro RefundOrder con la misma `idempotency_key`
- **THEN** la respuesta **SHALL** devolver el reembolso original
- **AND** `billing` **SHALL** tener un único reembolso para ese pedido
```

El test que sale de él, de integración y contra una base de datos real en un contenedor:

```go
func TestRefundOrder_RetryWithSameKey(t *testing.T) {
	ctx := context.Background()
	env := newTestEnv(t)
	order := env.chargedOrder(t, 5000)

	req := &ordersv1.RefundOrderRequest{
		OrderId:        order.Id,
		AmountCents:    5000,
		IdempotencyKey: "r-1",
	}

	first, err := env.orders.RefundOrder(ctx, req)
	require.NoError(t, err)

	second, err := env.orders.RefundOrder(ctx, req)
	require.NoError(t, err)

	assert.Equal(t, first.RefundId, second.RefundId)
	assert.Len(t, env.billing.Refunds(t, order.Id), 1)
}
```

Y la tabla de `design.md`:

| Escenario | Test |
|---|---|
| REQ-03 Reintento del mismo reembolso | `TestRefundOrder_RetryWithSameKey` (integración, `orders`) |
| REQ-03 Misma clave con otro importe | sin test dedicado |

Las filas que más me interesan son las que dicen "sin test dedicado". Antes esos huecos se descubrían en producción o no se descubrían. Ahora quedan escritos en el diseño, y quien revisa puede decidir si se aceptan o si hay que cubrirlos.

Esto cambia lo que hago cuando reviso la PR de código. Ya no leo las 2.000 líneas preguntándome si hacen lo que deberían. Compruebo que cada escenario tiene su test, que los tests prueban de verdad el escenario (lo conté en [el post sobre tests de IA en verde](/testing-con-ia-verde-no-es-probado/), un test generado puede pasar sin probar nada) y que lo que queda fuera está apuntado. Después leo con atención las partes donde el riesgo es de diseño. Esas suelen ser pocas.

### Cobertura desde el ticket (piloto)

Esta es la pieza más nueva y la que menos rodada está. Es un comando que se lanza sobre un ticket cuando el desarrollo está terminado. Lee el ticket y la rama de cada repo afectado, sigue las llamadas del código modificado más allá del diff y clasifica cada comportamiento como nuevo o como posible regresión.

Con eso genera tres cosas y las deja publicadas como comentarios en el ticket:

1. Una matriz de cobertura: qué hay que probar a mano y qué no hace falta, con el motivo. La funcionalidad nueva siempre va a la lista de probar. Las regresiones que ya cubren tests que pasan van a la de no probar, indicando qué tests las cubren.
2. Sugerencias de tests. Para cada hueco, el test más pequeño que lo cubriría y de qué tipo sería (unitario, integración o end-to-end).
3. Un plan de pruebas con los casos concretos, en una hoja de cálculo enlazada desde el ticket.

Todo se hace sobre worktrees de solo lectura de la rama, así que nunca toca el repo en el que está trabajando alguien.

Lo que me gusta de esta pieza es que lleva el harness fuera del código. La persona que valida el ticket ya no tiene que reconstruir qué cambió leyendo el diff. Tiene una lista de lo que hay que mirar, y la parte que ya está cubierta por tests viene justificada.

Lo que todavía no sé es cuánto nos podemos fiar de la matriz. El riesgo que más me preocupa es que dé por cubierta una regresión porque existe un test que pasa por ese código, aunque no compruebe el caso concreto. Por eso de momento la tratamos como un borrador que se revisa, y el plan de pruebas lo mira una persona antes de darlo por bueno.

### Revisores antes que personas

Antes de pedir revisión a un compañero, la rama pasa por revisores automáticos. Son agentes de solo lectura, cada uno con un foco:

- Uno revisa el diff de la rama contra main y devuelve hallazgos con severidad y `fichero:línea`, o una aprobación explícita si no encuentra nada.
- Otro hace un análisis de seguridad del diff.
- Otro compara el código con el documento de diseño del cambio y señala dónde se aparta de lo acordado.

Ninguno sustituye la revisión humana, pero le quitan trabajo. Los fallos mecánicos, como un error que se ignora o un campo nuevo que no se propaga, suelen llegar ya marcados, y la persona que revisa puede ir directa a lo que solo ella puede juzgar. Es la misma idea del [patrón escritor y revisor](/claude-code-escritor-revisor/), pero aplicada como paso fijo del flujo y no como algo que cada uno decide si hace.

### Reglas que no dependen de que alguien se acuerde

La última capa son las reglas que tienen que cumplirse siempre. El ejemplo que conté en el post del harness es el de `vendor/`: el agente a veces parcheaba a mano una dependencia vendorizada, el parche pasaba los tests y desaparecía en el siguiente `go mod vendor`. La regla pasó del CLAUDE.md a una skill y de la skill a un hook que bloquea la escritura.

Ese hook todavía vive solo en mi máquina, y ese es justo el paso que le falta. Una regla de harness que solo tengo yo me protege a mí, pero el siguiente compañero que se encuentre con el mismo problema lo va a repetir.

## Cada fallo va al harness

Esta es la parte que más me ha cambiado la forma de trabajar. Cuando algo sale mal, arreglar el bug ya no me parece suficiente. Si solo lo arreglo, el siguiente developer, o el siguiente agente, puede volver a cometer el mismo error dentro de un mes. Así que además del arreglo me pregunto qué parte del harness lo dejó pasar, y llevo el fallo allí.

Con el tiempo he ido viendo que cada tipo de fallo tiene un sitio natural:

| Tipo de fallo | Dónde lo llevo | Ejemplo real |
|---|---|---|
| Alguien entendió mal qué tenía que hacer el sistema | Un escenario en la spec | En [el refactor de `accounts`](/openspec-spec-antes-del-refactor/), escribir los escenarios antes de tocar código sacó dos bugs que llevaban tiempo en producción |
| La IA genera mal un tipo de artefacto | Una regla en el config de specs | Las primeras specs nombraban funciones y paquetes. Ahora hay una regla que lo prohíbe y los nombres van a `design.md` |
| Una regla que se rompe de vez en cuando y sale cara | Un hook | Las ediciones a mano en `vendor/` |
| Al agente le falta contexto o un procedimiento | Una skill o documentación | El procedimiento para cambiar una librería interna: repo hermano, `replace` y `go mod vendor` |
| Un comportamiento que se rompió sin que nadie se diera cuenta | Un test, y su fila en la tabla de escenarios | Los escenarios "sin test dedicado" que acaban cubiertos en el siguiente change |

La tabla no es una regla fija. Hay fallos que van a dos sitios a la vez, como el de `vendor/`, que tiene skill para explicar el camino bueno y hook para bloquear el malo. Lo importante para mí es que la pregunta se haga siempre, y que la respuesta acabe en algo versionado que le llegue a todo el equipo.

Esto también cambia cómo veo las revisiones. Si en una revisión encuentro el mismo tipo de problema por segunda vez, lo que tengo es un hueco en el harness. Dejar el comentario en la PR lo arregla una vez. Llevarlo a una regla, un test o un hook lo arregla para los siguientes.

## Cómo saber si el harness funciona

Todo lo anterior puede sonar a opinión, así que la prueba que me parece más honesta es fácil de plantear: darle la misma tarea a la IA con el harness y sin él, varias veces, y comparar.

La tarea tiene que ser realista y la misma en los dos casos, por ejemplo el reembolso idempotente de arriba. Sin harness, el agente tiene el código y el texto del ticket. Con harness, tiene además la spec aprobada, el CLAUDE.md con los comandos que definen "hecho", la regla de escenario a test y los hooks. Cada variante se ejecuta N veces en modo no interactivo y con el mismo modelo. Después miro tres cosas.

### Calidad de lo que sale

Para no depender de mi impresión, antes de lanzar nada escribo a partir de los escenarios una suite de tests de aceptación que el agente no ve. La nota de cada ejecución es cuántos de esos escenarios pasa, si el lint queda limpio y cuántos hallazgos le saca un revisor independiente.

### Gasto de tokens

Cada ejecución en modo headless devuelve los tokens de entrada y de salida, el coste y el número de turnos. Aquí espero que el harness gaste más por ejecución, porque carga la spec y ejecuta los tests. La pregunta que me interesa es cuánto cuesta llegar a un resultado bueno, contando las ejecuciones que hay que repetir o corregir a mano.

### Predictibilidad

Esta es la que más me importa para un equipo. Si lanzo la misma tarea cinco veces, quiero cinco resultados parecidos en calidad y en el diseño que eligen. Un agente que unas veces lo hace muy bien y otras mal obliga a revisarlo todo con lupa, y volvemos al senior leyendo 2.000 líneas. Un buen harness debería estrechar esa dispersión, porque le quita al modelo decisiones que no le tocaba tomar.

Todavía no tengo los números de esta comparación. Cuando los tenga los contaré, salgan como salgan. Mientras tanto me sirve como criterio: cada pieza que añadimos al harness debería mejorar al menos una de estas tres cosas sin empeorar mucho las otras.

## Agentic Engineer como habilidad

Llevo un tiempo pensando qué diferencia a alguien que usa bien la IA de alguien que es capaz de que todo un equipo la use bien. Creo que la diferencia está en el nivel de abstracción en el que trabaja.

La IA es una herramienta, igual que lo fue el IDE o la integración continua. Un developer que la usa bien escribe más rápido. Eso tiene valor, pero el beneficio se queda en una persona. Y si el resto del sistema sigue igual, ese código extra acaba en la cola de revisión de otro.

Cuando hablo de Agentic Engineer me refiero a una habilidad. No lo veo como un puesto nuevo en el organigrama ni como una persona dedicada solo a esto. Es la capacidad de pensar en el sistema en el que trabaja el equipo además de en el código que estás escribiendo hoy. Quien la tiene diseña el harness: dónde vive la spec y quién la revisa, qué reglas tiene la IA al generarla, cómo se derivan los tests, qué revisores pasan antes que una persona, qué se bloquea con un hook. Y lo diseña para que funcione igual para quien entró ayer que para quien lleva años en el equipo.

Al ser una habilidad, cualquier developer la puede desarrollar, y me parece mejor que la tengan varias personas del equipo. Si solo una persona sabe mejorar el harness, acabamos con el mismo cuello de botella de antes, solo que en otro sitio. Cada vez que alguien lleva un fallo al harness, en lugar de limitarse a arreglarlo, está ejerciendo esa habilidad.

Por eso le doy tanta importancia a la distribución. Las specs y su config viven en un repo compartido. Las skills, los agentes revisores y los comandos van en [el marketplace de plugins del equipo](/claude-code-marketplace-equipos/), así que quien se incorpora los tiene con una instalación. Una mejora del harness que se queda en mi `~/.claude` me ayuda a mí. Una que llega al plugin del equipo ayuda a todos desde el día siguiente.

Mi estimación, y es una estimación porque todavía no lo hemos medido, es que un harness así puede hacer que cada persona del equipo rinda entre un 10 y un 30 % más. Esa cifra me interesa mucho más que la mía. Que yo vaya un 30 % más rápido está bien. Que lo haga cada persona del equipo, y que ninguna tenga que esperar a que alguien senior se lea 2.000 líneas, cambia lo que el equipo puede sacar adelante.

## Impact

- **La revisión se ha movido antes y se ha hecho más pequeña.** Los malentendidos aparecen en la PR de la spec, donde producto también comenta, y la PR de código se revisa contra escenarios y tests en lugar de leerla entera.
- **Los huecos de cobertura quedan escritos.** La tabla escenario a test de `design.md` hace visibles los casos sin test antes del merge.
- **Tres revisores automáticos pasan antes que una persona**, y los fallos mecánicos llegan ya marcados a la revisión humana.
- **Varias reglas del harness salieron de fallos concretos**: tres reglas del config de specs y el hook de `vendor/`. Cada una cubre ahora a todo el que pasa por esa capa, no solo a quien sufrió el fallo.
- **Entre un 10 y un 30 % más por persona**, según mi estimación. No está medido y no quiero presentarlo como un dato.

## Decisions

- **Invertir en spec y tests antes que en revisar más.** Revisar mejor un diff enorme depende de la atención de una persona. Una spec revisada y un test por escenario no dependen de eso.
- **Repo central de specs con un config compartido.** Los cambios cruzan servicios y la spec tiene que poder revisarla gente de varios repos y de producto. Un config por repo hacía que cada change saliera con una forma distinta.
- **Revisores automáticos de solo lectura y como paso fijo.** Si cada uno decide si los pasa, se pasan cuando hay tiempo, que es justo cuando menos falta hacen.
- **Cada fallo se lleva a la capa del harness según lo caro que sea repetirlo.** Una regla en el config cuesta poco. Un hook es código que hay que mantener, así que lo reservo para lo que se rompe y sale caro.
- **Comparar con y sin harness en calidad, tokens y predictibilidad.** Medir solo la calidad de una ejecución esconde lo caro que sale llegar a ella y lo poco que se parece a la siguiente.
- **El flujo de cobertura como borrador revisado.** Mientras no sepamos cuánto acierta, una persona revisa la matriz y el plan antes de usarlos.

## Limitations

- **Es un piloto.** Lo estamos probando en el equipo y cambiando sobre la marcha. Algunas piezas, como el flujo de cobertura, llevan poco tiempo en uso.
- **No tenemos métricas.** No hemos medido el tiempo de revisión ni el lead time antes y después. El 10-30 % es una estimación mía, y la comparación con y sin harness está por hacer.
- **Mantener el harness cuesta.** Reglas, skills, revisores y hooks son código y texto que hay que cuidar, y alguien tiene que dedicarle tiempo de forma explícita.
- **Escribir la spec de algo que ya existe es caro**, porque necesita a alguien que sepa qué comportamientos son intencionados.
- **Parte del harness sigue siendo personal.** El hook de `vendor/` solo existe en mi máquina hasta que lo movamos al plugin del equipo.
- **El harness no sustituye el criterio.** Las decisiones de diseño, los límites entre servicios y los trade-offs los sigue revisando una persona con atención. Lo que hace el harness es dejarle tiempo para eso.
