---
title: "Claude Code: alcance vs permisos"
date: 2026-05-10 00:00:00
lang: es
ref: claude-code-scope-vs-permissions
tags: [claude-code, settings, permissions, workflow]
read_min: 3
excerpt_text: "El working directory decide qué puede leer Claude. Las acciones — ejecutar comandos, escribir ficheros, llamar herramientas MCP — se rigen por .claude/settings.json. Dos capas distintas, fáciles de confundir."
---

El [post anterior sobre workspace multi-repo](/claude-code-workspace-multi-repo/) presentaba el working directory como lo que Claude puede ver. Esa es sólo la mitad de la foto: cubre **lecturas**. La otra mitad — **acciones** como ejecutar un comando Bash, escribir un fichero o llamar a una herramienta MCP — se rige por otra cosa: `.claude/settings.json`.

Dos capas distintas. Confundirlas es lo que lleva a preguntas como *"el folder está dentro del alcance, ¿por qué Claude me pide confirmación antes de hacer `rm`?"*, o lo contrario, *"añadí la ruta y ahora está corriendo migraciones por su cuenta."*

## Lecturas vs acciones

- **Working directory** (y `--add-dir`) → lo que Claude *puede ver*.
- **`.claude/settings.json`** → lo que Claude *puede hacer*.

Un fichero dentro del working directory es técnicamente alcanzable. Que Claude pueda *escribirlo*, o ejecutar un comando que lo toque, sigue pasando por las reglas de `settings.json`.

## Dónde vive settings.json

El fichero se apila en varios sitios:

- **Usuario** — `~/.claude/settings.json`. Tus valores por defecto para todos los proyectos.
- **Proyecto (compartido)** — `<repo>/.claude/settings.json`. Se commitea al repo y aplica a quien trabaje en él.
- **Proyecto (local)** — `<repo>/.claude/settings.local.json`. No se commitea (está gitignored por defecto), para tus ajustes personales del proyecto.
- **Enterprise gestionado** — distribuido centralmente por tu organización, máxima prioridad.

Las capas se apilan. Una regla `deny` siempre gana, así que un deny a nivel proyecto sobrescribe un allow a nivel usuario.

## Un ejemplo mínimo

```json
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(go test:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Read(./.env)",
      "Edit(./prod-config.yaml)"
    ]
  }
}
```

`allow` se ejecuta sin pedir confirmación. `deny` está bloqueado de forma dura. Lo que no encaja en ninguna de las dos cae en el flujo por defecto (Claude pregunta).

<video controls playsinline>
  <source src="/assets/videos/demo-settings-permissions.webm" type="video/webm">
</video>

En la demo Claude se niega a leer `.env` por la regla `deny`. Si de verdad quieres ese contenido en el contexto, el [atajo `!`](/claude-code-atajo-bang/) es la salida: `! cat .env` corre en tu shell y la salida cae en la conversación. El `deny` aplicaba a la lectura autónoma de Claude; que tú ejecutes el comando es una decisión aparte.

## Por qué importa

La regla `Read(./.env)` del ejemplo es la ilustración más clara: el fichero vive dentro del working directory, así que por la regla de alcance de lectura estaría disponible. El `deny` de `settings.json` lo anula. El alcance dice *"podrías llegar a esto"*; `settings.json` dice *"no, no puedes"*.

Lo mismo con Bash. Que el folder esté dentro del alcance le dice a Claude *dónde* corren los comandos. `settings.json` le dice *qué* comandos están preaprobados.

Si te ves aprobando `Bash(go test:*)` diez veces al día, esa es la señal para añadirlo a `allow`. Si hay un comando que nunca quieres que Claude ejecute por su cuenta, esa es la señal para añadirlo a `deny`.

El modelo mental que me funciona: el working directory es el *mapa* de por dónde Claude puede moverse; `settings.json` es el *reglamento* de lo que tiene permitido hacer una vez que está allí.
