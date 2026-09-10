# CLAUDE.md

Jekyll blog on the **Chirpy** theme (v5.2.1), GitHub Pages. Korean-language posts.

## Posts

- Path: `_posts/<topic>/YYYY-MM-DD-<title>.md` (topic dirs: `think`, `devOps`, `spring`, `database`, …).
- Front matter: `title`, `author: OsoriAndOmori`, `date: YYYY-MM-DD HH:MM:SS +0900`, `categories: [A, B]`, `tags: [...]` (lowercase).
- Future-dated posts don't build. Use a time already past in KST (e.g. `09:00:00 +0900`).
- Diagrams: add `mermaid: true` to front matter, then use ```` ```mermaid ```` blocks. Without the flag they won't render.
- Post images: `assets/img/posts/<slug>/…`, referenced as `/assets/img/posts/<slug>/file.svg`. Authored SVG figures are fine and render crisp in both themes (give them their own light panel bg + dark strokes).

## Presentation mode (per-post slideshow)

A post gets a **"▶ 발표 모드"** button when its front matter has `presentation: true`.
Mechanism: `_layouts/post.html` (local override) injects the button + `_includes/presentation.html`
(self-contained overlay: h2/h4 split, keyboard + click nav, theme-aware, no external libs).

**Author a condensed deck** — do NOT let it slideshow the full prose. Put a hidden block at the
**end of the post body**:

```markdown
<div class="slides-src" hidden markdown="1">

#### Slide title
- keyword bullet
- keyword bullet

![alt](/assets/img/posts/<slug>/figure.svg)

---

#### Next slide
...

</div>
```

- `---` (hr) separates slides; `####` (h4) is the slide title.
- Use `####`, never `##`/`###`, inside the block — h4 keeps it out of the post TOC.
- `hidden` keeps it invisible in normal reading view; the script clones it into the overlay.
- Images and ```` ```mermaid ```` blocks work inside the deck.
- No `.slides-src` block → the script falls back to splitting the full article at each `##`.
- Editing the talk = edit the `.slides-src` block only; the article prose stays untouched.

## Local build / preview

Ruby 2.6 mis-detects encoding on Korean filenames, so **always** export UTF-8:

```bash
export LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 RUBYOPT="-E utf-8"
bundle exec jekyll build          # or: bundle exec jekyll serve --watch --drafts
```

The build touches `Gemfile.lock` (adds a platform line) — `git checkout Gemfile.lock` before committing.
`_site/` and `output/` are build artifacts; don't commit them.
