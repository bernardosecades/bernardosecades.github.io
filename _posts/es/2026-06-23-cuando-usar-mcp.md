---
title: "MCP: cuándo merece la pena y cuándo gana el CLI (el caso de GitHub)"
date: 2026-06-23 00:00:00
lang: es
ref: mcp-when-to-use
tags: [claude-code, mcp, tooling, workflow]
read_min: 4
excerpt_text: "El MCP de GitHub trae unas cuarenta herramientas que casi nunca tocas. Si tienes shell y `gh`, el CLI hace lo mismo con una sola herramienta en vez de cuarenta. El MCP sigue teniendo su sitio — solo que no aquí. Aquí va la regla de decisión."
---

Instalas el MCP de GitHub porque parece la forma *correcta* de darle a Claude acceso a GitHub — herramientas tipadas, un servidor oficial, el protocolo del que todo el mundo habla. Una semana después lo miras de reojo preguntándote si de verdad está aportando algo, o si `gh pr list` habría hecho lo mismo sin tanta ceremonia.

Respuesta honesta, para GitHub en concreto: normalmente gana el CLI. Pero el MCP sí merece la pena en otros casos. Aquí va el trade-off y la regla a la que llegué.

## Qué te da realmente un MCP

- **Una interfaz de herramientas tipada, reutilizable entre clientes.** El mismo servidor funciona en Claude Code, Cursor, la app web. Lo defines una vez y lo usas en todos.
- **Salida estructurada.** Las herramientas devuelven datos con forma de JSON, no texto libre que tienes que parsear rezando para que el formato no cambie.
- **Auth gestionada en el servidor.** Los tokens viven en la config del servidor, no en cada comando que ejecutas.
- **Control de permisos por herramienta.** Cada herramienta se direcciona como `mcp__<server>__<tool>`, así que puedes permitir el lado de lectura y denegar el de escritura — el [post de afinar permisos](/claude-code-afinar-permisos/) entra a fondo en esto.
- **Alcanzar cosas que no tienen un buen CLI.** Notion, Jira, Sentry, un SaaS interno con API web y nada más. Esta es la victoria de verdad.

## Qué te cuesta

- **Superficie de definiciones de herramientas.** Un solo servidor puede traer decenas de herramientas. El MCP de GitHub expone **unas cuarenta** (`mcp__github__*`). Eso es complejidad de selección para el modelo y superficie de permisos para ti, las llames o no.
- **Contexto, en clientes que cargan los esquemas de entrada.** Cuarenta definiciones no son gratis. Claude Code moderno las *difiere* (luego entro en eso), pero no todos los clientes lo hacen.
- **Otro proceso que mantener vivo.** Un servidor que puede colgarse, caerse o quedarse desactualizado.
- **Radio de impacto.** Las herramientas de escritura son reales: `merge_pull_request`, `delete_file`, `create_or_update_file`. Las cómodas son también las peligrosas.
- **Latencia y una capa que se puede romper** entre tú y lo que de verdad intentas hacer.

## El caso de GitHub

El MCP de GitHub expone ~40 herramientas. Claude Code moderno **difiere** sus esquemas — los carga bajo demanda vía tool-search en vez de volcar las cuarenta en el contexto al arrancar. Así que la crítica clásica de "el MCP infla tu ventana de contexto" es real, pero depende de la versión. En cualquier caso, rara vez necesitas cuarenta herramientas de GitHub en una sesión.

Y si tienes una shell con `gh` autenticado, una sola herramienta Bash cubre casi todo — y el modelo ya se sabe `gh` de memoria:

```bash
gh pr list
gh pr view 123
gh pr create --fill
gh issue list --label bug
gh api repos/:owner/:repo/commits
```

Lado a lado, la misma operación:

```text
MCP:  mcp__github__list_pull_requests   ← 1 de ~40 herramientas, necesita el servidor en marcha
CLI:  gh pr list --json number,title,state  ← 1 llamada Bash, JSON estructurado, sin servidor
```

Ambos te dan datos estructurados. El CLI lo hace con una herramienta que el modelo ya tiene, cero esquemas extra, y nada que mantener corriendo.

Así que para GitHub, tira del MCP solo cuando:

1. **Estás en un cliente sin shell** — claude.ai en la web, una integración de escritorio — donde `gh` no es una opción.
2. **Quieres control de permisos por herramienta** — permitir `mcp__github__get_file_contents`, denegar `mcp__github__merge_pull_request`. Eso es más limpio que un patrón de string en Bash, que no es un AST de shell y no distingue de forma fiable una invocación de `git` segura de una peligrosa.
3. **Quieres estructura garantizada** sin acordarte de los campos `--json` correctos.

Fuera de esos tres, `gh` por Bash es la herramienta más ligera.

## La regla de decisión

**Usa un MCP cuando:**

- la capacidad no tiene un CLI maduro (Notion, Jira, Sentry, una API interna);
- estás en un cliente sin shell;
- quieres salida tipada *y* control de permisos por herramienta;
- vas a reutilizar el mismo servidor en varios clientes.

**Prefiere el CLI / sáltate el MCP cuando:**

- ya existe un CLI maduro (`gh`, `aws`, `kubectl`, `psql`) **y** tienes shell;
- el servidor expone una superficie enorme de la que usarás el 5%;
- el presupuesto de tokens importa y tu cliente no difiere los esquemas.

## Impacto

- Quité el MCP de GitHub en mi setup con shell. `gh` por Bash cubre prácticamente todo — menos herramientas en juego, menos herramientas de escritura que vigilar, un servidor menos que cuidar.
- Mantuve el MCP para el SaaS de mi stack que no tiene CLI ninguno. Ahí es exactamente donde aporta algo que la shell no puede.

## Decisiones

- **Por defecto, el CLI cuando existe uno bueno y tengo shell.** Es la herramienta más pequeña y legible.
- **MCP para clientes sin shell y para el control de permisos por herramienta.** Esos son los casos que el CLI no cubre de verdad.
- **Vigilar las herramientas de escritura con reglas de deny pase lo que pase** — ver el [post de afinar permisos](/claude-code-afinar-permisos/).

## Limitaciones

- **La carga diferida de herramientas depende de versión y cliente.** En clientes que cargan todos los esquemas al arrancar, el argumento de tokens contra los MCP grandes pesa mucho más.
- **La salida de `gh` es texto libre por defecto** — tiras de `--json`/`--jq` para tener estructura, mientras que una herramienta MCP te la entrega tipada.
- **El deny por herramienta es más limpio con MCP que con Bash.** `deny mcp__github__merge_pull_request` es exacto; un patrón de Bash es un match de string, no un entendimiento del comando.
- **La regla se da la vuelta entera para capacidades sin CLI.** Ahí "sáltate el MCP" no está sobre la mesa — el MCP *es* la vía de acceso, y el trade-off pasa a ser qué herramientas habilitar, no si usarlo o no.
