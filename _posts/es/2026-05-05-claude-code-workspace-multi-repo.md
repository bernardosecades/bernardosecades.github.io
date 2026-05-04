---
title: "Claude Code: un workspace para varios repos"
date: 2026-05-05 00:00:00
lang: es
ref: claude-code-multi-repo-workspace
tags: [claude-code, workflow]
read_min: 3
excerpt_text: "Lanza Claude en el padre de tus repos para que un ticket cross-service deje de ser N sesiones aisladas — y deja que un CLAUDE.md del workspace describa cómo encajan las piezas."
---

En un monolito sólo trabajas en un folder, así que no hay nada que pensar. En multi-repo es distinto: un mismo ticket suele tocar un microservicio y una librería compartida que vive en una carpeta hermana, a veces usada por otros servicios.

Mi estructura es esta:

```text
~/work/
├── microservice-a/
├── microservice-b/
├── library-core/
└── library-x/
```

## 1. Lanza Claude desde el padre

```bash
cd ~/work
claude
```

Y ya está. Todos los subfolders bajo `~/work/` están dentro del alcance — no hace falta "abrir" ni "añadir" nada por microservicio. Eso no significa que Claude los lea ni los indexe automáticamente al arrancar: sólo toca un archivo cuando tú lo mencionas (`@path`) o cuando el modelo decide leerlo durante la tarea. Si el ticket toca `microservice-a` y `library-core`, ambos ya están disponibles para cuando haga falta.

La flag `--add-dir` sólo entra en juego cuando necesitas una carpeta **fuera** del directorio de trabajo:

```bash
claude --add-dir /alguna/ruta/externa
```

## 2. `@path` no amplía el alcance, fuerza una lectura

Aquí es donde veo que la gente se atasca. `@microservice-a/src/foo.ts` **no** cambia los permisos ni el alcance: lo que hace es forzar la lectura de ese archivo e incluirlo en el contexto en ese momento. Lo mismo con `@microservice-a/` para meter un listado del directorio.

Dicho de otra forma:

- El working directory (y `--add-dir`) define **qué puede leer** Claude.
- `@path` define **qué lee ahora mismo**, sin esperar a que el modelo decida ir a buscarlo.

## 3. Apila CLAUDE.md por alcance

El esquema de dos niveles que me funciona:

- **`~/work/CLAUDE.md`** — el mapa del workspace. Qué es cada folder, qué servicio usa qué librería, contratos compartidos, nombres. Es el `CLAUDE.md` del directorio donde lanzaste Claude, así que entra en el contexto inicial.
- **`~/work/<servicio>/CLAUDE.md`** — notas específicas del servicio (comandos de build, setup de tests, peculiaridades). Como viven en subdirectorios, Claude tiende a incorporarlos cuando empieza a trabajar dentro de ese servicio. No es algo determinista paso a paso: depende de cómo navegue durante la tarea.

La ganancia: el contexto inicial se queda ligero (sólo el mapa del workspace) y lo específico de cada servicio aparece cuando el trabajo aterriza ahí.

`~/.claude/CLAUDE.md` sigue aplicando por encima de todo esto para preferencias personales que valen para cualquier proyecto.

## Por qué lo hago

Un ticket cross-service deja de ser N sesiones aisladas en las que tengo que volver a explicar cómo encajan las piezas. Las dependencias entre servicios viven explícitamente en el `CLAUDE.md` del workspace, y Claude carga las reglas propias de cada servicio sólo cuando el trabajo aterriza ahí.
