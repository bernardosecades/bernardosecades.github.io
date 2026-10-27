{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2026-11-10

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [La spec vivía en un repo y el cambio tocaba tres](https://claudecodesessions.com/openspec-un-repo-para-todas-las-specs/)

Hace un mes defendí que la spec va en el repo del servicio dueño. Luego el equipo empezó a usar OpenSpec y casi todos los cambios tocaban varios servicios, así que los revisores de los otros repos nunca veían los escenarios. Movimos todas las specs a un repo propio. Es el primero de cuatro posts sobre cómo estamos aplicando OpenSpec en equipo.

*Lectura: 5 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [The spec lived in one repo and the change touched three](https://claudecodesessions.com/openspec-one-repo-for-all-specs/)

A month ago I argued the spec belongs in the repo of the owning service. Then my team adopted OpenSpec, most changes touched several services, and reviewers in the other repos never saw the scenarios. We moved every spec into a repo of its own. This is the first of four posts on how we're applying OpenSpec as a team.

*Read: 5 min.*

Bernardo

{% endif %}
