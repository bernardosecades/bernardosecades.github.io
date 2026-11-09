{# ---------------------------------------------------------------
   Digest — AI Tips & Tricks (EN / ES) — 2027-01-05

   Subject line (Jinja):
     {% if subscriber.metadata.lang == "es" %}Tu dosis de IA ✨{% else %}Your dose of AI tips ✨{% endif %}
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo en [AI Tips & Tricks](https://claudecodesessions.com/):

### [Arregla el harness, no el prompt: dónde poner cada regla para que tu agente la cumpla](https://claudecodesessions.com/arregla-el-harness-no-el-prompt/)

Tenía en el CLAUDE.md una regla para que el agente no editara `vendor/` a mano. Casi siempre la cumplía, y las pocas veces que no, el parche desaparecía en el siguiente `go mod vendor`. Reescribirla no sirvió. En el post explico qué es el harness, la escalera que uso para decidir dónde vive cada regla y el hook que acabé montando.

*Lectura: 5 min.*

Bernardo

{% else %}

Hi,

Here's what's new on [AI Tips & Tricks](https://claudecodesessions.com/):

### [Fix the harness, not the prompt: where each rule should live so your agent follows it](https://claudecodesessions.com/fix-the-harness-not-the-prompt/)

I had a rule in CLAUDE.md telling the agent not to hand-edit `vendor/`. It followed it almost every time, and when it didn't, the patch vanished on the next `go mod vendor`. Rewording it didn't help. The post covers what the harness is, the ladder I use to decide where each rule lives, and the hook I ended up building.

*Read: 5 min.*

Bernardo

{% endif %}
