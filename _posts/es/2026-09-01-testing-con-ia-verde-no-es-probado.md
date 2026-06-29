---
title: "Testing con IA: verde no es probado"
date: 2026-09-01 00:00:00
lang: es
ref: testing-with-ai
tags: [testing, failure-cases, reliability, workflow]
read_min: 5
excerpt_text: "Le pides tests a la IA y te da una suite entera en verde que parece cobertura. Luego los lees: la mayoría comprueban que se llamó a un mock, uno fija el bug que tienes como valor esperado, y ninguno toca el caso que de verdad rompe. Verde, e inútil. Así consigues tests que pueden fallar."
---

Le pedí al modelo que escribiera tests para el camino de cobro entre `orders` y `billing`. Produjo catorce, todos en verde, y parecían concienzudos — setup, teardown, nombres descriptivos. Luego los leí. La mayoría comprobaban que se había llamado a un mock con ciertos argumentos. Uno fijaba el off-by-one que yo de verdad tenía como valor *esperado*. Ninguno tocaba el doble cobro concurrente que era la razón entera por la que ese código me ponía nervioso. Catorce tests en verde que no probaban nada.

Una suite en verde se siente como seguridad. Los tests escritos por IA son muy buenos *sintiéndose* como seguridad sin probar casi nada — y el fallo es lo bastante concreto como para reconocerlo una vez lo has visto.

## Por qué los tests de IA salen verdes y no prueban nada

El modelo optimiza por tests que pasan y parecen completos, y la forma más barata de hacer que un test pase es comprobar lo que el código *hace*, no lo que *debería*. Se repiten tres formas:

- **Tautológico.** El test comprueba que se llamó a un mock, o refleja la implementación paso a paso. Pasa porque es un reflejo del código, no una comprobación sobre él — reescribe la función mal y el test sigue en verde.
- **Solo happy path.** Ejercita el caso que el código ya maneja y se salta el timeout, el input vacío, el duplicado — justo donde vive el bug.
- **Confirma el bug.** Fija el comportamiento *actual* como valor esperado. Si el comportamiento actual está mal, el test ahora guarda el bug y falla el día que lo arreglas.

## Escribe el test primero, luego el código

El único cambio que mata casi todo esto es el orden: consigue el test que falla *antes* de que exista la implementación. Un test escrito contra la implementación puede reflejarla; un test escrito contra la especificación no tiene nada que reflejar, así que no puede ser tautológico. Haz que el modelo proponga el test desde el requisito, confirma que **falla por la razón correcta** — no por una errata, sino porque el comportamiento de verdad aún no está — y solo entonces déjalo implementar. Rojo, luego verde, significa que el verde de verdad significaba algo.

## Apúntalo a los bordes que no ves

Este es el verdadero upside, y es la otra cara del fallo. El modelo es *bueno* enumerando casos límite que un humano cansado se salta — input vacío, el timeout, la petición duplicada, el off-by-one en el borde de un rango, el camino concurrente. El truco es pedirlos explícitamente en vez de dejar que escriba tests de confirmación. El mismo movimiento que [darle las pruebas al depurar](/depurar-con-ia-dale-las-pruebas/): no digas "escribe tests", di *"lista los casos que romperían esto que el happy path no cubre, ordenados por probabilidad"*. Luego escribe tests para esos. El modelo saca a la luz el caso de doble cobro que yo me habría saltado a las 6 de la tarde; yo decido cuáles son reales.

## Lee cada test como si mintiera

La disciplina de [revisar código generado por IA](/revisar-codigo-generado-por-ia-antes-de-confiar/) aplica también a los tests — quizá más, porque un mal test es peor que ningún test: es falsa confianza con una marca verde. Para cada uno, pregunta dos cosas: ¿esta aserción comprueba *comportamiento* o *el mock*? Y **¿fallaría si el código estuviera mal?** Muta la implementación en tu cabeza — invierte una comparación, quita un guard — y si el test seguiría pasando, no está probando nada. Un test que no puede fallar es documentación en el mejor caso.

## Impact

- **El verde vuelve a significar algo.** Los tests escritos spec-first y leídos con el criterio "¿puede fallar?" cazan regresiones reales, en vez de decorar el CI con confianza que nadie se ganó.
- **La cobertura aterriza donde están los bugs.** Pedir explícitamente los casos que rompen empuja los tests hacia el timeout y el duplicado, no hacia una quinta variación del happy path.
- **Se cazan los casos límite que yo me habría saltado.** La enumeración del modelo es de verdad mejor que yo-cansado a las 6 de la tarde — usada con intención, ese es el premio.

## Decisions

- **Test-first siempre que el comportamiento sea especificable.** Un ciclo rojo-luego-verde previene estructuralmente las formas tautológica y de confirmar-bug; escribir tests después del código invita a ambas.
- **Pide los casos que rompen, no "tests".** "Escribe tests" da confirmación; "qué rompe que el happy path se salta" da cobertura.
- **Lee cada aserción con el criterio ¿puede fallar?** Un test que pasa independientemente de si el código es correcto se borra o se reescribe — es peor que ausente.

## Limitations

- **Test-first no encaja en todo.** El código exploratorio, los retoques de UI y los scripts de usar y tirar no tienen una especificación nítida contra la que escribir un test que falle — forzar TDD ahí es ceremonia.
- **El modelo propone; no conoce tu dominio.** Puede listar casos límite plausibles, pero cuáles son *reales* para tu sistema — qué estados son de verdad alcanzables — sigue siendo decisión tuya.
- **Cobertura no es corrección.** Una suite en verde que puede fallar es necesaria, no suficiente; los casos que nunca se te ocurrió preguntar quedan sin probar por mucho que se hayan escrito los tests.
