---
layout: default
title: "Sobre mí"
permalink: /es/about/
lang: es
nav: about
about_ref: true
---
<div class="container">
  <section class="about-head">
    <div class="avatar-lg">BS</div>
    <h1>Bernardo Secades</h1>
    <p class="about-tagline">Ingeniero backend — Go y sistemas distribuidos. Construyo con IA en producción y escribo sobre lo que de verdad funciona.</p>
  </section>
  <div class="about-body">
    <p>Software engineer especializado en microservicios backend en Go. Me apasionan los sistemas distribuidos: el diseño de servicios, la comunicación asíncrona y la búsqueda del equilibrio entre simplicidad y resiliencia.</p>
    <p>Tengo Claude Code integrado en mi flujo de trabajo diario — está abierto mientras programo. Lo uso para razonar sobre diseños de servicios, detectar casos borde sutiles durante revisiones de código, redactar planes de migración y acelerar las partes más mecánicas de Go para poder dedicar más tiempo a las restricciones interesantes.</p>
    <p>Más allá del uso personal, estoy trabajando en la adopción a nivel cross-team: compartiendo patrones que funcionan, haciendo demos y definiendo convenciones comunes para que todo el equipo pueda obtener valor consistente del desarrollo asistido por IA, sin que cada persona tenga que descubrirlo por su cuenta.</p>
  </div>

  {%- comment -%} Lista best-first; muestra los 3 primeros ya publicados. Los posts flagship de IA aplicada van primero y ascienden solos según se publican cada semana; los posts de tooling ya publicados rellenan. {%- endcomment -%}
  {%- assign start_refs = "reviewing-ai-generated-code,choosing-the-right-model,ai-nearly-shipped-a-bug,multi-agent-when-worth-it,design-with-ai-interlocutor,debugging-with-ai,claude-code-team-marketplace,mcp-when-to-use,claude-code-multi-repo-workspace,claude-code-subagents" | split: "," -%}
  {%- assign lang_posts = site.posts | where: "lang", page.lang -%}
  {%- assign now_ts = site.time | date: '%s' | plus: 0 -%}
  {%- assign shown = 0 -%}
  <section class="about-start" aria-labelledby="start-heading">
    <div class="section-label" id="start-heading">Para empezar</div>
    <div class="feat-grid">
      {% for ref in start_refs %}{%- if shown < 3 -%}
        {%- assign fp = lang_posts | where: "ref", ref | first -%}
        {%- if fp -%}{%- assign post_ts = fp.date | date: '%s' | plus: 0 -%}{%- if post_ts <= now_ts -%}
        <a class="feat-card" href="{{ fp.url | relative_url }}">
          <div class="feat-card-body">
            <h2>{{ fp.title }}</h2>
            {% if fp.excerpt_text %}<p>{{ fp.excerpt_text }}</p>{% endif %}
          </div>
        </a>
        {%- assign shown = shown | plus: 1 -%}
        {%- endif -%}{%- endif -%}
      {%- endif -%}{% endfor %}
    </div>
    <p class="about-start-more"><a href="/es/systems/">Ver todos los sistemas que he construido →</a></p>
  </section>

  <section class="about-certs" aria-labelledby="certs-heading">
    <h2 id="certs-heading" class="about-certs-title">Certificaciones (18)</h2>
    <ul class="cert-list">
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/us429rbhizpg" target="_blank" rel="noopener">Claude Code in Action</a>
          <span class="cert-sub">Anthropic · Mar 2026 · <a href="https://verify.skilljar.com/c/us429rbhizpg" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/yf83vww2p9bj" target="_blank" rel="noopener">Introduction to Model Context Protocol</a>
          <span class="cert-sub">Anthropic · Abr 2026 · <a href="https://verify.skilljar.com/c/yf83vww2p9bj" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/fwr55oo2jq2r" target="_blank" rel="noopener">Model Context Protocol: Advanced Topics</a>
          <span class="cert-sub">Anthropic · Abr 2026 · <a href="https://verify.skilljar.com/c/fwr55oo2jq2r" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/wghxtwj2hrin" target="_blank" rel="noopener">Introduction to Agent Skills</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/wghxtwj2hrin" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/gsdr8gpt8fbm" target="_blank" rel="noopener">AI Fluency for Educators</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/gsdr8gpt8fbm" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/4bt8c9b6cg6j" target="_blank" rel="noopener">Claude 101</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/4bt8c9b6cg6j" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/ext27cxbwbuo" target="_blank" rel="noopener">Claude Code 101</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/ext27cxbwbuo" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/99myh8tt4tv6" target="_blank" rel="noopener">Introduction to Subagents</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/99myh8tt4tv6" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/imnudjtmig57" target="_blank" rel="noopener">AI Fluency for Students</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/imnudjtmig57" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/d9ih84sgvups" target="_blank" rel="noopener">AI Fluency for Nonprofits</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/d9ih84sgvups" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/m7awsup6jyuv" target="_blank" rel="noopener">Introduction to Claude Cowork</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/m7awsup6jyuv" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/bqnq2nedqrok" target="_blank" rel="noopener">AI Capabilities and Limitations</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/bqnq2nedqrok" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/mzks2rtjcbug" target="_blank" rel="noopener">Teaching AI Fluency</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/mzks2rtjcbug" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/f5ofn8sm4whf" target="_blank" rel="noopener">AI Fluency: Framework & Foundations</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/f5ofn8sm4whf" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/6eb4im8h9232" target="_blank" rel="noopener">AI Fluency for Small Businesses</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/6eb4im8h9232" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/b3b4xm8nfb5p" target="_blank" rel="noopener">Building with the Claude API</a>
          <span class="cert-sub">Anthropic · Jun 2026 · <a href="https://verify.skilljar.com/c/b3b4xm8nfb5p" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/569vpbp8dod3" target="_blank" rel="noopener">Claude with Amazon Bedrock</a>
          <span class="cert-sub">Anthropic · Jun 2026 · <a href="https://verify.skilljar.com/c/569vpbp8dod3" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/2ienkt4jhsxb" target="_blank" rel="noopener">Claude with Google Cloud's Vertex AI</a>
          <span class="cert-sub">Anthropic · Jun 2026 · <a href="https://verify.skilljar.com/c/2ienkt4jhsxb" target="_blank" rel="noopener">Verificar ↗</a></span>
        </div>
      </li>
    </ul>
  </section>

  <div class="about-socials">
    <a href="https://linkedin.com/in/bernardosecades" target="_blank" rel="noopener">LinkedIn</a>
    <a href="https://github.com/bernardosecades" target="_blank" rel="noopener">GitHub</a>
    <a href="mailto:bernardosecades@gmail.com">Email</a>
    <a href="/es/#newsletter">Boletín</a>
    <a href="{{ '/feed.xml' | relative_url }}">RSS</a>
  </div>
</div>
