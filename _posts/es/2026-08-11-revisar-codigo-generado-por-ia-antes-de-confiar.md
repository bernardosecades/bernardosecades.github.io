---
title: "Revisar código generado por IA: qué miro antes de confiar"
date: 2026-08-11 00:00:00
lang: es
ref: reviewing-ai-generated-code
tags: [code-review, failure-cases, distributed-systems, workflow]
read_min: 4
excerpt_text: "El código de IA falla distinto al humano: es plausiblemente incorrecto, no obviamente incorrecto. Se lee limpio, los tests están en verde, y el bug está en el traspaso que no pudo ver. Esta es la checklist que corro antes de confiar en un diff que no escribí."
---

Llega un diff de mi pareja de IA: un handler nuevo en `orders`, nombres limpios, tests en verde, se lee como algo que yo habría escrito en un buen día. El instinto es ojearlo y aprobarlo. Ese instinto es justo el problema — *parece terminado* es exactamente el modo de fallo, porque el modelo optimizó por código que **parece** correcto, y se le da muy bien.

Un junior escribe código obviamente incorrecto: una errata, un return que falta, algo que el compilador o un vistazo cazan. La IA escribe código *plausiblemente* incorrecto: la forma correcta, el traspaso equivocado. Así que dejé de revisarlo como código humano y empecé a revisarlo por las formas concretas en que se tuerce.

## Por qué el código de IA necesita otra pasada

El modelo escribió el diff con una vista estrecha: la función que pegué, quizá el fichero. No ve el llamante a tres servicios de distancia, ni la migración que tiene que salir con él, ni la race que acaba de introducir. Rellena esos huecos con lo *plausible*, no con lo *verdadero en tu sistema* — la misma trampa que pedirle que ["arregle el bug"](/depurar-con-ia-dale-las-pruebas/) a ciegas. La revisión es donde aportas la vista de sistema que nunca tuvo.

## Qué miro primero

Estas son las señales propias de la IA, en el orden en que muerden:

- **Los caminos de error, no el happy path.** El modelo adora el camino donde todo funciona. ¿Maneja el timeout, el `nil`, la fila a medio escribir — o solo hace `return err` y sigue? Los errores tragados son lo que más devuelvo.
- **El radio de impacto más allá del diff.** Cambió la función; ¿actualizó los llamantes, la migración, el consumidor en `billing` que lee la salida? El bug suele estar en lo que el diff *no* tocó.
- **Concurrencia y estado compartido.** Una goroutine añadida sin pensar en la race; un lock que protege la escritura pero no la lectura. En un camino distribuido es donde vive el "funciona en mi portátil".
- **Secretos e inyección.** Un token en una línea de log, SQL construido por concatenación, un scope ensanchado "para que funcione". El modelo hace las tres encantado.
- **Scope creep silencioso.** ¿Reformateó un bloque que no venía a cuento, cambió un default, o "mejoró" algo que nunca pedí? Los cambios no solicitados son donde se esconden las regresiones.
- **Tests que verifican el mock, no el comportamiento.** Verde no significa probado. Si el test solo demuestra que se llamó al mock, no demuestra nada.

## Deja que el modelo revise sus propios traspasos

La primera pasada no tiene que ser humana. Antes de leer el diff, le pregunto al modelo de forma adversarial — no "¿esto está bien?" sino *"lista qué se rompe en producción que este diff no maneja, ordenado por probabilidad."* Ese es el [patrón escritor/revisor](/claude-code-escritor-revisor/): un prompt de revisor sin nada en juego, leyendo el diff en frío, buscando problemas en vez de aprobar. Caza barato los traspasos mecánicos. Lo que queda en mí es la decisión de sistema — si *este* cambio es correcto para cómo se comportan de verdad `orders` y `billing` bajo carga.

## Impact

- **Llegan menos diffs plausibles-pero-incorrectos a `main`.** La revisión apunta a donde el código de IA falla de verdad, así que la tasa de captura sube sin que la revisión tarde más.
- **La revisión se vuelve más rápida, no más lenta.** Una checklist fija de señales gana a releer cada línea como si todas fueran igual de sospechosas.
- **La confianza se calibra.** Dejo de oscilar entre sellar a ciegas y reescribirlo todo, y reviso los traspasos que el modelo no pudo ver.

## Decisions

- **Revisa el código de IA por sus modos de fallo, no como si lo hubiera escrito un humano.** Las señales son distintas, así que la checklist es distinta.
- **Deja que corra primero una pasada adversarial, reserva la decisión de sistema para el humano.** El modelo encuentra problemas mecánicos; tú decides si el cambio encaja en el sistema.
- **Trata "parece terminado" como bandera roja, no como luz verde.** Cuanto más limpio se lee, más deliberadamente miro los traspasos.

## Limitations

- **Una checklist no sustituye a entender el cambio.** Si no sabes qué debería hacer el código, ninguna lista caza un diseño sutilmente incorrecto.
- **La pasada adversarial comparte los puntos ciegos del modelo.** Lee el diff en frío pero sigue sin ver tu sistema entero — no sabrá que existe el consumidor en `billing` a menos que se lo digas.
- **Escala con el tamaño del diff.** Un diff de 40 líneas recibe una revisión de verdad; uno de 2.000 se ojea por mucha checklist que tengas. Los diffs pequeños siguen siendo el arreglo real.
