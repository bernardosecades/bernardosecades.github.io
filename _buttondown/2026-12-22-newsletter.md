{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2026-12-22

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [Un change no termina con el merge. Lo archivamos después de producción](https://claudecodesessions.com/openspec-archivar-tras-produccion/)

El archive de OpenSpec pasa los escenarios de un change a las specs que describen el sistema. Si lo lanzas al hacer merge, esas specs describen develop y no producción. Cuento el ciclo completo de un change en el equipo y el error que nos hizo dejar el archive para el final. Con este post cierro la serie sobre OpenSpec en equipo.

*Lectura: 5 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [A change isn't done when the code merges. We archive it after production](https://claudecodesessions.com/openspec-archive-after-prod/)

OpenSpec's archive moves a change's scenarios into the specs that describe the system. Run it at merge time and those specs describe develop, not production. Here's the full lifecycle of a change in our team and the mistake that made us put the archive last. This post closes the series on OpenSpec as a team.

*Read: 5 min.*

Bernardo

{% endif %}
