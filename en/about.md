---
layout: default
title: "About"
permalink: /en/about/
lang: en
nav: about
about_ref: true
---
<div class="container">
  <section class="about-head">
    {%- assign avatar_img = site.static_files | where: "path", "/assets/img/avatar.jpg" | first -%}
    {% if avatar_img %}<img class="avatar-lg avatar-photo" src="{{ '/assets/img/avatar.jpg' | relative_url }}" alt="Bernardo Secades" width="64" height="64">{% else %}<div class="avatar-lg">BS</div>{% endif %}
    <h1>Bernardo Secades</h1>
    <p class="about-tagline">Backend engineer — Go &amp; distributed systems. I build with AI in production and write about what actually works.</p>
  </section>
  <div class="about-body">
    <p>Software engineer building backend microservices in Go. I focus on distributed systems: service boundaries, event-driven communication, and finding the right balance between simplicity and resilience.</p>
    <p>I have Claude Code wired into my daily workflow — it's open whenever I code. I use it for reasoning through service designs, catching subtle edge cases during code review, drafting migration plans, and speeding up the boilerplate-heavy parts of Go so I can spend more time on the interesting constraints.</p>
    <p>Beyond personal use, I've been working on cross-team adoption: sharing patterns that work, running demos, and building shared conventions so the whole team can get consistent value out of AI-assisted development without each person reinventing the wheel.</p>
  </div>

  {%- comment -%} Best-first list; render the first 3 that are already published. Flagship applied-AI posts lead and auto-promote as they go live weekly; live tooling posts backfill. {%- endcomment -%}
  {%- assign start_refs = "reviewing-ai-generated-code,choosing-the-right-model,ai-nearly-shipped-a-bug,multi-agent-when-worth-it,design-with-ai-interlocutor,debugging-with-ai,claude-code-team-marketplace,mcp-when-to-use,claude-code-multi-repo-workspace,claude-code-subagents" | split: "," -%}
  {%- assign lang_posts = site.posts | where: "lang", page.lang -%}
  {%- assign now_ts = site.time | date: '%s' | plus: 0 -%}
  {%- assign shown = 0 -%}
  <section class="about-start" aria-labelledby="start-heading">
    <div class="section-label" id="start-heading">Start here</div>
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
    <p class="about-start-more"><a href="/en/systems/">See all the systems I've built →</a></p>
  </section>

  <section class="about-certs" aria-labelledby="certs-heading">
    <h2 id="certs-heading" class="about-certs-title">Certifications</h2>
    <h3 class="cert-group-title">Anthropic (19)</h3>
    <ul class="cert-list">
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/us429rbhizpg" target="_blank" rel="noopener">Claude Code in Action</a>
          <span class="cert-sub">Anthropic · Mar 2026 · <a href="https://verify.skilljar.com/c/us429rbhizpg" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/yf83vww2p9bj" target="_blank" rel="noopener">Introduction to Model Context Protocol</a>
          <span class="cert-sub">Anthropic · Apr 2026 · <a href="https://verify.skilljar.com/c/yf83vww2p9bj" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/fwr55oo2jq2r" target="_blank" rel="noopener">Model Context Protocol: Advanced Topics</a>
          <span class="cert-sub">Anthropic · Apr 2026 · <a href="https://verify.skilljar.com/c/fwr55oo2jq2r" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/wghxtwj2hrin" target="_blank" rel="noopener">Introduction to Agent Skills</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/wghxtwj2hrin" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/gsdr8gpt8fbm" target="_blank" rel="noopener">AI Fluency for Educators</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/gsdr8gpt8fbm" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/4bt8c9b6cg6j" target="_blank" rel="noopener">Claude 101</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/4bt8c9b6cg6j" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/ext27cxbwbuo" target="_blank" rel="noopener">Claude Code 101</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/ext27cxbwbuo" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/99myh8tt4tv6" target="_blank" rel="noopener">Introduction to Subagents</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/99myh8tt4tv6" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/imnudjtmig57" target="_blank" rel="noopener">AI Fluency for Students</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/imnudjtmig57" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/d9ih84sgvups" target="_blank" rel="noopener">AI Fluency for Nonprofits</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/d9ih84sgvups" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/m7awsup6jyuv" target="_blank" rel="noopener">Introduction to Claude Cowork</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/m7awsup6jyuv" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/bqnq2nedqrok" target="_blank" rel="noopener">AI Capabilities and Limitations</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/bqnq2nedqrok" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/mzks2rtjcbug" target="_blank" rel="noopener">Teaching AI Fluency</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/mzks2rtjcbug" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/f5ofn8sm4whf" target="_blank" rel="noopener">AI Fluency: Framework & Foundations</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/f5ofn8sm4whf" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/6eb4im8h9232" target="_blank" rel="noopener">AI Fluency for Small Businesses</a>
          <span class="cert-sub">Anthropic · May 2026 · <a href="https://verify.skilljar.com/c/6eb4im8h9232" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/b3b4xm8nfb5p" target="_blank" rel="noopener">Building with the Claude API</a>
          <span class="cert-sub">Anthropic · Jun 2026 · <a href="https://verify.skilljar.com/c/b3b4xm8nfb5p" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/569vpbp8dod3" target="_blank" rel="noopener">Claude with Amazon Bedrock</a>
          <span class="cert-sub">Anthropic · Jun 2026 · <a href="https://verify.skilljar.com/c/569vpbp8dod3" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/2ienkt4jhsxb" target="_blank" rel="noopener">Claude with Google Cloud's Vertex AI</a>
          <span class="cert-sub">Anthropic · Jun 2026 · <a href="https://verify.skilljar.com/c/2ienkt4jhsxb" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">A</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://verify.skilljar.com/c/rfa3fjxg68e7" target="_blank" rel="noopener">AI Fluency for Creative Work</a>
          <span class="cert-sub">Anthropic · Sep 2026 · <a href="https://verify.skilljar.com/c/rfa3fjxg68e7" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
    </ul>
    <h3 class="cert-group-title">The Linux Foundation (2)</h3>
    <ul class="cert-list">
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">L</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/780bbaf3-ad3a-45d8-b216-347cafe54670-bernardo-secades-618d039d-d3b0-44cc-8229-74b138695a3a-certificate.pdf" target="_blank" rel="noopener">Getting Started with OpenTelemetry (LFS148)</a>
          <span class="cert-sub">The Linux Foundation · Jul 2026 · <a href="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/780bbaf3-ad3a-45d8-b216-347cafe54670-bernardo-secades-618d039d-d3b0-44cc-8229-74b138695a3a-certificate.pdf" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
      <li class="cert-item">
        <span class="cert-badge" aria-hidden="true">L</span>
        <div class="cert-meta">
          <a class="cert-title" href="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/780bbaf3-ad3a-45d8-b216-347cafe54670-bernardo-secades-94c3e3f1-8392-4f3d-a53a-3b01e09f2ad5-certificate.pdf" target="_blank" rel="noopener">Secure AI/ML-Driven Software Development (LFEL1012)</a>
          <span class="cert-sub">The Linux Foundation · Jul 2026 · <a href="https://ti-user-certificates.s3.amazonaws.com/e0df7fbf-a057-42af-8a1f-590912be5460/780bbaf3-ad3a-45d8-b216-347cafe54670-bernardo-secades-94c3e3f1-8392-4f3d-a53a-3b01e09f2ad5-certificate.pdf" target="_blank" rel="noopener">Verify ↗</a></span>
        </div>
      </li>
    </ul>
  </section>

  <div class="about-socials">
    <a href="https://linkedin.com/in/bernardosecades" target="_blank" rel="noopener">LinkedIn</a>
    <a href="https://github.com/bernardosecades" target="_blank" rel="noopener">GitHub</a>
    <a href="mailto:bernardosecades@gmail.com">Email</a>
    <a href="/en/#newsletter">Newsletter</a>
    <a href="{{ '/feed.xml' | relative_url }}">RSS</a>
  </div>
</div>
