---
title: "Arregla el harness, no el prompt: dónde poner cada regla para que tu agente la cumpla"
date: 2027-01-05 00:00:00
lang: es
ref: harness-not-prompt
tags: [harness, hooks, claude-md, workflow]
read_min: 5
excerpt_text: "Tenía una regla en el CLAUDE.md: no edites vendor/ a mano. El agente la cumplía casi siempre, y las pocas veces que no, el parche pasaba los tests y desaparecía en el siguiente go mod vendor. Reescribir la regla no sirvió. Moverla a otra parte del harness, sí."
---

En los servicios Go de mi equipo las librerías internas van vendorizadas. Si un cambio toca `auditlib`, lo correcto es editar el repo de `auditlib`, apuntar el `go.mod` de `orders` a esa copia local con un `replace` y regenerar con `go mod vendor`. Tenía una línea en el CLAUDE.md que lo decía: no edites `vendor/` a mano.

El agente la cumplía casi siempre. Las pocas veces que no, parcheaba directamente `vendor/example.com/auditlib/...`, los tests pasaban y el cambio parecía bueno. Hasta que alguien volvía a ejecutar `go mod vendor` y el parche desaparecía sin avisar. El bug volvía y nadie entendía por qué, porque el código que lo arreglaba ya no existía en ningún sitio.

Mi primera reacción fue la de siempre. Reescribí la regla, la puse en mayúsculas y añadí un "IMPORTANTE". No cambió nada. Ahí empecé a sospechar del sitio donde había puesto la regla.

## Qué es el harness

El modelo pone el razonamiento. El harness es todo lo demás: el bucle que lo hace llamar herramientas, las herramientas en sí, las instrucciones que carga, los permisos, los hooks y lo que el agente ve cuando algo falla. Una parte la trae Claude Code de serie. La otra la construyes tú, y es la que más influye en cómo se comporta el agente en tu código.

Cuando el agente hace algo mal, ahora me pregunto qué parte de ese harness lo dejó pasar, antes de pensar en cómo pedírselo mejor.

## La escalera

Cada sitio donde puedes poner una regla da una garantía distinta y tiene un coste distinto. Yo los ordeno así, de más blando a más duro:

1. **CLAUDE.md.** Siempre está en contexto y es barato de escribir. Pero es una indicación más entre muchas, y el modelo la sopesa contra todo lo demás.
2. **Skill.** Un procedimiento que se carga cuando la tarea encaja con su descripción. Sirve para el "cómo se hace", con pasos y ejemplos. Sigue siendo texto que el modelo puede no seguir, y si la descripción es floja, [ni se carga](/claude-code-prioridad-skills/).
3. **Feedback loop.** Un test, el lint o el build fallan y el agente ve el error. La señal es determinista, aunque luego dependes de que el agente reaccione bien.
4. **Hook o permisos.** La acción se bloquea antes de ejecutarse. El agente no puede saltárselo por mucho que lo intente.

La regla que sigo es sencilla. Si una regla se rompe más de una vez, sube un escalón.

La de `vendor/` los recorrió casi todos. Empezó en el CLAUDE.md. Cuando vi que fallaba, la saqué a una skill con el procedimiento completo (repo hermano, `replace`, `go mod vendor`), y pasó a fallar menos. Pero menos no es nunca, y un fallo que se borra solo en el siguiente `go mod vendor` es de los que más tiempo hacen perder. Así que la subí a hook.

## El hook

Es un `PreToolUse` que mira las escrituras de ficheros. Si la ruta cae dentro de un `vendor/` que tiene un `go.mod` al lado, la bloquea:

```bash
#!/usr/bin/env bash
set -euo pipefail

file=$(jq -r '.tool_input.file_path // empty')
[[ "$file" == */vendor/* ]] || exit 0

root="${file%%/vendor/*}"
[[ -f "$root/go.mod" ]] || exit 0

cat >&2 <<EOF
Blocked: $file is inside a Go vendor/ directory. Manual edits there are lost on the next 'go mod vendor'.
Do this instead:
  1. Edit the source in the sibling repo (usually ../<lib>, next to $root).
  2. Point go.mod at it with: replace <module> => ../<lib>
  3. Run 'go mod vendor' in $root.
See the go-internal-deps skill for the full procedure.
EOF
exit 2
```

Y en `~/.claude/settings.json`:

```json
"hooks": {
  "PreToolUse": [
    {
      "matcher": "Edit|Write|MultiEdit",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/block-vendor-edits.sh" }]
    }
  ]
}
```

La comprobación del `go.mod` está para no bloquear el `vendor/` de un proyecto PHP. Y como solo miro Edit y Write, `go mod vendor` sigue funcionando, que es justo lo que quiero que haga el agente. Si nunca has montado un hook, en [este post](/claude-code-hooks-notificacion/) explico lo básico con uno de notificaciones.

## Tres cosas que aprendí montándolo

### El mensaje de error también es un prompt

Con `exit 2`, lo que el hook escribe en stderr le llega al modelo como error de la herramienta. Si el mensaje solo dice "denegado", el agente se pone a probar alternativas a ciegas, y una de ellas puede ser peor que la original. Por eso el mío explica qué hacer en su lugar, paso a paso. Lo probé en una sesión real: el Write sobre `vendor/` falló y el mensaje llegó entero, con los tres pasos. El agente no tiene que adivinar el camino bueno.

### Dale un comando que defina "hecho"

El escalón 3 solo funciona si el agente sabe qué ejecutar. En mi CLAUDE.md hay unas líneas que le piden ejecutar siempre los tests (`make test-unit`, `make test-integration`) y `make lint`. Sin eso, el agente decide solo cuándo ha acabado, y suele decidir que pronto. Con eso, lo normal es que un fallo de lint vuelva al bucle y lo arregle él antes de que yo lo vea. Es la misma idea del [post del bug que casi llega a prod](/el-dia-que-la-ia-casi-mete-un-bug-en-prod/): alguna comprobación tiene que venir de fuera del propio modelo.

### Saca del CLAUDE.md lo que ya se cumple solo

Cuando una regla sube de escalón, la quito del sitio anterior. La prohibición de `vendor/` ya no está en el CLAUDE.md. Vive en la skill como procedimiento y en el hook como bloqueo. Cada línea del CLAUDE.md compite por la atención del modelo con las demás, y una regla que ya hace cumplir un hook solo le resta peso a las que todavía dependen de que el modelo las lea.

## Impact

- **Un fallo poco frecuente pero caro deja de ser posible por la vía habitual.** Antes me enteraba cuando el bug volvía. Ahora el agente recibe el bloqueo en el mismo turno, junto con los pasos correctos.
- **El CLAUDE.md es más corto.** Lo que queda son indicaciones de verdad, sin reglas que ya cumple otra pieza.
- **El hook es reciente.** No tengo todavía cifras de "después" que valga la pena contar, y prefiero no inventarlas.

## Decisions

- **Hook a nivel de usuario y no de proyecto.** Trabajo con worktrees y a veces abro Claude dentro de un solo repo. Un hook de workspace no cubriría esas sesiones.
- **Comprobar el `go.mod` junto a `vendor/`.** Bloquear cualquier ruta con `vendor/` era más simple, pero rompía otros lenguajes.
- **No parsear comandos Bash.** Detectar `sed -i` o `cp` sobre `vendor/` sin bloquear `go mod vendor` es frágil, y prefería un hook pequeño que entienda de un vistazo.
- **La skill se queda.** El hook dice qué no hacer y la skill explica cómo hacerlo bien. Necesito las dos.

## Limitations

- **Desde Bash el hook se puede saltar.** Un `sed -i` sobre `vendor/` pasa sin problema. Es un agujero conocido.
- **Cada hook es código que hay que mantener** y añade algo de latencia a cada escritura. No todas las reglas merecen uno. Solo las que se rompen y salen caras.
- **Un bloqueo sin explicación enseña poco.** Si el mensaje del hook fuera malo, el agente sabría qué no hacer, pero no qué hacer en su lugar.
