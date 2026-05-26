---
title: "Cómo un marketplace por equipos acabó con la duplicación de skills en Claude"
date: 2026-05-25 00:00:00
lang: es
ref: claude-code-team-marketplace
tags: [claude-code, plugins, marketplace, workflow]
read_min: 5
excerpt_text: "Tres equipos, tres formas incompatibles de compartir skills de Claude, cero reutilización. Un marketplace de plugins por equipos lo arregló — aquí está el layout, la plantilla y qué cambió de verdad."
---

Tres equipos de ingeniería compartían skills de Claude de tres formas incompatibles y nadie reutilizaba el trabajo de nadie. Las mismas skills escritas dos veces, las buenas invisibles para quien las necesitaba, y una carpeta `~/.claude/` por desarrollador derivando de todas las demás.

Un marketplace de plugins por equipos, organizado como ya trabajan los equipos, lo arregló. Este post documenta el layout, la plantilla que puedes forkear hoy y qué cambió en concreto una vez instalado.

## Tres soluciones incompatibles, ningún marketplace

Antes de que esto estuviera centralizado, vi tres patrones conviviendo en la misma organización de ingeniería:

- **Un repo solo para skills.** Un equipo creó un repo Git con nada más que una carpeta `skills/` y un README pidiendo que la gente la enlazara con symlink bajo `~/.claude/`. Funcionaba para ese equipo — y nadie más lo instalaba.
- **Skills dentro del repo del servicio.** Otro equipo comiteaba sus skills en el mismo repo que el servicio que poseían. La skill solo se cargaba si alguien estaba trabajando *dentro* de ese repo. Útil, pero atado a un único cwd.
- **Copias personales en `~/.claude/`.** Una pareja de ingenieros lo mantenía personal y compartía pegando ficheros `SKILL.md` en hilos de Slack. Dos semanas después había tres versiones ligeramente distintas de la misma skill rodando.

Tres soluciones, ninguna componible. Las skills eran buenas — la distribución era un desastre. La idea del marketplace apareció exactamente aquí: un único sitio que la organización instala una vez, con todo organizado para que no se convierta en el cajón de sastre que sustituye.

## El marketplace es solo un repo

Un marketplace de Claude Code es un repo Git con un `.claude-plugin/marketplace.json` en la raíz. Ese fichero lista todos los plugins del repo, cada plugin es una carpeta, y las carpetas contienen skills / agentes / comandos / hooks. Una vez alguien publica el repo, cualquiera en la organización ejecuta:

```bash
claude plugin marketplace add yourorg/ai-marketplace
claude plugin install <nombre-plugin>@<nombre-marketplace>
```

Ese es todo el interfaz. La decisión interesante es qué metes dentro del repo.

## La trampa: un único plugin gigante

El primer instinto es crear un plugin llamado `engineering` y meterlo todo dentro. A los seis meses:

- Dos equipos han escrito skills `code-review` ligeramente distintos.
- Una skill que backend usa cada día está mezclada con veinte comandos que solo aplican a Android.
- Cada PR toca la misma carpeta, cada PR tiene merge conflicts.
- Nadie quiere borrar nada de otro, así que nunca se borra nada.

El plugin se convierte en un cajón de sastre que todos tienen miedo de limpiar.

## El layout que funciona: una carpeta por equipo

Dale a cada equipo su propia carpeta de plugin. El equipo es su dueño igual que es dueño de un microservicio — nombre, contenido, ciclo de vida.

```text
ai-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    ├── orders/         ← el equipo de orders posee esta carpeta
    │   ├── .claude-plugin/plugin.json
    │   ├── skills/
    │   ├── agents/
    │   ├── commands/
    │   └── hooks/
    ├── billing/        ← el equipo de billing posee esta carpeta
    │   └── …
    ├── notifier/       ← el equipo de notifier posee esta carpeta
    │   └── …
    └── common/         ← plugin transversal (más sobre esto abajo)
        └── …
```

Tres cosas salen casi gratis de este layout:

- **Cero fricción de merge.** Dos equipos editando dos carpetas no chocan jamás.
- **CODEOWNERS funciona solo.** `plugins/orders/* @org/orders-team` y las reviews se enrutan solas.
- **Instalación a la carta.** Una persona de frontend puede instalar `orders@ai-marketplace` y saltarse el resto. No tiene que asumir los hooks de otro equipo.

El nombre del plugin en `plugin.json` debería coincidir con la carpeta (`name: "orders"`). La carpeta se convierte en la unidad de propiedad, en el identificador de instalación y en la superficie de descubrimiento — tres cosas alineadas en una sola frontera.

## Probar el plugin de otro equipo es justo el punto

La razón por la que un marketplace de equipos gana a una pila personal de scripts no es la velocidad. Es que **ves lo que otros equipos han resuelto** sin tener que preguntar.

Instalo `notifier@ai-marketplace` una semana. Tienen un [subagente](/claude-code-subagentes/) que escanea plantillas estilo Slack buscando variables que nunca se resolvieron. Yo no tengo notificaciones en mi servicio, pero la *técnica* — un subagente read-only que recorre una carpeta e informa de referencias rotas — es exactamente lo que necesito para mis fixtures de SQL.

Robo el patrón, lo escribo para mi contexto, lo publico en `plugins/orders/`. Dos meses después alguien de `billing` hace lo mismo con sus ficheros de configuración. Ahora tres equipos tienen una variante de la misma idea.

Esa es la señal para promover.

## El plugin `common` (llámalo como quieras)

Cuando la misma forma de skill, agente o comando aparece en tres plugins de equipo, deja de ser una cosa de equipo. Sácala a un plugin transversal y deja que lo instale todo el mundo:

```text
plugins/common/
├── skills/
│   ├── secrets-no-commit/      ← regla de seguridad que todos quieren
│   ├── pr-checklist/           ← higiene de review que todos quieren
│   └── unresolved-references/  ← la técnica que apareció en tres sitios
└── hooks/
    └── hooks.json              ← bloquear commits a main, avisar de .env, etc.
```

El nombre es guerra de religión. `common`, `shared`, `org`, `misc`, `platform` — los he visto todos. Elige uno y escribe una frase de README explicando qué cabe ahí. La regla que uso:

> Un plugin va en `common` cuando al menos dos equipos lo usan **y** quitarlo les molestaría a todos.

Las reglas de seguridad y los checklists de review casi siempre acaban aquí. Nunca iban a pertenecer a un solo equipo.

## La promoción es toda la gobernanza que necesitas

La mayoría de la documentación de gobernanza de marketplaces se lee como una política de compras. No necesitas nada de eso. El flujo es:

1. **Experimenta** dentro del plugin de tu equipo. A nadie más le tiene que importar.
2. **Usa los plugins de otros equipos** durante una semana. Fíjate en a cuáles vuelves.
3. **Promueve** a `common` solo cuando la misma idea ha aparecido en dos o tres equipos de forma independiente.

Ni comité central, ni bikeshed de nombres por adelantado, ni formulario "solicito añadir un plugin". Los equipos son la unidad, `common` es la síntesis a posteriori. El marketplace acaba pareciéndose al organigrama, que es exactamente lo que quieres — porque así ya funcionan las reviews, la propiedad y las guardias.

## Las actualizaciones viajan por el mismo canal

El marketplace no solo centraliza *dónde* viven los plugins — centraliza *cómo cambian*. Mismo canal para la instalación y para cada actualización posterior.

Antes del marketplace, actualizar una skill significaba: editar tu copia local, pegar el nuevo `SKILL.md` en un hilo de Slack, cruzar los dedos para que los otros equipos lo bajasen. Dos semanas después había tres versiones derivadas circulando. El fix bueno no propagaba; el fix roto tampoco.

Con un único repo y carpetas propiedad de cada equipo, cada plugin declara una `version` en su `plugin.json`. Subir `plugins/orders/.claude-plugin/plugin.json` de `0.4.0` a `0.5.0` y empujar a main es el único paso. Cada instalador de `orders@ai-marketplace` recoge el cambio la próxima vez que refresca el marketplace.

- **Una actualización, todos los equipos.** Un fix en `common/skills/secrets-no-commit/SKILL.md` aterriza en el entorno de cada desarrollador en su siguiente refresh del marketplace. Se acabó el hilo de Slack preguntando "¿bajaste la última versión?".
- **El rollback es fijar versión, no una limpieza desesperada.** ¿Un equipo publica un hook roto en `common/hooks.json`? Fija la versión anterior del plugin hasta que lo arreglen. Antes del marketplace, el rollback era "todos borrad el fichero malo a mano de `~/.claude/`".
- **Los nuevos se incorporan al mismo nivel desde el primer minuto.** El onboarding pasa a ser un `claude plugin marketplace add yourorg/ai-marketplace`, más los plugins de equipo que quieras. Cinco segundos después, la persona nueva está al mismo nivel que alguien que lleva dos años aquí — sin handoff de un zip de `~/.claude/`, sin arqueología en Slack.
- **El changelog es el historial de git.** ¿Quieres saber qué cambió en `plugins/billing/` el mes pasado? `git log` sobre esa carpeta. Antes del marketplace, la "historia de los estándares de Claude en la organización" vivía en repos personales, hilos de Slack y la memoria de la gente — parcial, distribuida, con pérdidas.

El layout de carpeta por equipo iba de *quién posee qué*. El canal de versión + actualización va de *cómo se mueve el cambio*. Poseer tu carpeta no sirve de nada si las actualizaciones siguen pegándose en Slack.

## Una plantilla que puedes forkear hoy

Si quieres probar el layout sin diseñarlo desde cero, he publicado una plantilla mínima en [github.com/bernardosecades/ai-marketplace](https://github.com/bernardosecades/ai-marketplace). Incluye:

- `.claude-plugin/marketplace.json` ya cableado.
- Un `plugins/example/` completo mostrando cada primitiva — skill, agente, comando, hook.
- Un esqueleto vacío `plugins/my-team/` listo para copiar como `plugins/<tu-equipo>/`.

Forkéala, renombra las carpetas de equipo, púshala a tu organización y ejecuta `claude plugin marketplace add yourorg/ai-marketplace`. El layout de carpeta-por-equipo de arriba es justo lo que ya trae la plantilla — este post es el *por qué*, el repo es el *cómo*.

## Impacto

A los dos o tres meses, lo que cambió en concreto:

- **Una instalación en vez de tres.** Antes: cada equipo distribuía a su manera y nadie instalaba el trabajo de otro. Ahora: cada ingeniero ejecuta `claude plugin marketplace add` una vez y elige plugins a la carta.
- **Las actualizaciones se propagan en vez de derivar.** Un fix en `common/` aterriza en la siguiente actualización de cada desarrollador — a los dos o tres meses, cero incidencias de deriva de versiones en los cuatro plugins de equipo.
- **Reutilización donde había cero.** El patrón de un equipo (variables-de-plantilla-sin-resolver) fue la semilla para otros dos, y acabó en un plugin `common/` que ahora usa todo el mundo.
- **Convenciones cross-equipo, por fin aplicables.** Prevención de fugas de secretos e higiene de review pasaron de "deberíamos hacer esto todos" a un único bundle de hooks en `common/` que cada desarrollador baja.
- **De abajo a arriba gana al comité.** Cero reuniones para decidir qué es "oficial". Los patrones se demuestran primero en dos o tres plugins de equipo y luego se promueven por code review.

## Decisiones técnicas

- **Carpeta por equipo, no un único plugin gigante.** La variante de plugin gigante concentra conflictos en una carpeta compartida y nadie se atreve a borrar el contenido de nadie. Una carpeta por equipo le da algo concreto a CODEOWNERS y permite instalaciones a la carta.
- **`common` vive dentro del mismo marketplace, no en uno aparte.** Un segundo marketplace convierte la promoción en un baile de publicar-y-luego-instalar. Mismo repo significa que promover es mover carpeta más un PR.
- **La promoción es de abajo arriba, no de arriba abajo.** Un comité central eligiendo "estándares" produce un cajón de plugins aprobados-pero-no-usados. Esperar a que la misma forma aparezca en dos o tres plugins de equipo mantiene `common` corto y ganado.

## Limitaciones reales

- **Los plugins siguen al final del [orden de prioridad](/claude-code-prioridad-skills/).** Una skill personal en `~/.claude/skills/` gana en silencio a un plugin del marketplace con el mismo nombre. Elige nombres de plugin distintivos — `orders-review`, no `review`.
- **No hay scope de permisos por plugin.** Una vez instalas un plugin confías en todos sus hooks. `common/hooks.json` tiene radio de impacto a nivel de organización y merece el mismo rigor de review que código de infraestructura.
- **La propiedad de carpeta no es aislamiento de runtime.** CODEOWNERS enruta code review, pero nada impide que alguien de un equipo instale y ejecute los hooks de otro equipo. La frontera es social y de revisión, no impuesta por la herramienta.
- No ejecutaría hooks de terceros externos a la organización sin auditoría manual. El formato de plugin facilita demasiado la confianza al vuelo.

## Por qué esto importa

Un plugin que escribiste tú solo es un truco personal. Un plugin que un equipo posee es una convención. Un plugin en `common` es un estándar.

El layout de carpetas decide cuál de las tres llega a ser un pedazo de conocimiento de Claude. Una carpeta por equipo es la estructura más barata que permite que las tres existan a la vez — y que una buena idea viaje de "truco personal" a "estándar" sin que nadie organice una reunión.
