---
title: "Claude Code: el patrón writer/reviewer — que el autor no revise su propio diff"
date: 2026-07-07 00:00:00
lang: es
ref: claude-code-writer-reviewer
tags: [claude-code, subagents, code-review, workflow]
read_min: 4
excerpt_text: "Pedirle a la sesión que escribió el código que lo revise no es revisar — es confirmar: cada atajo ya está justificado en su contexto. Un subagente revisor juzga el mismo diff desde una ventana limpia, y de repente los hallazgos aparecen."
---

Antes de empujar una rama, yo tecleaba lo que suena responsable: *"revisa este código antes de que lo suba"*. La revisión volvía en segundos — un par de apuntes de estilo y un visto bueno. Y luego un compañero encontraba un evento duplicado en la ruta de reintentos, en la función exacta que esa revisión acababa de bendecir.

El modelo no falló. El montaje sí. Le pregunté al autor.

## Por qué la auto-revisión falla (esto es lo clave)

La sesión que escribió el código lleva en su contexto toda su historia creativa: los requisitos tal y como los entendió, las alternativas que descartó, la justificación de cada atajo. Cuando le pides a esa misma sesión que revise, no vuelve a derivar las conclusiones — las relee. El bug no es invisible porque el modelo sea flojo; es invisible porque el razonamiento que lo produjo está ahí mismo, en la ventana, ya aceptado.

La revisión de código entre humanos resolvió esto hace mucho: el autor no aprueba su propia PR. El arreglo nunca fue un autor más listo. Fue un segundo par de ojos sin nada en juego.

## El patrón: separar al escritor del revisor

Dos roles, dos contextos:

- El **escritor** es tu sesión principal, haciendo lo que ya hace: implementa, corre los tests, deja la rama en verde.
- El **revisor** arranca a ciegas: un contexto limpio, el diff y los criterios de aceptación. Sin razonamientos previos, sin historial de chat, sin nada en juego.

En Claude Code el aislamiento sale gratis: [los subagentes tienen su propia ventana de contexto](/claude-code-subagentes/) y no ven tu conversación. En aquel post la frontera era un mecanismo de *coste* — el ruido de la exploración muere dentro del subagente. Aquí la misma frontera es el mecanismo de *corrección*: el revisor no puede heredar el sesgo del escritor, porque el razonamiento del escritor nunca la cruza.

## El agente revisor

Un fichero markdown en `.claude/agents/diff-reviewer.md`, commiteado en el repo:

```markdown
---
name: diff-reviewer
description: Revisa el diff de la rama actual contra main. Solo lectura.
  Devuelve hallazgos con severidad y file:line, o una aprobación explícita.
tools: [Bash, Read, Grep, Glob]
---

Eres un revisor de código. Tú no escribiste este código y no tienes
nada en juego. Tu trabajo es encontrar problemas, no aprobar.

1. Ejecuta `git diff main...HEAD` tú mismo. No aceptes un resumen de
   los cambios de nadie — el diff es la fuente de verdad.
2. Lee el código circundante que haga falta para juzgar cada cambio
   en contexto.
3. Reporta solo hallazgos que afecten a comportamiento, corrección o
   seguridad. Detalles de estilo: sáltalos salvo que el linter del
   repo los marcaría.

Por cada hallazgo: severidad (blocker / should-fix / nit), file:line,
qué se rompe y con qué input. Si una pasada de verdad no encuentra
nada, di "aprobado" y lista qué comprobaste — nunca rellenes con
hallazgos para parecer exhaustivo.
```

Dos detalles hacen el trabajo pesado: la lista de tools **no tiene Edit ni Write**, y el paso 1 obliga al revisor a **sacar el diff él mismo** en vez de fiarse del resumen de nadie. Los dos son deliberados — más en Decisiones.

## El bucle

El escritor implementa. Después, desde la misma sesión:

```text
> usa el subagente diff-reviewer sobre esta rama

3 hallazgos:

1. [blocker] internal/worker/process.go:88 — el retry envuelve el
   cobro Y la emisión del evento de auditoría: en un cobro
   reintentado el evento sale dos veces. Eventos duplicados para
   todos los consumidores aguas abajo.
2. [should-fix] internal/handler/submit.go:61 — el err de
   body.Close() queda eclipsado por la asignación en el defer; un
   close fallido se pierde en silencio.
3. [should-fix] internal/store/list.go:47 — la comparación del
   cursor usa >= sobre created_at; cuando el límite de página cae en
   timestamps iguales, el último elemento se repite en la siguiente.
```

El hallazgo 1 había sido "revisado" por la sesión escritora minutos antes — y aprobado. Mismo modelo, mismo diff. Lo único que cambió es que la ventana del revisor no contiene el párrafo donde el escritor se convenció de que el retry era seguro.

El escritor arregla los tres y vuelve a lanzar:

```text
> usa el subagente diff-reviewer sobre esta rama otra vez

aprobado — comprobé la ruta de reintentos (la emisión ya queda fuera
del closure del retry), el manejo del error del Close en el defer y
los límites del cursor con timestamps iguales. Sin más hallazgos.
```

Orden de magnitud: el revisor quemó ~30k tokens sacando el diff y leyendo alrededor; mi sesión principal recibió ~600 de vuelta. La revisión costó una ventana que de todas formas nunca iba a conservar.

## Impacto

- **Hallazgos que la sesión escritora ya había dejado pasar** — el evento duplicado en el retry, el error del `Close` tragado. El mismo modelo los encuentra en cuanto se planta fuera del contexto del escritor.
- **El contexto principal se mantiene limpio.** Los greps y lecturas de la revisión mueren en el subagente; al escritor le llega una lista de hallazgos, aplica los arreglos y sigue.
- **El estándar de revisión está versionado.** `.claude/agents/diff-reviewer.md` vive en el repo: cada compañero — y cada sesión — tiene el mismo revisor, no el prompt que alguien improvise a las seis de la tarde.

## Decisiones

- **El revisor es de solo lectura.** Ni `Edit` ni `Write`. Un revisor que puede parchear se pone a parchear, y pierdes la frontera que hacía fiable su juicio. Los hallazgos cruzan de vuelta como texto; los aplica el escritor, que es donde vive el contexto completo.
- **El revisor ejecuta `git diff` él mismo.** El prompt de despacho lo escribe la sesión escritora — si lleva un "añadí un retry, que es seguro porque…", el sesgo que acababas de aislar vuelve a colarse. El orquestador pasa la rama y los criterios de aceptación, nada más.
- **Escalera de severidad en el prompt.** "Encuentra problemas" sin listón produce doce nits por pasada y el bucle nunca converge. Blocker / should-fix / nit, más un "no rellenes" explícito, mantiene las pasadas cortas y honestas.
- **Dos rondas, y luego un humano.** Escribir → revisar → arreglar → revisar. Lo que sobreviva a la segunda pasada va a una persona, no a una tercera pasada — las rondas repetidas convergen en el gusto del revisor, no en la corrección.

## Limitaciones

- **Independencia de contexto, no de modelo.** Los dos roles corren sobre los mismos pesos; los puntos ciegos del modelo sobreviven a la separación. Poner un `model:` distinto en el frontmatter del revisor diversifica un poco — la revisión humana en cambios arriesgados sigue siendo la red de seguridad.
- **Es análisis estático por defecto.** El revisor lee el diff; no corre la suite salvo que se lo digas. Puedes permitirle `make test` en su prompt, al precio de pasadas más lentas.
- **Una pasada completa de agente por ronda.** Sobre un diff grande son tokens y minutos reales. Para un arreglo de tres líneas, léete el diff tú.
- **El despacho aún puede filtrar encuadre.** "Solo rama + criterios" es una disciplina, no un mecanismo. Si la sesión escritora editorializa en el prompt de despacho, algo de sesgo viaja — la regla del `git diff` estrecha el canal, no lo cierra.
