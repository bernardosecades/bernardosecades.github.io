{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2026-12-08

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [Escribimos la spec de un código que ya funcionaba. Fallaron dos tests](https://claudecodesessions.com/openspec-spec-antes-del-refactor/)

Antes de un refactor que no debía cambiar nada, escribimos con OpenSpec la spec del comportamiento actual y pasamos sus escenarios a tests contra una base de datos real. Sobre develop fallaron dos, y eran dos bugs que nadie había reportado. Tercer post de la serie sobre OpenSpec en equipo.

*Lectura: 6 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [We wrote a spec for code that already worked. Two tests failed](https://claudecodesessions.com/openspec-spec-before-refactor/)

Before a refactor that wasn't supposed to change anything, we wrote an OpenSpec spec of the current behaviour and turned its scenarios into tests against a real database. On develop, two failed, and they were two bugs nobody had reported. Third post in the series on OpenSpec as a team.

*Read: 6 min.*

Bernardo

{% endif %}
