# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Jekyll site deployed to GitHub Pages: a small bilingual blog (English / Spanish) for AI tips and tricks.

## Local development

```bash
bundle install
bundle exec jekyll serve
# open http://localhost:4000
```

## Deployment

Push to `main`. GitHub repo Settings → Pages → Source: `main` / `/` (root).

## Site architecture

- **Landing** (`/`, `index.md`) — bilingual entry with two language cards.
- **Per-language home** — `/en/` and `/es/` (`en/index.md`, `es/index.md`) use `_layouts/home.html` to list posts for that language.
- **Posts** live in `_posts/<lang>/YYYY-MM-DD-slug.md`. The folder name (`en` or `es`) becomes the post's category, which Jekyll uses to build the URL. Each post must declare:
  - `lang: en` or `lang: es` (used by layouts and the language switcher)
  - `ref: <translation-id>` — same value across the EN and ES versions of the same article so the language switcher can link them.
- **Permalinks** for posts default to `/:categories/:slug/` via `_config.yml`, e.g. `_posts/en/2026-04-27-welcome.md` → `/en/welcome/`.
- **Layouts** (`_layouts/`):
  - `default.html` — base shell, header with language switcher.
  - `home.html` — per-language post listing.
  - `post.html` — single post view.

## Adding a new post

1. Create two files, one per language, sharing the same `ref`:
   - `_posts/en/YYYY-MM-DD-my-tip.md` — `lang: en`, `ref: my-tip`
   - `_posts/es/YYYY-MM-DD-mi-tip.md` — `lang: es`, `ref: my-tip`
2. Both should share the same `date`. Slugs can be translated independently.
3. The language switcher on the post page will automatically link to the matching translation via the shared `ref`.

If a post only exists in one language, the switcher falls back to the per-language home page.
