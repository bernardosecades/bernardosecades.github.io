{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2027-01-19

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [Ir detrás de la IA no escala: el trabajo de un Agentic Engineer es el harness del equipo](https://claudecodesessions.com/agentic-engineer-harness-de-equipo/)

Muchos equipos generan el doble de código con IA y lo siguen revisando línea a línea, con alguien senior bloqueado leyendo PRs de miles de líneas. Se desarrolla más rápido, pero el tiempo ahorrado se va a revisar, y la productividad no sube tanto como se esperaba. En este post, más largo de lo normal, cuento el piloto que estamos haciendo en mi equipo: specs revisadas antes del código, un test por escenario, revisores automáticos antes que una persona y cada fallo llevado al harness para que el siguiente no lo repita.

*Lectura: 15 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [Chasing the AI doesn't scale: an agentic engineer's job is the team's harness](https://claudecodesessions.com/agentic-engineer-team-harness/)

A lot of teams generate twice as much code with AI and still review it line by line, with a senior stuck reading thousand-line PRs. Development is faster, but the time saved moves into review, and productivity doesn't go up as much as expected. In this longer-than-usual post I walk through the pilot we're running on my team: specs reviewed before any code, one test per scenario, automated reviewers before a person, and every failure taken into the harness so the next person doesn't repeat it.

*Read: 15 min.*

Bernardo

{% endif %}
