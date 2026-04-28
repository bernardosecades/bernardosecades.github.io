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
    <div class="avatar-lg">BS</div>
    <h1>About</h1>
  </section>
  <div class="about-body">
    <p>Hi, I'm Bernardo. I've been writing software for two decades and using LLMs in production since the GPT-3 days. This blog is where I keep the small, useful things I learn — short notes, code snippets, and the occasional rant.</p>
    <p>I currently work on platform tooling and developer experience. Before that, backend infrastructure at a few companies you've heard of.</p>

    <div class="about-grid">
      <div class="about-card">
        <div class="label">Currently</div>
        <div class="value">Platform &amp; DX tooling</div>
      </div>
      <div class="about-card">
        <div class="label">Based in</div>
        <div class="value">Madrid, ES</div>
      </div>
      <div class="about-card">
        <div class="label">Writing since</div>
        <div class="value">2026</div>
      </div>
      <div class="about-card">
        <div class="label">Posts published</div>
        <div class="value">{{ site.posts | where: "lang", "en" | size }}</div>
      </div>
    </div>

    <p>The fastest way to reach me is GitHub or Twitter.</p>
    <div class="about-socials">
      <a href="https://github.com/bernardosecades">
        <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 .5C5.65.5.5 5.65.5 12c0 5.08 3.29 9.39 7.86 10.91.58.11.79-.25.79-.56v-2c-3.2.7-3.87-1.36-3.87-1.36-.52-1.32-1.27-1.67-1.27-1.67-1.04-.71.08-.7.08-.7 1.15.08 1.76 1.18 1.76 1.18 1.02 1.75 2.69 1.24 3.34.95.1-.74.4-1.24.72-1.53-2.55-.29-5.24-1.28-5.24-5.69 0-1.26.45-2.29 1.18-3.1-.12-.29-.51-1.46.11-3.05 0 0 .96-.31 3.15 1.18a10.9 10.9 0 0 1 5.74 0c2.19-1.49 3.15-1.18 3.15-1.18.62 1.59.23 2.76.11 3.05.74.81 1.18 1.84 1.18 3.1 0 4.42-2.69 5.39-5.25 5.68.41.36.78 1.06.78 2.14v3.17c0 .31.21.68.8.56C20.21 21.39 23.5 17.07 23.5 12 23.5 5.65 18.35.5 12 .5z"/></svg>
        GitHub
      </a>
      <a href="https://twitter.com/">
        <svg viewBox="0 0 24 24" fill="currentColor"><path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/></svg>
        Twitter / X
      </a>
      <a href="https://linkedin.com/">
        <svg viewBox="0 0 24 24" fill="currentColor"><path d="M20.45 20.45h-3.55v-5.57c0-1.33-.03-3.04-1.85-3.04-1.85 0-2.13 1.45-2.13 2.95v5.66H9.36V9h3.41v1.56h.05c.48-.9 1.64-1.85 3.37-1.85 3.6 0 4.27 2.37 4.27 5.46v6.28zM5.34 7.43a2.06 2.06 0 1 1 0-4.12 2.06 2.06 0 0 1 0 4.12zM7.12 20.45H3.56V9h3.56v11.45zM22.23 0H1.77C.79 0 0 .77 0 1.72v20.56C0 23.23.79 24 1.77 24h20.45C23.2 24 24 23.23 24 22.28V1.72C24 .77 23.2 0 22.23 0z"/></svg>
        LinkedIn
      </a>
      <a href="{{ '/feed.xml' | relative_url }}">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round"><path d="M4 11a9 9 0 0 1 9 9"/><path d="M4 4a16 16 0 0 1 16 16"/><circle cx="5" cy="19" r="1" fill="currentColor"/></svg>
        RSS
      </a>
    </div>
  </div>
</div>
