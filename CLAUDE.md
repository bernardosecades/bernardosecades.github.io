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
  - `default.html` — base shell, header with language switcher. Emits all JSON-LD via `{% include seo/... %}`.
  - `home.html` — per-language post listing.
  - `post.html` — single post view.
- **SEO / JSON-LD** (`_includes/seo/`): reusable schema partials emitted from `default.html`.
  - `organization.html`, `website.html` — global, on every page.
  - `article.html`, `breadcrumb.html` — emitted when `page.layout == "post"`.
  - `video.html` — emitted only when a post declares `video:` frontmatter (see below).
  - All strings go through `jsonify` (never `escape`) to keep JSON valid. One `<script>` per schema.

## Adding a new post

1. Create two files, one per language, sharing the same `ref`:
   - `_posts/en/YYYY-MM-DD-my-tip.md` — `lang: en`, `ref: my-tip`
   - `_posts/es/YYYY-MM-DD-mi-tip.md` — `lang: es`, `ref: my-tip`
2. Both should share the same `date`. Slugs can be translated independently.
3. The language switcher on the post page will automatically link to the matching translation via the shared `ref`.

If a post only exists in one language, the switcher falls back to the per-language home page.

### Posts with embedded video

If the post embeds a `<video>` from `/assets/videos/`, declare a `video:` block in the frontmatter and add a JPG poster next to the `.webm`:

```yaml
video:
  src: /assets/videos/demo-foo.webm
  thumbnail: /assets/videos/demo-foo.jpg   # required for VideoObject schema
  duration: PT45S                          # ISO 8601: PT<min>M<sec>S
  width: 1920                              # optional
  height: 1080                             # optional
```

Also add the same path as `poster=` on the `<video>` tag so the still frame is consistent visually and in schema.

Without `video:` frontmatter the VideoObject schema is silently skipped — useful when a `<video>` is embedded but the asset isn't ready yet.

To extract a thumbnail and read duration from a `.webm` (system has GStreamer, no ffmpeg):

```bash
gst-launch-1.0 -q filesrc location=assets/videos/demo-foo.webm ! decodebin ! videoconvert ! jpegenc \
  ! multifilesink location=assets/videos/demo-foo.jpg max-files=1 next-file=buffer
gst-discoverer-1.0 assets/videos/demo-foo.webm | grep -E 'Duration|Width|Height'
```

### Previewing a post in production (unlisted)

GitHub Pages doesn't accept build flags or env vars, so there's no `?preview=1`. The workaround is an `unlisted: true` frontmatter flag: the post is built and reachable at its real URL (shareable), but hidden from listings, sitemap, RSS, the EN/ES switcher on its sibling, and search engines.

1. The post `date:` can be future-dated: `future: true` is enabled in `_config.yml`, so future-dated posts ARE built. ⚠️ Side effect: any future-dated post **without** `unlisted: true` will be visible in production. Always pair a future date with `unlisted: true` until you mean to launch.
2. `git push origin main`. The URL `/en/<slug>/` (or `/es/<slug>/`) serves the post; nothing public references it.
3. To officially launch: remove both lines (and adjust the date if needed) and push again.

Where the `unlisted` flag is honored:
- `_layouts/home.html` — filters the per-language listing and "featured" lookup.
- `_layouts/post.html` — filters "Suggested next" / "Sigue leyendo".
- `_layouts/default.html` — switcher EN↔ES skips unlisted matches, emits `<meta name="robots" content="noindex,nofollow">`, and suppresses Article/Breadcrumb/Video JSON-LD.
- `feed.xml` (root) — custom Atom feed overriding `jekyll-feed`, filters unlisted out.
- `sitemap: false` — handled natively by `jekyll-sitemap`.
