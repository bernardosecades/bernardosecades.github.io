---
title: "Claude Code: reglas de permisos — wildcards, allow/deny, prioridad y plugins"
date: 2026-06-02 00:00:00
lang: es
ref: claude-code-permission-tuning
tags: [claude-code, permissions, settings, plugins, workflow]
read_min: 6
excerpt_text: "Aprobar Bash(ls) cuarenta veces al día te adormece hasta que pulsas Enter sin leer. Permitir Bash(*) te despierta un martes con el repo borrado. La solución son wildcards con la granularidad correcta — y darse cuenta de que los permisos cubren mucho más que Bash."
---

El [post anterior sobre alcance vs permisos](/claude-code-alcance-vs-permisos/) dividía el mundo en "lo que Claude puede ver" y "lo que Claude puede hacer". Este va de la segunda mitad — cómo afinar esas reglas para que te protejan sin enterrarte en diálogos de aprobación.

Dos modos de fallo. Los dos comunes, los dos dolorosos, y ninguno obvio para alguien que empieza hasta que se ha quemado con uno de ellos.

**Demasiado cerrados.** Apruebas `Bash(ls)` cuarenta veces al día. Cada prompt se para en un diálogo de permisos. A los dos días estás pulsando Enter sin leer lo que apruebas — y así es exactamente como se cuela un comando destructivo de verdad, el día que Claude construye algo mal.

**Demasiado abiertos.** Pones `Bash(*)` para que pare la fricción. Tres semanas después, un martes a las 18:42, Claude compone mal un comando durante un refactor y ejecuta `rm -rf` contra una ruta que te importaba. No había una segunda pareja de ojos porque tú habías eliminado explícitamente la segunda pareja de ojos.

La solución no es "más aprobaciones" ni "menos aprobaciones". Es **wildcards con la granularidad correcta**, **reglas `deny` que pillen las trampas obvias**, y darse cuenta de que **los permisos no son solo `Bash`**.

## Los wildcards son el dial, no el interruptor

Una regla de permisos tiene la forma `Tool(patrón)`. El patrón acepta `*`. El dial va desde "precisión inútil" hasta "apertura irresponsable", y tú eliges dónde sentarte.

```text
Bash(ls)              ← solo el literal "ls"
                        inútil: ls /tmp no matchea
Bash(ls *)            ← ls con cualquier arg
Bash(git *)           ← cualquier subcomando git: amplio pero razonable
Bash(git status:*)    ← git status con cualquier arg. Misma idea,
                        forma canónica de Claude Code con dos puntos
Bash(*)               ← todo. No.
```

La trampa típica: empiezas con `Bash(ls)` (demasiado estrecho, inútil), te rindes y saltas a `Bash(*)` (demasiado amplio, peligroso). El punto medio es lo que funciona.

Una base razonable a nivel de usuario en `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(grep *)",
      "Bash(find *)",
      "Bash(cd *)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log)",
      "Bash(npm test)",
      "Bash(npm run lint)"
    ]
  }
}
```

Lo de solo-lectura o idempotente entra. Cualquier cosa que muta estado se queda fuera y pasa por el flujo de aprobación. El resto vive a nivel de proyecto donde el equipo lo modela junto.

## /fewer-permission-prompts — arranca desde tu propio historial

Escribir esto desde cero es aburrido. Claude Code trae un skill que lo hace por ti:

```text
/fewer-permission-prompts
```

Escanea tus transcripts, encuentra los comandos que has aprobado repetidamente, y propone una allowlist ordenada por frecuencia. Aceptas o saltas cada uno; las reglas elegidas se añaden a `.claude/settings.json`.

Lo típico es ejecutarlo una vez tras un par de semanas de uso normal, y luego cada mes para pillar nuevos patrones. No propone `Bash(*)` por más veces que lo hayas aprobado — por diseño, prefiere patrones específicos a los amplios.

Tiene un punto ciego: solo sugiere reglas `allow`, nunca `deny`. Apretar el lado peligroso depende de ti.

## Los wildcards van en los dos sentidos — `deny` es la otra mitad

La misma sintaxis de wildcards aplica a `deny`. Y **`deny` siempre gana sobre `allow`**, así que es tu barrera dura para cosas que nunca deberían pasar de forma autónoma:

```json
{
  "permissions": {
    "deny": [
      "Bash(sudo *)",
      "Bash(rm -rf /*)",
      "Bash(chown *)",
      "Bash(chmod *)",
      "Bash(systemctl *)",
      "Bash(curl * | sh)",
      "Bash(curl * | bash)",
      "Read(./**/.env*)",
      "Read(./**/secrets.*)",
      "Edit(./prod/**)",
      "Edit(./infra/**/*.tf)"
    ]
  }
}
```

Lo que se olvida la gente que solo escribe `allow`: no hay ninguna regla que diga que `Bash(git *)` excluye `Bash(rm -rf:*)`. Si Claude compone un comando raro, el allow amplio no lo pilla. El deny sí.

Modelo mental simétrico: `allow` quita la fricción de lo que te fías de ti mismo; `deny` quita la autonomía de lo que no te fías de ti mismo a las 2 de la mañana.

## Prioridad — cuando las reglas chocan

Los permisos se apilan desde cuatro orígenes, en este orden:

1. **Enterprise gestionado** — distribuido centralmente por tu organización. Máxima prioridad.
2. **Personal** — `~/.claude/settings.json`.
3. **Proyecto (compartido)** — `<repo>/.claude/settings.json`, commiteado.
4. **Proyecto (local)** — `<repo>/.claude/settings.local.json`, gitignored.

Dos reglas de resolución:

- Un `deny` en cualquier parte de la pila **siempre gana** sobre cualquier `allow`.
- Para dos reglas que matchean la misma operación, **gana el patrón más específico**: `Bash(git push origin main)` gana a `Bash(git push:*)` que gana a `Bash(git *)`.

Esto se puede explotar. Un equipo puede meter en un `deny` a nivel proyecto las rutas de producción y sobrevive cualquier `allow` personal. Un individuo puede meter un `deny` personal que ningún settings de proyecto puede deshacer.

Si un permiso parece ignorado, la causa casi siempre es una de estas dos reglas — normalmente un `deny` más amplio en la pila que se te había olvidado que existía.

## No es solo Bash

Esta es la parte que más gente se salta. Las reglas `allow`/`deny` aplican a **toda herramienta que Claude pueda invocar**, no solo a `Bash`. La gramática es `Tool(patrón)`:

- `Read(./.env)` — bloquea leer un fichero concreto.
- `Read(./secrets/**)` — bloquea leer un árbol de directorios entero.
- `Edit(./prod/**)` — bloquea editar rutas de producción.
- `Write(./vendor/**)` — bloquea escrituras (p.ej. al `vendor/` de Go).
- `WebFetch(domain:internal.company.tld)` — restringe fetches salientes.
- `mcp__github__get_file_contents` — permite una tool MCP concreta.
- `mcp__github__merge_pull_request` — deja la tool MCP peligrosa pasando por el flujo de aprobación.

La forma MCP es `mcp__<server>__<tool>`, sin paréntesis. Así es como das a Claude acceso de lectura a GitHub vía MCP pero mantienes el lado de escritura — issues, PRs, releases — controlado.

Una allowlist práctica y mixta puede ser así:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(go test:*)",
      "Bash(ls *)",
      "Read(./**)",
      "mcp__github__get_file_contents",
      "mcp__github__search_repositories"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Edit(./prod/**)",
      "Read(./**/.env*)",
      "mcp__github__merge_pull_request"
    ]
  }
}
```

Mismo dial, distintas herramientas. El modelo mental de `Bash` se traslada a todas.

## Plugins y sus skills — la capa que todo el mundo ignora

Los plugins vienen como un paquete: skills, agentes, slash commands, a veces servidores MCP. Instalar uno es "cargar todo lo anterior". La mayoría instala un plugin una vez y se olvida de que tiene superficie de ataque.

Hay dos palancas.

**Palanca 1 — desactivar el plugin entero.** `settings.json` tiene un mapa `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "slack@claude-plugins-official": false,
    "gopls-lsp@claude-plugins-official": true,
    "marketplace-internal@tuempresa-plugins": true
  }
}
```

Poner un plugin a `false` quita todo lo que aporta — skills, agentes, todo — sin desinstalarlo. Útil cuando solo quieres los skills de un plugin disponibles en algunos proyectos y no en otros (un `false` personal, y luego `true` en el `.claude/settings.local.json` del proyecto).

**Palanca 2 — mantener el plugin, denegar piezas concretas.** Esta es la que se le escapa a la gente. El plugin carga, pero te excluyes de superficie individual. Los skills con namespace de plugin aparecen como `plugin:skill-name` en el harness; la misma forma va en una regla `deny`:

```json
{
  "permissions": {
    "deny": [
      "Skill(marketplace-internal:risky-deploy)",
      "Skill(marketplace-internal:db-restore)"
    ]
  }
}
```

Misma idea para los agentes y las tools MCP que un plugin trae consigo — todos viven detrás de la misma sintaxis `Tool(patrón)` que ya conoces.

Por qué importa: cuando instalas un plugin de marketplace, estás aceptando los skills que el maintainer entrega, *y* los que pueda empujar en el siguiente update. Un `deny` quirúrgico te deja quedarte el 90% del plugin que sí quieres y parar el 10% que no quieres autorizar en piloto automático. Esa conversación nunca llega a darse si solo piensas en `Bash`.

Un primo cercano: si colisionas en nombres entre un skill personal y uno de un plugin, [el orden de prioridad de este post](/claude-code-prioridad-skills/) decide cuál se ejecuta. Personal gana a proyecto, que gana a plugins — pero enterprise gana a los tres.

## Un ejemplo real

Trabajo entre ~25 microservicios Go. Wildcards mixtos, un MCP, un plugin en la pila. Trozo relevante de `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(go *)",
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(grep *)",
      "Bash(find *)",
      "Read(./**)",
      "mcp__github__get_file_contents",
      "mcp__github__search_repositories"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(sudo *)",
      "Bash(curl * | sh)",
      "Read(./**/.env*)",
      "Edit(./infra/**/*.tf)",
      "Skill(marketplace-internal:db-restore)"
    ]
  },
  "enabledPlugins": {
    "slack@claude-plugins-official": false,
    "marketplace-internal@tuempresa-plugins": true
  }
}
```

Lectura: cualquier fichero, cualquier ruta. Escritura/edición: nada de Terraform bajo `./infra/`, nunca. Bash: cualquier cosa que empiece por `git`, `go`, `ls`, `cat`, `grep`, `find` corre sin preguntar; cualquier cosa que empiece por `sudo`, `rm -rf /`, o un curl-pipe-a-shell está bloqueada duro. MCP de GitHub: lado de lectura permitido, lado de escritura cae al flujo de aprobación. Dos skills del plugin del marketplace interno están bloqueadas pero el resto del plugin sigue funcionando.

Una semana de trabajo con este setup: unos tres o cuatro prompts al día van al diálogo de aprobación — los patrones nuevos que aún no he visto — y ninguno de ellos es `ls`.

## Impact

- Los prompts de permisos pasaron de ~40/día a ~3/día tras la primera pasada de `/fewer-permission-prompts` + un bloque `deny` curado a mano. Esos 3/día son lo que quiero: me fuerzan a leer lo que voy a aprobar.
- Dos casi-incidentes evitados en el primer mes tras añadir `Bash(rm -rf /*)` y `Edit(./prod/**)` al deny — Claude propuso ambos durante refactors, el bloqueo duro los paró.
- Un update de plugin cambió silenciosamente el comportamiento de un skill; el `deny` por-skill que tenía sobre el peligroso lo mantuvo inerte hasta que tuve tiempo de leer la nueva versión.

## Technical decisions

- **Wildcards en vez de reglas por comando.** Mantener `Bash(git status)`, `Bash(git diff)`, `Bash(git log)`, `Bash(git checkout *)`, etc. es insostenible. `Bash(git *)` más un bloque `deny` apretado en la mitad peligrosa es donde converge.
- **/fewer-permission-prompts para arrancar, nunca como fuente de verdad.** Siembra la allow list. La deny list se escribe a mano: el skill nunca la propone.
- **`enabledPlugins` para apagar de un plumazo, `deny` para opt-out quirúrgico.** Desactivar un plugin entero es la decisión correcta cuando no te fías del publisher; denegar skills/tools concretas es la correcta cuando confías en el plugin en general pero no en una o dos piezas. Las dos palancas existen por algo — no uses solo la primera.
- **Project-local para rutas de riesgo, user-level para hábito personal.** El `deny` sobre `./prod/**` vive en los settings commiteados del proyecto para que aplique a cada teammate. El `allow` de `Bash(go *)` vive en mi config personal; no todo el mundo lo quiere.

## Real limitations

- **Los wildcards en `Bash()` son patrones de string, no ASTs de shell.** `Bash(git *)` no entiende que `git diff | xargs rm` sea peligroso — el prefijo matchea. El bloque `deny` sobre `rm -rf` lo pilla; el allow solo no.
- **`/fewer-permission-prompts` no sabe qué expone tu plugin.** Solo aprende del historial de `Bash`. Los skills, agentes, y MCP tools que has aprobado figuran como "aprobados" pero el skill no va a sintetizar un allow para ellos. Eso se mete a mano.
- **Un update de plugin puede traer skills nuevos.** Tu deny list solo cubre lo que existía cuando la escribiste. El primer turno tras un update de plugin es el turno para leer de verdad el changelog en vez de pulsar Enter sin pensar.
- **No hay un comando "preview de la política".** No existe `claude settings explain Bash(rm -rf X)` que te diga qué capa ganaría. Cuando algo te chirría, te toca grepear a mano los cuatro `settings.json`.

El dial es un hábito, no un setup de un día. Trata el fichero como un `.bashrc`: edítalo según vas aprendiendo cómo es tu día real, no cómo el primer día te dijo que iba a ser.
