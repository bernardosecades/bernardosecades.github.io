---
title: "Roles en los prompts: vale más una descripción del puesto que un cargo"
date: 2026-07-28 00:00:00
lang: es
ref: prompt-roles
tags: [prompting, claude, subagents, workflow]
read_min: 4
excerpt_text: "\"Eres un experto senior\" apenas mueve a un modelo moderno: es una pegatina de credencial, no una instrucción. Un rol gana su sitio cuando codifica comportamiento, restricciones y audiencia: qué hacer, qué no hacer y para quién escribes."
---

Durante un tiempo abría los prompts con el conjuro de aspecto responsable: *"Eres un ingeniero senior con 20 años de experiencia. Refactoriza esta función."* Sentía que estaba preparando al modelo para dar lo mejor de sí. La respuesta llegaba limpia, segura — y no mejor que cuando borraba esa primera frase del todo.

Esa frase no era una instrucción. Era una pegatina de credencial. Y el modelo no se vuelve más capaz porque yo le diga que lo es.

## Por qué el cargo apenas suma

Hay un recuerdo real detrás del hábito. En modelos antiguos y pequeños, el prefijo de persona movía la aguja — empujaba a un modelo débil hacia un registro del que de otro modo se escapaba. Así que "eres un experto en X" se convirtió en memoria muscular, y el músculo sobrevivió al motivo.

En un Claude moderno (de la 3.5 en adelante, y la familia 4.x) el efecto sobre la *precisión objetiva* es marginal o nulo. Un estudio de 2024 sobre personas en los system prompts le puso números: añadir una persona no mejoraba de forma fiable el rendimiento medible y, en algunas tareas, lo empeoraba un poco. La intuición es simple — el modelo ya sabe responder bien a una pregunta de programación. Un título delante de la tarea no le añade una habilidad que le faltaba. Solo gasta tokens.

## Qué te da de verdad un rol

Un rol gana su sitio cuando deja de ser una etiqueta y empieza a codificar una *descripción del puesto* — tres cosas, en concreto:

- **Comportamiento** — qué hacer y qué *no* hacer.
- **Restricciones y postura** — el sesgo desde el que juzga ("no tienes nada en juego").
- **Audiencia** — para quién es la respuesta, lo que fija el nivel de detalle y el registro.

El blog ya tiene un rol que funciona, y funciona exactamente por estos motivos. El [patrón escritor/revisor](/claude-code-escritor-revisor/) se apoya en un agente revisor cuyo prompt empieza así:

```text
You are a code reviewer. You did not write this code and you have
no stake in it. Your job is to find problems, not to approve.
```

La palabra "reviewer" no es lo que hace el trabajo. El trabajo está en el resto: una postura (*no stake*), un comportamiento (*find problems*) y un anti-objetivo explícito (*not to approve*). Recórtalo a solo "Eres un revisor senior" y vuelves a la pegatina.

## Malo vs bueno

La versión cargo-culteada:

```text
Eres un ingeniero de Go senior de talla mundial. Refactoriza esta función.
```

La versión que de verdad acota el trabajo:

```text
Refactoriza esta función. Mantén la firma pública. Sin dependencias
nuevas. Optimiza la legibilidad por encima de la astucia. Explica
cualquier cambio de comportamiento en una línea.
```

El segundo prompt no tiene rol alguno y aterriza mejor — porque las mejoras viven en las instrucciones, las restricciones y el formato de salida, no en una autoimagen. El rol solo vuelve a entrar cuando aporta su parte en *tono o audiencia*:

```text
Explica este stack trace a un dev de backend sin experiencia en Go —
da por hecho que sabe HTTP pero no goroutines.
```

Ese último no es una credencial. Es una calibración: le dice al modelo quién está escuchando, y la explicación se reorganiza en consecuencia.

## Cuándo ayuda un rol (y cuándo es ruido)

| Usa un rol para… | Sáltatelo para… |
|---|---|
| Tono y voz (revisor escéptico, profe en lenguaje llano) | Precisión pura — "eres un experto" ≠ corrección |
| Calibrar la audiencia ("explícaselo a alguien que…") | Tareas factuales/de código donde basta una instrucción clara |
| Encuadre de comportamiento en agentes y subagentes reutilizables | Prompts de usar y tirar donde la instrucción ya lo dice todo |

## Impact

- **Los prompts se acortan y se vuelven honestos.** El esfuerzo se mueve a lo que importa — la instrucción — en vez de a un preámbulo decorativo que parece trabajo pero no lo es.
- **Las tareas medibles mejoran por el motivo correcto.** La precisión sube por el contexto, los ejemplos y un formato de salida explícito. Cuando sube, sabes *por qué*, y puedes reproducirlo.
- **Los roles que conservas son los que merecen la pena.** En la definición de un agente o subagente, un rol de comportamiento se versiona y se reutiliza — como el `diff-reviewer` — en vez de re-improvisarse en cada chat.

## Decisions

- **Separa el rol-credencial del rol-comportamiento.** "Eres un experto senior" es el primero; "no tienes nada en juego, encuentra problemas, no apruebes" es el segundo. Esa línea es toda la diferencia entre cargo cult y técnica.
- **Gasta los roles elaborados donde se reutilizan.** El prompt de un agente versionado merece una persona cuidada. Uno de usar y tirar no — escribe la instrucción y ya.
- **Para subir precisión, invierte en instrucciones, contexto, ejemplos y formato de salida — no en una persona.** Ahí es donde están las mejoras de verdad.

## Limitations

- **Esto no es "los roles no sirven".** Sirven para tono, audiencia y encuadre. Lo que se desinfla es el rol como booster de *precisión*.
- **El efecto exacto depende del modelo y de la tarea.** En un modelo pequeño o local, una persona todavía puede ganarse mejor el sueldo que en un Claude de frontera.
- **Un buen rol no rescata a una mala instrucción.** Si la tarea está mal especificada, ninguna persona lo arregla — el rol es el condimento, no el plato.
