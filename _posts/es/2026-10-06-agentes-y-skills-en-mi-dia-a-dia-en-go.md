---
title: "Deja de crear un \"agent de Go\": organiza el tooling de IA por forma de tarea, no por lenguaje"
date: 2026-10-06 00:00:00
lang: es
ref: agents-skills-go-dev
tags: [agents, skills, workflow, go]
read_min: 5
excerpt_text: "El instinto al adoptar agents y skills como dev de Go es hacer versiones de Go de todo — un reviewer de Go, un bot de tests de Go. Acabas con una pila de herramientas que se solapan y nunca recuerdas cuál invocar. El eje nunca fue el lenguaje."
---

El instinto al adoptar agents y skills como dev de Go es hacer versiones *de Go* de todo: un reviewer de Go, un generador de tests de Go, un comprobador de observabilidad de Go. Seis meses después tienes un cajón lleno de herramientas que se solapan y nunca recuerdas cuál coger. El error no son las herramientas — es el eje sobre el que las organizaste. El lenguaje nunca fue el eje correcto.

El eje correcto es la **forma de la tarea**. Agents y skills son dos primitivas con física completamente distinta, y en cuanto ordenas tu trabajo por la física que necesita, la pregunta "¿me hago una cosa de Go?" se disuelve.

## Dos primitivas, física distinta

Una **skill** se carga en tu contexto principal automáticamente cuando tu petición hace match con su `description`. Viaja contigo *mientras tecleas* — es conocimiento, una convención, un checklist, un procedimiento. Moldea el código a medida que se escribe. No hay paso extra ni esperas a un veredicto.

Un **agent** corre en un contexto aislado, en paralelo, y te devuelve solo una conclusión. Es un investigador. Nunca ve lo que haces en tu propia ventana, y tú nunca ves sus cuarenta lecturas de fichero intermedias — solo el dictamen al final.

Esa diferencia es la decisión entera:

> ¿Lo necesito *mientras escribo*? → **skill.** ¿Es un *veredicto que quiero después* y puede correr a ciegas y en paralelo? → **agent.**

Fíjate en que el lenguaje no entró en ningún momento.

## Mi loop diario de Go, mapeado

Recorre un ticket normal y las dos primitivas caen en momentos distintos.

**Orientarme** (inicio del ticket). Mando un [agent tipo explore](/claude-code-subagentes/) a hacer fan-out por los repos `orders` y `billing` para encontrar dónde vive algo de verdad. Es un agent precisamente porque *no* quiero cuarenta volcados de fichero contaminando mi contexto — quiero una respuesta.

**Escribir** (el grueso del trabajo). Aquí mandan las skills, porque son inline:

- Una skill de **idioms Go** — error wrapping, propagación de `context.Context`, tests table-driven. Aplicado a medida que sale el código, no marcado después.
- Una skill de **observabilidad** — cómo instrumenta `orders` con OpenTelemetry: naming de spans, propagación del context por la llamada, los atributos estándar. Quiero esto en el momento en que escribo el handler, no como un veredicto que llega cuando ya lo hice mal.
- Una skill de **dependencias** — el baile `go.mod replace` + `go mod vendor` cuando apunto a un checkout local de `auditlib`. Procedimental y fácil de equivocar de forma sutil: una skill perfecta.

**Verificar** (antes del PR). Ahora los agents se ganan el sueldo. Despliego un reviewer, un auditor de seguridad y un revisor de tests sobre `git diff main...HEAD` — [en paralelo, cada uno devolviendo su rebanada](/orquestacion-multi-agente-cuando-merece-la-pena/), y yo sintetizo. Este trabajo es aislado y paralelo por naturaleza: ninguno de esos revisores necesita estar en mi cabeza mientras codeo, y ninguno necesita ver a los demás.

**Operar** (cuando algo se rompe). Skills otra vez — el conocimiento procedimental que envuelve un CLI para hacer tail de logs o inspeccionar un registro atascado. Conocimiento que quiero inline en el momento en que depuro, no un informe.

## Por qué no me hago un "agent de Go"

Porque "Go" describe el material, no la tarea. La preocupación de observabilidad aparece como *skill* cuando escribo el handler y podría aparecer como *agent* cuando reviso el diff — mismo tema, dos formas, dos primitivas. Meter ambas en un único "agent de Go" te deja una herramienta cuyo trigger se solapa con otras tres y que invocas en el momento equivocado la mitad de las veces.

Además, cada agent extra tiene un coste silencioso: no es escribirlo, es que compite por tu atención con los que ya tienes. La [precedencia y descubribilidad de skills](/claude-code-prioridad-skills/) es una restricción real. El conocimiento de OpenTelemetry, escrito bien como skill para que la instrumentación salga bien a la primera, hace casi redundante un "auditor de OTel" aparte — solo lo añadiría si quisiera una segunda red deliberada en review, no por completitud.

## Impact

- **Se acabó el tooling duplicado.** Ordenar por forma de tarea colapsa el sprawl de "reviewer de Go / tester de Go / checker de Go" en un set pequeño donde cada cosa tiene un trabajo y un momento.
- **La herramienta correcta aparece sola.** Las skills se autocargan mientras escribo; los agents de review son la pasada deliberada pre-PR. Dejé de preguntarme "¿cuál invoco ahora?".
- **El código sale bien antes.** Las preocupaciones caras (idioms, observabilidad, el baile del vendor) son skills inline, así que moldean el código en vez de llegar como veredicto tardío.

## Decisions

- **Organiza por forma de tarea, no por lenguaje.** Inline-al-escribir → skill; veredicto-aislado-paralelo → agent. "Go" nunca es el factor decisivo.
- **El conocimiento, por defecto a skill.** Reserva los agents para investigación y review que de verdad se benefician de un contexto aislado y paralelo.
- **Añade un agent solo ante necesidad probada y recurrente** — no para cubrir un tema que una skill ya maneja inline.

## Limitations

- **Una skill con una `description` floja es invisible.** Si no se autodispara con el fraseo correcto, el conocimiento es como si no existiera — el trabajo de descubribilidad es parte del coste.
- **Un agent es la forma equivocada para cualquier cosa que necesitabas inline.** Un veredicto que llega después de que escribiste el código solo puede criticar, no prevenir.
- **Algunos casos son genuinamente un juicio.** La observabilidad es el más claro: skill, agent o ambos es una decisión deliberada, no una regla que puedas dejar fija.
