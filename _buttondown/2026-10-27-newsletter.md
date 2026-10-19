{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2026-10-27

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [El ticket también da órdenes](https://claudecodesessions.com/prompt-injection-corta-la-salida/)

Le paso tickets de Jira al agente para que los implemente. Un día me pregunté qué pasaría si el ticket traía instrucciones, y la respuesta no me gustó: mi setup le dejaba leer secretos y mandarlos fuera. Lo que cambié fue quitarle la salida, con unas pocas líneas en el settings.json.

*Lectura: 3 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [The ticket gives orders too](https://claudecodesessions.com/prompt-injection-cut-the-way-out/)

I hand Jira tickets to my coding agent and tell it to implement them. One day I wondered what would happen if a ticket contained instructions, and I didn't like the answer: my setup let it read secrets and send them out. So I took away the way out, with a few lines in settings.json.

*Read: 3 min.*

Bernardo

{% endif %}
