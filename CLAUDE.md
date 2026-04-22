# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Jekyll site deployed to GitHub Pages that hosts Terms & Conditions and Privacy Policies for paid indie mobile apps, in English, Spanish, and Portuguese.

## Local development

```bash
bundle install
bundle exec jekyll serve
# open http://localhost:4000
```

## Deployment

Push to `main`. In GitHub repo Settings → Pages → Source: `main` / `/` (root). Update `_config.yml` with the correct GitHub username and, for project sites, set `baseurl: "/repo-name"`.

## Site architecture

**Language switcher** is driven by three front matter fields in every document page (`en.md`, `es.md`, `pt.md`):
- `lang: en` / `es` / `pt` — marks the active tab in `_layouts/default.html`
- `app: myapp` — used to build the switcher URLs
- `doc: terms` / `privacy` — used to build the switcher URLs

The layout only renders the language nav when both `page.app` and `page.doc` are set.

**Redirect files** (`apps/myapp/terms/index.md`, `apps/myapp/privacy/index.md`) use `<meta http-equiv="refresh">` + JS to forward bare URLs to the English version. Required for App Store review links.

## Adding a new app

1. Copy `apps/myapp/` → `apps/<new-app>/`
2. Update the `permalink`, `app`, and `title` front matter in every file inside the new folder
3. Replace all `myapp` references within the new folder's content
4. Add an entry to `index.md` pointing to the new app's pages

## Placeholders to replace before publishing

Search for and replace all bracketed placeholders in every document:

| Placeholder | Meaning |
|---|---|
| `[APP NAME]` | The app's display name |
| `[DEVELOPER 1]`, `[DEVELOPER 2]` | Developer names |
| `[CONTACT EMAIL]` | Public contact email |
| `[DATE]` | "Last updated" date |
| `[CITY]` | Jurisdiction city (governing law clause) |

Also fill in `_config.yml`: `title`, `description`, `author`, `email`, `url`, and `baseurl`.

## Legal notes

- Liability is capped at the purchase price of the app — this is the standard for paid indie apps and passes App Store review.
- Pages are served with `noindex` to keep them out of search results while still being publicly accessible for store review.
