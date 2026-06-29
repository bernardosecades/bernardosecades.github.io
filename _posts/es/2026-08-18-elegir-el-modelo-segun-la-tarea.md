---
title: "Elegir modelo: deja de tirar siempre del más grande"
date: 2026-08-18 00:00:00
lang: es
ref: choosing-the-right-model
tags: [models, cost, prompting, workflow]
read_min: 4
excerpt_text: "El instinto dice que el modelo más grande es la opción segura. Pero para clasificar, extraer y enrutar, el tier barato suele ser indistinguible en precisión y mucho más rápido — y cuando falta precisión, el arreglo suele ser un prompt mejor, no un modelo más grande."
---

Un endpoint de extracción — sacar las líneas de un payload de `orders` a JSON estructurado — corriendo sobre el modelo más potente. Funciona genial. También cuesta unas cinco veces lo que necesita y su latencia p95 es el doble de lo que podría ser. El modelo nunca fue la parte difícil de esa tarea; lo eran el schema y el prompt. Había tirado del modelo más grande por instinto, como quien coge la herramienta más fuerte de la caja sin comprobar si el tornillo la necesita.

Elegir modelo es una decisión de ingeniería de verdad, con una factura de coste y latencia detrás, no un default.

## Los tiers, y para qué sirve cada uno

El catálogo se ordena en tres tiers, y la diferencia de precio entre ellos es grande — el tier más barato cuesta por token alrededor de un quinto que el de arriba, con el del medio entre ambos:

- **Tier alto (clase Opus, y el modelo de razonamiento de frontera por encima).** El razonamiento más difícil, trabajo agéntico de largo recorrido, decisiones de juicio donde equivocarse sale caro. Pagas más por token y esperas más.
- **Tier medio (clase Sonnet).** El caballo de batalla: producción de alto volumen, razonamiento sólido, mucho más rápido y barato. Donde debería vivir la mayoría del tráfico de una aplicación.
- **Tier barato (clase Haiku).** Rápido y económico — clasificación, extracción, enrutado, resúmenes cortos, cualquier cosa acotada y bien especificada.

## Empareja el tier con la tarea, no con tu ansiedad

El tirón hacia el modelo más grande es ansiedad, no análisis: *más grande parece más seguro*. Pero en una tarea acotada y bien especificada — etiqueta este ticket, saca estos campos, enruta esta petición — el tier barato suele ser indistinguible del más potente en precisión, siendo varias veces más rápido y barato. La capacidad extra que pagaste no tenía nada que hacer en esa tarea.

Gasta el tier alto donde la tarea de verdad lo pide: razonamiento multi-paso, un bucle agéntico abierto, revisión de código, una decisión de juicio con consecuencias reales. Ahí es donde la diferencia entre tiers se nota de verdad en la salida.

## El esfuerzo es un segundo dial

La elección de modelo no es la única palanca. En los modelos de frontera, un ajuste de **esfuerzo** (bajo / medio / alto) cambia minuciosidad por tokens y latencia *independientemente* del modelo que elegiste. Así que "tier alto a esfuerzo bajo" y "tier medio a esfuerzo alto" son dos opciones reales y distintas — a veces la más barata y rápida gana en la misma tarea. Bárrelo sobre tu propia carga en vez de dejarlo en el default.

## Cómo elijo de verdad

Empieza un tier *por debajo* de tu instinto. Mide sobre un set de evaluación real — tus prompts, tus datos, tu definición de correcto — no sobre sensaciones. Sube solo cuando los números digan que la precisión lo exige. Y cuando falta precisión, mira primero el prompt: instrucciones más claras, un schema más afilado, un par de ejemplos ([descripción del puesto por encima del cargo](/roles-prompts-descripcion-no-cargo/)) cierran el hueco más a menudo que un modelo más grande — y no añaden coste en cada llamada para siempre.

## Impact

- **La factura sigue al trabajo.** Las tareas acotadas de alto volumen corren en el tier barato; el tier alto se reserva para lo que lo necesita, en vez de pagar tarifa de buque insignia por un clasificador.
- **La latencia baja donde el usuario la siente.** Un tier más barato en una tarea acotada suele ser varias veces más rápido — mejora real de p95 gratis.
- **La elección de modelo se vuelve una decisión medida.** Respaldada por un set de evaluación, puedes decir *por qué* una ruta dada corre en un tier dado, y revisarla cuando la tarea cambie.

## Decisions

- **Por defecto hacia abajo, no hacia arriba.** Empieza un tier por debajo del instinto y deja que el set de evaluación te suba, en vez de empezar arriba y no cuestionarlo nunca.
- **Trata el esfuerzo como un eje aparte del modelo.** Ajusta ambos; la combinación más barata y rápida a veces gana sin discusión.
- **Arregla el prompt antes de subir de modelo.** Un modelo más grande es la respuesta cara a un problema que una instrucción mejor suele resolver gratis.

## Limitations

- **Necesitas un set de evaluación real para hacer esto con honestidad.** Sin él, "el tier barato vale" es una corazonada — y la corazonada equivocada manda errores sutiles a escala.
- **El tier barato tiene techo.** Mete una tarea de razonamiento o de largo recorrido de verdad difícil y te dará salida segura e incorrecta — el ahorro se evapora en cuanto la corrección importa.
- **Los tiers y los precios se mueven.** La *forma* de la decisión es duradera; los ratios exactos y qué modelo está en cada tier no lo son — re-calibra cuando cambie el catálogo.
