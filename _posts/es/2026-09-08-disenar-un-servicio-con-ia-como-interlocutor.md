---
title: "Diseñar un servicio: usa la IA como interlocutor, no como autor"
date: 2026-09-08 00:00:00
lang: es
ref: design-with-ai-interlocutor
tags: [architecture, design, distributed-systems, workflow]
read_min: 5
excerpt_text: "Pídele a la IA que diseñe tu servicio y te devuelve un diagrama de tres cajas seguro que fijó en silencio el límite, la llamada sync/async y la garantía de entrega — decisiones que tú nunca tomaste. Un diseño generado es el peor sitio para gastar su seguridad. Úsala para atacar tu diseño."
---

Estaba separando una capacidad de `reports` de `orders` y le pedí al modelo que "diseñara el servicio de reports". Volvió un diagrama limpio de tres cajas: una cola, un worker, una caché. Seguro, plausible, de los que pondrías en una slide. Luego miré lo que había decidido *por* mí — dónde caía el límite del servicio, que el camino era async, que la entrega era at-least-once — tres de las decisiones más difíciles de todo el diseño, zanjadas en silencio y presentadas como hechas. El diseño no estaba mal, exactamente. Estaba sin examinar, y ahora tenía el brillo de un artefacto terminado.

La parte difícil de un servicio nuevo nunca fue el código. Es el límite, la llamada sync-vs-async, qué pasa cuando una dependencia se cae treinta segundos. Un diseño generado es el peor sitio posible para gastar la seguridad del modelo — porque la seguridad es justo lo que esas decisiones no merecen hasta que las has sometido a estrés.

## Un diseño generado esconde las decisiones

El valor del trabajo de diseño vive en las decisiones y el *porqué* detrás de ellas. Una arquitectura generada te entrega el artefacto saltándose el razonamiento, así que heredas elecciones que nunca sopesaste — y descubres cuáles en producción, cuando la suposición que el diagrama hizo en silencio resulta ser falsa. Peor: el modelo encaja con una forma genérica (cola + worker + caché lo arregla todo) en vez de con *tu* carga, *tus* necesidades de consistencia, *tu* tolerancia al fallo. Parece tu diseño. Es el promedio del diseño de otros mil servicios.

## Úsala para someter a estrés, no para producir

Así que invierte el rol. Trae *tu* boceto — tu límite, tu patrón de llamada — y apunta al modelo contra él como adversario, la misma postura escéptica sin nada en juego que el [patrón escritor/revisor](/claude-code-escritor-revisor/):

```text
Aquí está mi diseño para sacar reports de orders: [boceto].
No propongas una alternativa. Ataca este. ¿Por dónde se filtra el
límite? ¿Qué se rompe cuando billing se cae 30s? ¿Qué asume sobre
el orden que yo no he garantizado?
```

El modelo es mucho mejor encontrando el agujero en un diseño que eligiendo el diseño correcto. Pedirle que ataque el tuyo juega a su favor — y mantiene las decisiones donde corresponde. Es el mismo instinto que [dejar que te interrogue sobre el plan](/claude-code-machacame-modo-plan/): el valor está en el interrogatorio, no en una respuesta nueva.

## Hazle defender los dos lados

Cuando de verdad estás atascado entre dos enfoques — una llamada síncrona a `billing` frente a un evento que el worker consume — no preguntes "cuál es mejor". Te dará una elección segura que en realidad es una moneda al aire disfrazada de análisis. Pídele que defienda *con la máxima fuerza cada uno*, y que nombre el modo de fallo que cada uno acepta. Ahora el modelo está ampliando tu espacio de opciones y sacando a la luz el trade-off que de otro modo habrías descubierto por las malas, en vez de esconder la elección dentro de una recomendación. La decisión sigue siendo tuya; el input mejoró.

## Quédate el lápiz, escribe la decisión

Tú eres dueño del límite y del trade-off. El modelo es un pato de goma que contesta — rápido, leído y de verdad útil para encontrar lo que se te escapó, pero no el responsable del sistema a las 3 de la mañana. Así que quédate el lápiz, y cuando la decisión aterrice, escríbela *con* la alternativa que descartaste y por qué. Ese registro — la misma forma que el bloque **Decisions** al pie de cada post de aquí — es lo que deja que la siguiente persona, incluido tu yo futuro, vea el razonamiento y no solo las cajas.

## Impact

- **Las decisiones se toman a propósito.** El límite, el patrón de llamada y el comportamiento ante fallo se eligen y se registran, no se heredan de un diagrama que los dio por supuestos.
- **Los modos de fallo afloran en tiempo de diseño.** "Qué se rompe cuando `billing` se cae 30s" se pregunta sobre una pizarra, donde es barato, en vez de en un incidente, donde no lo es.
- **El espacio de opciones se amplía.** Obligado a defender los dos lados, el modelo saca a la luz el enfoque — y el trade-off — que no habrías sopesado solo.

## Decisions

- **Trae un diseño para atacar, no pidas uno.** El modelo encuentra agujeros mejor de lo que elige arquitecturas; úsalo para lo que se le da bien.
- **Prohíbe la recomendación cuando estás eligiendo.** "Defiende cada uno, nombra qué sacrifica cada uno" gana a "cuál es mejor" — lo segundo esconde una moneda al aire como veredicto.
- **Registra la decisión y la alternativa descartada.** Un diseño que no puedes explicar después es un diseño que vas a relitigar; el *porqué* es la parte duradera.

## Limitations

- **No conoce tus restricciones reales.** Carga, presupuestos de latencia, forma del equipo, la rareza legacy en `orders` — el modelo solo sabe lo que le cuentas, y un estrés contra contexto que falta se pierde los riesgos reales.
- **Un buen interrogatorio puede sellar una premisa mala.** Si la suposición central de tu boceto está mal, el modelo puede atacar los detalles y dejar en pie la base defectuosa — critica lo que le enseñas.
- **Seguridad no es autoridad.** El modelo defenderá cualquier lado con elocuencia; la elocuencia sobre un trade-off no es prueba de que el trade-off se resuelva a tu favor. Sigues teniendo que decidir tú.
