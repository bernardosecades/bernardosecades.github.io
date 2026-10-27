{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2026-11-24

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [La IA escribía specs que parecían código. Lo arregló un config.yaml](https://claudecodesessions.com/openspec-reglas-config-specs-ia/)

Nuestras primeras specs generadas con IA nombraban funciones, convertían en requisito un bug que manteníamos a propósito y tenían escenarios sin test. El config.yaml de OpenSpec le da a la IA las mismas reglas en cada artefacto. Cuento las nuestras y el fallo que hay detrás de cada una. Segundo post de la serie sobre OpenSpec en equipo.

*Lectura: 5 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [The AI wrote specs that read like code. A config.yaml fixed it](https://claudecodesessions.com/openspec-config-rules-for-ai-specs/)

Our first AI-generated specs named functions, turned a bug we were keeping into a requirement, and had scenarios no test covered. OpenSpec's config.yaml gives the AI the same rules for every artifact. Here are ours and the mistake behind each one. Second post in the series on OpenSpec as a team.

*Read: 5 min.*

Bernardo

{% endif %}
