## AI Tips & Tricks

Small bilingual (EN / ES) blog hosted on GitHub Pages with Jekyll.

```
_config.yml             ← site config + per-language post permalinks
Gemfile                 ← for local development
index.md                ← bilingual landing
en/index.md             ← English post list (/en/)
es/index.md             ← Spanish post list (/es/)
_layouts/
  default.html          ← header + language switcher
  home.html             ← per-language post listing
  post.html             ← single post view
_posts/<lang>/          ← posts: _posts/en/YYYY-MM-DD-slug.md (folder = category)
assets/css/style.css
```

### Local development
```
bundle install
bundle exec jekyll serve
# open http://localhost:4000
```

### Deploy
Push to `main`. GitHub repo → Settings → Pages → Source: `main` / `/` (root).

### Adding a post
Create two files (one per language) sharing the same `ref`:

- `_posts/en/YYYY-MM-DD-slug.md`
- `_posts/es/YYYY-MM-DD-slug.md`

Front matter:

```yaml
---
title: "..."
date: 2026-04-27
lang: en          # or: es
ref: my-tip       # same value in both translations
---
```

The language switcher uses `ref` to link a post to its translation. If a translation doesn't exist yet, it falls back to the language home page.
