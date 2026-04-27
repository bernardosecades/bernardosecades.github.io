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
- **Posts** live in `_posts/` as `YYYY-MM-DD-slug-<lang>.md`. Each post must declare:
  - `lang: en` or `lang: es`
  - `ref: <translation-id>` — same value across the EN and ES versions of the same article so the language switcher can link them.
- **Permalinks** for posts default to `/:lang/:slug/` via `_config.yml`.
- **Layouts** (`_layouts/`):
  - `default.html` — base shell, header with language switcher.
  - `home.html` — per-language post listing.
  - `post.html` — single post view.

## Adding a new post

1. Create two files in `_posts/`, one per language, with the same `ref`:
   - `YYYY-MM-DD-my-tip-en.md` — `lang: en`, `ref: my-tip`
   - `YYYY-MM-DD-my-tip-es.md` — `lang: es`, `ref: my-tip`
2. Both share the same `date` and ideally the same slug (translated if you want).
3. The language switcher on the post page will automatically link to the matching translation.

If a post only exists in one language, the switcher falls back to the per-language home page.
