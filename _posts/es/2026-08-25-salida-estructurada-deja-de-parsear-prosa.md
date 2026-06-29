---
title: "Salida estructurada: deja de parsear prosa, exige un schema"
date: 2026-08-25 00:00:00
lang: es
ref: structured-output-prompting
tags: [prompting, structured-output, reliability, workflow]
read_min: 4
excerpt_text: "Le pides al modelo que extraiga campos y te devuelve un párrafo amable envuelto alrededor de un blob de JSON. Escribes un regex para pescarlo; se rompe en la siguiente respuesta. El arreglo no es un prompt más amable — es acotar la salida a un schema y validarla en el borde."
---

Volvamos a ese endpoint de extracción — sacar las líneas de un payload de `orders`. Le pedí al modelo que "extrajera las líneas como JSON". Lo hizo: *"¡Claro! Encontré 3 ítems:"* seguido de un blob ordenado dentro de un fence ```` ```json ````. Mi `json.loads` se atragantó con la prosa, así que le añadí un regex para coger el bloque del fence. A la semana siguiente llegó una respuesta con dos fences y el regex cogió el equivocado. Estaba escribiendo arqueología de parser contra algo que no intentaba ser parseado.

El modelo no se estaba portando mal. Yo había pedido una *respuesta* — una contestación útil con forma humana — cuando lo que necesitaba eran *datos*.

## Pide datos, no una respuesta

"Extrae las líneas" es una petición que un colega servicial contesta en prosa. Para obtener datos, acota el *formato de salida*, y hay dos mecanismos:

- **Salida estructurada** — entregas un schema JSON y la respuesta queda obligada a cumplirlo. Sin fences, sin preámbulo, sin "¡Claro!" — solo el objeto.
- **Uso estricto de herramientas** — el modelo rellena una llamada a herramienta tipada en vez de escribir texto libre. Los argumentos se validan contra el schema de la herramienta.

En ambos casos la garantía pasa de *"el modelo suele cumplir"* a *"la respuesta cumple la forma, o te enteras al instante"*. Esa es toda la diferencia entre un parser que funciona en la demo y uno que sobrevive al tráfico de producción.

## Un schema es un contrato, y hace doble función

El schema *documenta* la forma y la *exige*. Gasta el esfuerzo aquí — este es el trabajo de verdad, la misma lección que [descripción del puesto por encima del cargo](/roles-prompts-descripcion-no-cargo/): la fiabilidad vive en la especificación, no en pedirlo con educación.

```text
items: array de objetos, cada uno:
  sku:       string, requerido
  quantity:  integer, requerido       # no "tres", no "3 unidades"
  category:  enum[food, electronics, other], requerido
  note:      string, opcional
```

`enum` colapsa una adivinanza abierta en un conjunto cerrado sobre el que puedes ramificar. `requerido` nombra lo que no puedes mandar sin ello. Tipar `quantity` como entero significa que nunca más parseas `"tres"` aguas abajo. El schema es donde la ambigüedad va a morir.

## Valida en el borde, siempre

Aun con la salida acotada, trata la respuesta como input no confiable que entra en tu sistema — la misma postura que [revisar código generado por IA](/revisar-codigo-generado-por-ia-antes-de-confiar/). Parséala a tu struct tipado en el borde; recházala si no encaja en vez de dejar que un objeto medio-válido fluya tres servicios adentro hasta `billing`. Y maneja el caso vacío: un rechazo o una respuesta truncada significa *ningún objeto válido*, no un default que te inventaste en silencio. El schema deja limpio el camino feliz; la comprobación en el borde es lo que evita que el camino infeliz se convierta en una alerta a las 2 de la mañana.

## Qué te da aguas abajo

El parseo se vuelve determinista — `json.loads` a un modelo tipado, listo, sin regex que se rompe con el segundo fence. Las respuestas malas fallan ruidosamente en el borde en vez de aflorar como una fila malformada en `billing` un día después. Y el schema se autodocumenta: la siguiente persona que lea el endpoint ve exactamente a qué forma está obligado el modelo.

## Impact

- **El parseo deja de ser frágil.** Sin arqueología de regex, sin pescar strings — la respuesta encaja con el struct tipado o se rechaza en la puerta.
- **Los errores afloran en el borde, no tres servicios adentro.** Una extracción malformada se caza donde entra, no donde acaba rompiendo algo aguas abajo.
- **El contrato es visible.** El schema documenta la forma exacta, así que el comportamiento es reproducible y el siguiente lector no tiene que ingeniería-inversa-rlo desde salidas de ejemplo.

## Decisions

- **Acota el formato; no lo pidas en prosa.** Un schema o una llamada estricta a herramienta gana a "por favor responde solo con JSON" — esa instrucción aguanta hasta la respuesta en la que no.
- **Pon el esfuerzo en el schema.** `enum`, `requerido` y tipos correctos hacen más por la fiabilidad que cualquier cantidad de cortesía en el prompt.
- **Valida en el borde igualmente.** La salida acotada baja la probabilidad de una forma mala; no elimina tu obligación de comprobar antes de que el dato siga adelante.

## Limitations

- **Un schema acota la forma, no la verdad.** El modelo puede devolver un objeto perfectamente válido con el `sku` equivocado dentro — estructura no es corrección, y sigues necesitando comprobaciones puntuales sobre los valores.
- **Los rechazos y la truncación siguen pasando.** Un rechazo de seguridad o un límite de tokens alcanzado no produce objeto válido; sin un camino explícito para el caso vacío confundirás "nada" con "default".
- **Los schemas demasiado estrictos salen mal.** Prohíbe un campo que el dato legítimamente necesita y obligas al modelo a soltarlo o destrozarlo — modela la forma real de tus datos, no la que desearías que tuvieran.
