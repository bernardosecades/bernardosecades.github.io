---
title: "Claude Code: notificar bloqueos de sesiones usando hooks"
date: 2026-06-03 00:00:00
lang: es
ref: claude-code-hooks-notification
tags: [claude-code, hooks, notifications, settings, workflow]
read_min: 5
excerpt_text: "Claude ya te avisa cuando necesita tu atención — pero la notificación por defecto se desvanece en unos segundos. Ten tres sesiones en tres tareas y te perderás la que se bloqueó. Un hook lanza una alerta determinista y persistente que nombra la rama y el proyecto, para que la sesión atascada no se pueda esconder."
---

Cuando trabajo suelo tener **varias sesiones de Claude Code abiertas a la vez** — una refactorizando un servicio, otra escribiendo tests, otra cazando un bug — cada una en su terminal y en su rama. Corren solas mientras hago otra cosa, y cada poco una se bloquea: un diálogo de permisos, o termina y se queda esperando mi siguiente instrucción.

Claude *sí* te avisa cuando eso pasa — lanza una notificación de escritorio. El problema es que la notificación por defecto **se desvanece a los pocos segundos**. Si estoy metido en otra ventana cuando salta, me la pierdo, y esa sesión se queda ahí parada. Con una sola sesión te acabarías dando cuenta. Con tres o cuatro repartidas en tareas distintas, ese "acabarías" se convierte en diez minutos perdidos en la sesión que olvidaste que esperaba.

Lo resolví con un **hook**: una notificación persistente e imposible de perder que nombra la rama y el proyecto, para que en cuanto cualquier sesión me necesite sepa *cuál* es. Este post va sobre todo de ese ejemplo — los hooks en general los toco muy por encima.

## Los hooks, en un párrafo

Un hook es un **comando de shell que el harness ejecuta en un punto fijo de la sesión**, configurado en `settings.json`. La palabra clave es *determinista*: lo ejecuta el runtime, no el modelo, así que se dispara **siempre** que ocurre el evento — sin posibilidad de que Claude "se olvide" o decida que no hace falta. Ahí está toda la gracia. Hay eventos para antes/después de ejecutar una herramienta, al enviar un prompt, cuando Claude para, cuando arranca o termina una sesión — pero el que me importa aquí es **`Notification`**, que se dispara justo cuando Claude necesita tu atención.

## El hook

Esto vive en `~/.claude/settings.json` (a nivel usuario, así que aplica a todos los proyectos):

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "RAMA=$(git -C \"${CLAUDE_PROJECT_DIR:-$PWD}\" rev-parse --abbrev-ref HEAD 2>/dev/null || echo '(sin rama)'); PROYECTO=$(basename \"$PWD\"); notify-send -u critical \"⚠️ CLAUDE NEEDS YOUR ATTENTION ⚠️\" \"\n\n\n👤 Hi $USER,\n\nBranch 🌿 $RAMA requires your attention.\n\nProject: 📁 $PROYECTO\n\nCheck the terminal 🤖\n\n\n\""
          }
        ]
      }
    ]
  }
}
```

Y esto es lo que aparece en pantalla — una notificación crítica y pegajosa, encima de lo que esté haciendo, en el instante en que una sesión se bloquea:

![Notificación de escritorio lanzada por un hook de Notification de Claude Code: "CLAUDE NEEDS YOUR ATTENTION", con el usuario, la rama y el proyecto](/assets/screenshots/demo-hooks-notification.png)

Las dos cosas que la hacen funcionar para el caso multi-sesión:

- **Es persistente.** `-u critical` es el único flag que importa: por la spec de freedesktop, el demonio de escritorio nunca auto-expira las notificaciones críticas, así que esperan hasta que las cierro — al contrario que la de por defecto, que se desvanece en segundos. No hace falta ningún flag de timeout; la urgencia crítica por sí sola la mantiene en pantalla.
- **Nombra la sesión.** `RAMA` lee la rama actual y `PROYECTO` la carpeta del proyecto, interpolados en el cuerpo. Cuando hay cinco terminales corriendo, la alerta me dice *cuál* me necesita sin revisarlas una a una. `CLAUDE_PROJECT_DIR` es una variable de entorno que el harness exporta a cada hook, apuntando a la raíz del proyecto; `:-$PWD` es el fallback, y `|| echo '(sin rama)'` evita que falle fuera de un repo git.

`matcher: ""` solo significa "dispara en cada notificación" — `Notification` no se subdivide por herramienta, así que vacío es lo normal.

Pega el bloque, guarda y arranca una sesión **nueva** — el harness recoge los cambios de hooks al inicio de sesión, así que una sesión en marcha no verá la edición hasta que la reinicies. En macOS, cambia el comando por `osascript -e 'display notification "Cuerpo" with title "Título" sound name "Glass"'`; la estructura del hook es idéntica.

## Impacto

- El tiempo muerto entre "una sesión se bloqueó" y "me di cuenta" pasó de minutos a segundos — y se mantiene así por mucho que tarde en volver, porque la alerta no se desvanece.
- Correr tres o cuatro sesiones en tareas distintas se volvió práctico de verdad. La línea de rama + proyecto hace que salte directo a la terminal correcta en vez de ir rotando por todas.
- Dejé de *vigilar* terminales por completo. La garantía determinista — se dispara siempre — es lo que me permite fiarme de eso y cambiar de contexto del todo.

## Decisiones técnicas

- **Un hook, no un hábito.** Echarle ojo a la terminal es un *pull* que falla en cuanto estás absorto en otra cosa. Un hook es un *push*, y como el harness lo ejecuta de forma determinista nunca se salta en silencio.
- **`-u critical` y nada más.** La urgencia crítica es lo único que la spec garantiza que no se auto-expira, y basta por sí sola — quité el flag `-t 0` porque es redundante (y en urgencia normal demonios comunes como GNOME Shell lo ignoran igual). Un solo flag hace todo el trabajo.
- **Rama + proyecto en el cuerpo.** Sin ellos la alerta es inútil cuando manejas sesiones — "Claude necesita atención" pero *¿cuál?*. Las dos lecturas `$(...)` son la razón de que el hook escale más allá de una terminal.
- **A nivel usuario, no de proyecto.** Es ergonomía personal, no algo que mis compañeros deban heredar, así que vive en `~/.claude/settings.json` y no en los settings versionados del repo.

## Limitaciones reales

- **`notify-send` es solo Linux.** La *estructura* del hook es portable pero el comando no — macOS necesita `osascript`, y una máquina sin pantalla (CI, SSH sin display) no tiene escritorio al que notificar.
- **No te dice *por qué* paró una sesión.** `Notification` se dispara tanto por un diálogo de permisos como por una espera; la alerta dice "necesita atención", no "quiere ejecutar `rm`". Te lleva rápido a la terminal correcta — sigues teniendo que leerla para actuar.
- **Recogida al inicio de sesión.** Editar el hook no afecta a una sesión que ya está corriendo; reinicias para cargar los cambios.
- **Es fire-and-forget.** Un hook de `Notification` solo observa y avisa — no puede bloquear ni cambiar lo que hace Claude. Moldear el comportamiento es para lo que están los otros eventos (`PreToolUse` y compañía).

Los hooks son la capa determinista debajo de un asistente por lo demás probabilístico. Este es el sitio más suave para empezar: no cambia nada de cómo funciona Claude, y te devuelve cada minuto que perdías con una sesión esperando en silencio por una notificación que nunca viste.
