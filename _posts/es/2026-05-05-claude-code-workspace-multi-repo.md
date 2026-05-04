---
title: "Claude Code: un workspace para varios repos"
date: 2026-05-05 00:00:00
lang: es
ref: 3
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

Y ya está. Todos los subfolders bajo `~/work/` son accesibles para Claude — no hace falta "abrir" ni "añadir" nada por microservicio. Si un ticket toca `microservice-a` y `library-core`, ambos ya están en el alcance.

La flag `--add-dir` sólo entra en juego cuando necesitas una carpeta **fuera** del directorio de trabajo:

```bash
claude --add-dir /alguna/ruta/externa
```

## 2. `@path` es una referencia, no un permiso

Aquí es donde veo que la gente se atasca. Escribir `@microservice-a/src/foo.ts` en el prompt **no** le da acceso a Claude a ese archivo — sólo inyecta su contenido en la conversación como atajo. Lo mismo con `@microservice-a/` para inyectar un listado del directorio.

El acceso lo decide el working directory (y `--add-dir`). `@` es sólo una forma de meter algo más rápido que pedirle a Claude que lo vaya a leer.

## 3. Apila CLAUDE.md por alcance

El esquema de dos niveles que me funciona:

- **`~/work/CLAUDE.md`** — el mapa del workspace. Qué es cada folder, qué servicio usa qué librería, contratos compartidos, nombres. Se carga al arrancar porque está en el working directory.
- **`~/work/<servicio>/CLAUDE.md`** — notas específicas del servicio (comandos de build, setup de tests, peculiaridades). Como viven en subdirectorios, se cargan **bajo demanda** la primera vez que Claude lee un archivo de ese servicio.

La ganancia: el contexto inicial se queda ligero (sólo el mapa del workspace) y lo específico de cada servicio sólo se carga cuando Claude entra ahí.

`~/.claude/CLAUDE.md` sigue aplicando por encima de todo esto para preferencias personales que valen para cualquier proyecto.

## Por qué lo hago

Un ticket cross-service deja de ser N sesiones aisladas en las que tengo que volver a explicar cómo encajan las piezas. Las dependencias entre servicios viven explícitamente en el `CLAUDE.md` del workspace, y Claude carga las reglas propias de cada servicio sólo cuando el trabajo aterriza ahí.
