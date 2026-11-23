## Project

Jekyll blog at `https://blog.rubygems.org`. Auto-deploys to GitHub Pages on push to `master` (`.github/workflows/build.yml`). Ruby 3.4.7.

## Commands

```bash
bin/setup                    # bundle install
bundle exec jekyll serve     # Dev server at http://127.0.0.1:4000 (auto-rebuilds)
bundle exec jekyll build     # One-shot build into _site/
```

Tailwind v4 is compiled by the `jekyll-tailwindcss` plugin during the Jekyll build — no Node toolchain. Edit `_tailwind.css`; the plugin emits `_site/assets/css/styles.css`, which the layout links via `/assets/css/styles.css`. The empty `assets/css/styles.tailwindcss` source file is the marker that tells the plugin where to land the output — leave it as is.

## Layout

- `_posts/YYYY-MM-DD-slug.md` — ~470 posts back to 2004
- `_layouts/{default,post}.html` — page shell + article wrapper
- `_includes/{article,archive,nav_links}.html` — post card, year sidebar, nav
- `_tailwind.css` — Tailwind entry; holds `@theme` tokens and `.prose` overrides
- `index.html` — blog index; reuses `_includes/article.html` with `index = false`
- `_config.yml` — site URL is `https://blog.rubygems.org` (see `CNAME`)

## Writing posts

Filename `_posts/YYYY-MM-DD-slug.md`. Required frontmatter:

```yaml
---
title: "Post title"
layout: post
author: Author Name
author_email: author@example.com
---
```

If the post's `author_email` isn't already a key or alias in `_data/authors.yml`, add an entry for the author (`name`, plus optional `url`, `bio`, `aliases`; see the schema comment at the top of the file). Without one, the byline falls back to the plain `author` string with no profile link or bio.

## Gotchas

- **Never edit `_site/assets/css/styles.css`** — that's compiled output, regenerated on every build. Edit `_tailwind.css`.
- **Use Tailwind tokens, not raw hex.** All colors and the font stack are defined as `@theme` tokens in `_tailwind.css` — reference `text-orange-700`, `bg-neutral-050`, etc. The token table is the source of truth; do not re-declare values in CLAUDE.md.
- **Don't override `prose` styles in markup.** Tweak them in the `.prose` layer of `_tailwind.css` so all posts stay consistent.
- **Light mode only** — no dark variants.
- The blog mirrors rubygems.org (the "hammy" template). Header, footer, nav, type, and color should match the main site so the blog reads as an extension of it.

## Design intent

Reliable, open, technical. Content-first: the orange brand color (`orange-500` `#F74C27`) and Titillium Web wordmark are the only signature elements; everything else stays out of the way of the writing. References: GitHub Blog, Rust Blog, Go Blog. Anti-references: marketing-heavy or startup-y aesthetics, animation, dark patterns.

Page shell: `max-w-screen-xl`. Article body: `prose prose-lg` inside `max-w-3xl mx-auto` (centered, ~768px). Display heading: `text-3xl md:text-4xl font-bold text-neutral-800`.
