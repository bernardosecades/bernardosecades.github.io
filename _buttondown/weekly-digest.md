{# ---------------------------------------------------------------
   Weekly digest — AI Tips & Tricks (EN / ES)
   Compatible con Buttondown (Markdown + plantillas Jinja).

   Cómo usar:
   1. En Buttondown: New email → pega el contenido de abajo (sin estas
      líneas de comentario, aunque Jinja {# … #} también las elimina).
   2. Subject line (también puede llevar Jinja):
        {% if subscriber.metadata.lang == "es" %}Esta semana en AI Tips & Tricks{% else %}This week in AI Tips & Tricks{% endif %}
   3. Rellena los bloques POST_N con los posts de esta semana.
      Si solo hay un post, borra los bloques sobrantes.
   4. Cada subscriber debe tener metadata.lang = "es" o "en".
      Fallback: si no hay metadata, se envía en inglés.

   Notas Jinja:
   - {# ... #} = comentario (no se renderiza).
   - {{ ... }} = variable.
   - {% ... %} = lógica.
--------------------------------------------------------------- #}

{% if subscriber.metadata.lang == "es" %}

Hola,

Esto es lo nuevo esta semana en [AI Tips & Tricks](https://claudecodesessions.com/):

### [POST_1_TITULO_ES](https://claudecodesessions.com/POST_1_SLUG_ES/)

POST_1_EXCERPT_ES

*Lectura: POST_1_READ_MIN min.*

---

### [POST_2_TITULO_ES](https://claudecodesessions.com/POST_2_SLUG_ES/)

POST_2_EXCERPT_ES

*Lectura: POST_2_READ_MIN min.*

---

### [POST_3_TITULO_ES](https://claudecodesessions.com/POST_3_SLUG_ES/)

POST_3_EXCERPT_ES

*Lectura: POST_3_READ_MIN min.*

— Bernardo

{% else %}

Hi,

Here's what's new this week on [AI Tips & Tricks](https://claudecodesessions.com/):

### [POST_1_TITLE_EN](https://claudecodesessions.com/POST_1_SLUG_EN/)

POST_1_EXCERPT_EN

*Read: POST_1_READ_MIN min.*

---

### [POST_2_TITLE_EN](https://claudecodesessions.com/POST_2_SLUG_EN/)

POST_2_EXCERPT_EN

*Read: POST_2_READ_MIN min.*

---

### [POST_3_TITLE_EN](https://claudecodesessions.com/POST_3_SLUG_EN/)

POST_3_EXCERPT_EN

*Read: POST_3_READ_MIN min.*

— Bernardo

{% endif %}
