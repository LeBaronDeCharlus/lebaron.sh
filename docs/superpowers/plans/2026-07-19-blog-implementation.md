# lebaron.sh Blog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a tiny, self-contained Hugo static blog for "Le Baron de Charlus" with a hand-written parchment/typewriter theme, seeded with placeholder content, ready to push to GitHub Pages later.

**Architecture:** A Hugo site using project-root `layouts/` and `static/` (no `themes/` subdirectory — Hugo supports this directly, and it's simpler for a one-off custom theme). No JS, one CSS file, self-hosted font. A dormant GitHub Actions workflow handles future Pages deploys but is not wired to any live repo in this plan.

**Tech Stack:** Hugo v0.164.0 (installed via Homebrew, already present at `/opt/homebrew/bin/hugo` in this environment), plain HTML/CSS, git.

## Global Constraints

- No JS framework, no third-party Hugo theme, no CMS — spec explicitly rules these out.
- Font must be self-hosted under `static/fonts/`, not loaded from a CDN.
- Palette: background `#f4ecd8` (parchment), text `#2b1d0e` (ink) — from spec.
- Special Elite font used for everything (title, headings, body) — from spec.
- No pagination controls, no search, no comments, no dark-mode toggle — spec non-goals.
- `baseURL` in `hugo.toml` is `https://lebaron.sh/` — from spec.
- GitHub repo creation, DNS, and Pages activation are explicitly out of scope for this plan (author does this later) — only the workflow file is prepared.
- Every task's changes are committed to git at the end of the task.

---

## File Structure

```
hugo.toml                          # site config (Task 1)
.gitignore                         # excludes /public, /resources, lock file (Task 1)
content/
  about.md                         # persona bio placeholder (Task 2)
  posts/
    hello-world.md                 # placeholder post (Task 2)
    on-the-directorship.md         # placeholder post (Task 2)
layouts/
  _default/
    baseof.html                    # base HTML skeleton (Task 3)
    list.html                      # generic list template: term pages, /posts/ (Task 3)
    single.html                    # post + about page template (Task 3)
    taxonomy.html                  # /tags/ index (tag names only, no dates) (Task 3)
  index.html                       # homepage: reverse-chron post list (Task 3)
  partials/
    header.html                    # site title + nav (Task 3)
    footer.html                    # copyright + RSS link (Task 3)
static/
  css/
    style.css                      # single stylesheet, parchment theme (Task 4)
  fonts/
    SpecialElite-Regular.ttf       # self-hosted font file (Task 4)
    NOTICE.md                      # font license attribution (Task 4)
.github/
  workflows/
    hugo.yml                       # dormant Hugo + Pages deploy workflow (Task 6)
```

Tags/RSS (Task 5) and the final production build (Task 7) touch no new files — they verify behavior that Tasks 1–4 already produce, and Task 7 also updates `docs/superpowers/specs/2026-07-19-blog-design.md`'s status if needed and does a last full-site check.

---

### Task 1: Hugo project scaffold and config

**Files:**
- Create: `hugo.toml`
- Create: `.gitignore`

**Interfaces:**
- Produces: a `hugo.toml` with `baseURL = "https://lebaron.sh/"`, `title = "Le Baron de Charlus"`, and `[taxonomies] tag = "tags"` — every later task's templates rely on `.Site.Title` and the `tags` taxonomy existing.

- [ ] **Step 1: Confirm Hugo is installed**

Run: `hugo version`
Expected: output containing `hugo v0.164` (already installed in this environment via Homebrew). If it's missing, run `brew install hugo` first.

- [ ] **Step 2: Create `hugo.toml`**

```toml
baseURL = "https://lebaron.sh/"
title = "Le Baron de Charlus"

[taxonomies]
  tag = "tags"
```

- [ ] **Step 3: Create `.gitignore`**

```
/public/
/resources/
.hugo_build.lock
```

- [ ] **Step 4: Verify the config builds cleanly with no content**

Run: `hugo`
Expected: exit code 0, no `WARN` or `ERROR` lines, output table showing `Pages │ 3` (Hugo always creates a home, a 404, and a sitemap-adjacent internal page count even with zero content) — for our purposes, the pass criterion is: no warnings/errors printed, and `public/sitemap.xml` and `public/index.xml` exist.

Check: `test -f public/sitemap.xml && test -f public/index.xml && echo OK`
Expected: `OK`

- [ ] **Step 5: Commit**

```bash
git add hugo.toml .gitignore
git commit -m "Scaffold Hugo site config"
```

---

### Task 2: Seed content (about page + sample posts)

**Files:**
- Create: `content/about.md`
- Create: `content/posts/hello-world.md`
- Create: `content/posts/on-the-directorship.md`

**Interfaces:**
- Consumes: nothing from Task 1 directly (content is independent of config validity).
- Produces: two pages under the `posts` section with `tags` front matter (`meta`; `engineering`, `leadership`) and one `about` page — Task 3's templates and Task 5's taxonomy check depend on these tags and section existing exactly as named.

- [ ] **Step 1: Create the About page placeholder**

`content/about.md`:
```markdown
---
title: "About"
---

*(Placeholder — replace with the real Baron de Charlus bio.)*

Notes on code, command, and culture, from someone who has spent more than a decade moving through the ranks of IT: developer, devops engineer, SRE, engineering manager, and now director. Writing here under the name of Le Baron de Charlus.
```

- [ ] **Step 2: Create the first placeholder post**

`content/posts/hello-world.md`:
```markdown
---
title: "Hello, World"
date: 2026-07-19
tags: ["meta"]
draft: false
---

*(Placeholder post — replace with real content.)*

This is the first entry on this blog. More to come.
```

- [ ] **Step 3: Create the second placeholder post**

`content/posts/on-the-directorship.md`:
```markdown
---
title: "On the Directorship"
date: 2026-07-18
tags: ["engineering", "leadership"]
draft: false
---

*(Placeholder post — replace with real content.)*

A short placeholder musing on moving from engineering into direction, to prove the tag and archive pages work correctly.
```

- [ ] **Step 4: Verify content parses correctly**

Run: `hugo list all`
Expected: CSV output with exactly 3 rows (plus header), containing `content/posts/hello-world.md`, `content/posts/on-the-directorship.md`, and `content/about.md`, each with `draft` = `false` and correct `permalink` values (`https://lebaron.sh/posts/hello-world/`, `https://lebaron.sh/posts/on-the-directorship/`, `https://lebaron.sh/about/`).

Check: `hugo list all | grep -c 'content/posts/hello-world.md\|content/posts/on-the-directorship.md\|content/about.md'`
Expected: `3`

- [ ] **Step 5: Commit**

```bash
git add content/
git commit -m "Add placeholder about page and sample posts"
```

---

### Task 3: Layout templates

**Files:**
- Create: `layouts/partials/header.html`
- Create: `layouts/partials/footer.html`
- Create: `layouts/_default/baseof.html`
- Create: `layouts/index.html`
- Create: `layouts/_default/list.html`
- Create: `layouts/_default/taxonomy.html`
- Create: `layouts/_default/single.html`

**Interfaces:**
- Consumes: `.Site.Title` (from Task 1's `hugo.toml`), the `posts` section and `tags` front matter (from Task 2's content).
- Produces: rendered HTML at `/`, `/about/`, `/posts/<slug>/`, `/tags/`, `/tags/<tag>/` — Task 4's CSS selectors (`.site-header`, `.site-nav`, `.post-list`, `.post-item`, `.post-date`, `.post-tags`, `.post-meta`, `.site-footer`, `.tag-list`) must match the classes used here exactly.

- [ ] **Step 1: Create header and footer partials**

`layouts/partials/header.html`:
```html
<header class="site-header">
  <a class="site-title" href="{{ "/" | relURL }}">{{ .Site.Title }}</a>
  <nav class="site-nav">
    <a href="{{ "/" | relURL }}">Posts</a>
    <a href="{{ "/about/" | relURL }}">About</a>
    <a href="{{ "/tags/" | relURL }}">Tags</a>
  </nav>
</header>
```

`layouts/partials/footer.html`:
```html
<footer class="site-footer">
  <p>&copy; {{ now.Format "2006" }} {{ .Site.Title }} &middot; <a href="{{ "/index.xml" | relURL }}">RSS</a></p>
</footer>
```

- [ ] **Step 2: Create the base template**

`layouts/_default/baseof.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ .Title }} &middot; {{ .Site.Title }}{{ end }}</title>
  <link rel="stylesheet" href="{{ "css/style.css" | relURL }}">
  <link rel="alternate" type="application/rss+xml" href="{{ "index.xml" | relURL }}" title="{{ .Site.Title }}">
</head>
<body>
  {{ partial "header.html" . }}
  <main>
    {{ block "main" . }}{{ end }}
  </main>
  {{ partial "footer.html" . }}
</body>
</html>
```

- [ ] **Step 3: Create the homepage template**

`layouts/index.html`:
```html
{{ define "main" }}
<ul class="post-list">
  {{ range where .Site.RegularPages "Type" "posts" }}
  <li class="post-item">
    <span class="post-date">{{ .Date.Format "2006-01-02" }}</span>
    <a href="{{ .RelPermalink }}">{{ .Title }}</a>
    {{ if .Params.tags }}
    <span class="post-tags">
      {{ range .Params.tags }}<a href="{{ "/tags/" | relURL }}{{ . | urlize }}/">#{{ . }}</a> {{ end }}
    </span>
    {{ end }}
  </li>
  {{ end }}
</ul>
{{ end }}
```

- [ ] **Step 4: Create the generic list template (used for `/posts/` and `/tags/<tag>/`)**

`layouts/_default/list.html`:
```html
{{ define "main" }}
<h1>{{ .Title }}</h1>
{{ if .Content }}<div class="page-content">{{ .Content }}</div>{{ end }}
<ul class="post-list">
  {{ range .Pages }}
  <li class="post-item">
    <span class="post-date">{{ .Date.Format "2006-01-02" }}</span>
    <a href="{{ .RelPermalink }}">{{ .Title }}</a>
  </li>
  {{ end }}
</ul>
{{ end }}
```

- [ ] **Step 5: Create the taxonomy index template (used for `/tags/`)**

`layouts/_default/taxonomy.html`:
```html
{{ define "main" }}
<h1>{{ .Title }}</h1>
<ul class="tag-list">
  {{ range .Pages }}
  <li><a href="{{ .RelPermalink }}">#{{ .Title }}</a> ({{ len .Pages }})</li>
  {{ end }}
</ul>
{{ end }}
```

- [ ] **Step 6: Create the single-page template (used for posts and About)**

`layouts/_default/single.html`:
```html
{{ define "main" }}
<article>
  <h1>{{ .Title }}</h1>
  {{ if eq .Type "posts" }}
  <p class="post-meta">
    <span class="post-date">{{ .Date.Format "January 2, 2006" }}</span>
    {{ if .Params.tags }}
    <span class="post-tags">
      {{ range .Params.tags }}<a href="{{ "/tags/" | relURL }}{{ . | urlize }}/">#{{ . }}</a> {{ end }}
    </span>
    {{ end }}
  </p>
  {{ end }}
  <div class="post-content">
    {{ .Content }}
  </div>
</article>
{{ end }}
```

- [ ] **Step 7: Build and verify rendered output**

Run: `rm -rf public && hugo`
Expected: exit code 0, no `WARN` or `ERROR` lines this time (Task 1's home/taxonomy warnings should be gone now that templates exist).

Check each of the following (all should print a match):
```bash
grep -o 'Hello, World' public/index.html
grep -o 'href="/posts/hello-world/"' public/index.html
grep -o '#meta' public/index.html
grep -o 'Le Baron de Charlus' public/about/index.html
grep -io 'placeholder' public/about/index.html
```
Expected: each command prints at least one matching line; none print empty output.

- [ ] **Step 8: Commit**

```bash
git add layouts/
git commit -m "Add homepage, post, about, and taxonomy templates"
```

---

### Task 4: Font and stylesheet

**Files:**
- Create: `static/fonts/SpecialElite-Regular.ttf`
- Create: `static/fonts/NOTICE.md`
- Create: `static/css/style.css`

**Interfaces:**
- Consumes: the CSS class names produced by Task 3's templates (`.site-header`, `.site-title`, `.site-nav`, `.post-list`, `.post-item`, `.post-date`, `.post-tags`, `.post-meta`, `.tag-list`, `.site-footer`, `.page-content`, `.post-content`).
- Produces: `public/css/style.css` and `public/fonts/SpecialElite-Regular.ttf` after build — nothing downstream depends on this beyond the final visual check in Task 7.

- [ ] **Step 1: Download the Special Elite font file**

Run:
```bash
mkdir -p static/fonts
curl -sL -o static/fonts/SpecialElite-Regular.ttf "https://fonts.gstatic.com/s/specialelite/v20/XLYgIZbkc4JPUL5CVArUVL0nhnc.ttf"
file static/fonts/SpecialElite-Regular.ttf
```
Expected: `static/fonts/SpecialElite-Regular.ttf: TrueType Font data` (approx. 147KB).

- [ ] **Step 2: Add a font license notice**

`static/fonts/NOTICE.md`:
```markdown
# Special Elite

Font: "Special Elite" by Google Fonts.
Source: https://fonts.google.com/specimen/Special+Elite
License: SIL Open Font License 1.1 — https://openfontlicense.org/
```

- [ ] **Step 3: Create the stylesheet**

`static/css/style.css`:
```css
@font-face {
  font-family: "Special Elite";
  src: url("/fonts/SpecialElite-Regular.ttf") format("truetype");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

:root {
  --parchment: #f4ecd8;
  --ink: #2b1d0e;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background-color: var(--parchment);
  color: var(--ink);
  font-family: "Special Elite", monospace;
  line-height: 1.6;
}

main {
  max-width: 640px;
  margin: 0 auto;
  padding: 1rem 1.5rem 3rem;
}

a {
  color: var(--ink);
  text-decoration: underline;
}

.site-header {
  max-width: 640px;
  margin: 0 auto;
  padding: 2rem 1.5rem 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.site-title {
  font-size: 1.75rem;
  text-decoration: none;
}

.site-nav a {
  margin-right: 1rem;
}

.post-list {
  list-style: none;
  padding: 0;
}

.post-item {
  margin-bottom: 1rem;
}

.post-date {
  display: inline-block;
  min-width: 6.5rem;
  opacity: 0.75;
}

.post-tags a {
  margin-right: 0.5rem;
  font-size: 0.9rem;
}

.post-meta {
  opacity: 0.85;
  margin-top: -0.5rem;
}

.site-footer {
  max-width: 640px;
  margin: 0 auto;
  padding: 1rem 1.5rem 2rem;
  opacity: 0.75;
  font-size: 0.9rem;
}
```

- [ ] **Step 4: Build and verify static assets are copied and referenced**

Run: `rm -rf public && hugo --minify`
Expected: exit code 0, no warnings/errors, output table shows `Static files │ 2`.

Check:
```bash
test -f public/css/style.css && echo "css OK"
test -f public/fonts/SpecialElite-Regular.ttf && echo "font OK"
grep -o 'Special Elite' public/css/style.css | head -1
grep -o 'stylesheet' public/index.html
```
Expected: `css OK`, `font OK`, `Special Elite`, and a match for `stylesheet`.

- [ ] **Step 5: Manual visual check (not automatable — do this yourself)**

Run: `hugo server -D`, open `http://localhost:1313/` in a browser. Confirm: cream background, dark ink text, Special Elite font visibly applied to title/headings/body, single centered column, nav links to Posts/About/Tags all work. Stop the server with Ctrl+C when done.

- [ ] **Step 6: Commit**

```bash
git add static/
git commit -m "Add self-hosted Special Elite font and parchment stylesheet"
```

---

### Task 5: Verify tags taxonomy and RSS feed

**Files:** none created or modified — this task only verifies behavior already produced by Tasks 1–4's config, content, and templates.

**Interfaces:**
- Consumes: the `tags` taxonomy from Task 1, the tagged posts from Task 2, `taxonomy.html`/`list.html` from Task 3.
- Produces: nothing new for later tasks; this is a verification checkpoint.

- [ ] **Step 1: Build the site fresh**

Run: `rm -rf public && hugo`
Expected: exit code 0, no warnings or errors.

- [ ] **Step 2: Verify the tag index page lists all three tags**

Run:
```bash
grep -o '#meta' public/tags/index.html
grep -o '#engineering' public/tags/index.html
grep -o '#leadership' public/tags/index.html
```
Expected: each prints a match.

- [ ] **Step 3: Verify each tag's term page lists the correct tagged post**

Run:
```bash
grep -o 'Hello, World' public/tags/meta/index.html
grep -o 'On the Directorship' public/tags/engineering/index.html
grep -o 'On the Directorship' public/tags/leadership/index.html
```
Expected: each prints a match.

- [ ] **Step 4: Verify the RSS feed contains both posts**

Run:
```bash
grep -o '<title>Hello, World</title>' public/index.xml
grep -o '<title>On the Directorship</title>' public/index.xml
```
Expected: each prints a match.

- [ ] **Step 5: Commit**

No files changed in this task — nothing to commit. If any check above fails, fix the relevant file from Task 1–3 and re-run this task's checks before proceeding.

---

### Task 6: GitHub Pages deploy workflow (dormant)

**Files:**
- Create: `.github/workflows/hugo.yml`

**Interfaces:**
- Consumes: nothing from this repo's build — the workflow builds Hugo fresh in CI.
- Produces: nothing consumed by other tasks; this file has no effect until the author pushes to an actual GitHub repository with Pages enabled.

- [ ] **Step 1: Create the workflow file**

`.github/workflows/hugo.yml`:
```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main", "master"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
          extended: true
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build
        run: hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Verify the YAML is well-formed**

Run: `python3 -c "import yaml; d = yaml.safe_load(open('.github/workflows/hugo.yml')); print(sorted(d.keys()))"`
Expected: prints a list containing `'jobs'`, `'name'`, `'permissions'`, `'concurrency'`, and (parsed as boolean `True` under YAML 1.1 rules — this is expected and harmless; GitHub Actions parses `on:` correctly regardless) `True`.

- [ ] **Step 3: Commit**

```bash
git add .github/
git commit -m "Add dormant GitHub Pages deploy workflow"
```

---

### Task 7: Final production build check

**Files:** none created or modified — final integration verification across all prior tasks.

**Interfaces:**
- Consumes: everything from Tasks 1–6.
- Produces: nothing — this is the terminal verification step for this plan.

- [ ] **Step 1: Clean production build**

Run: `rm -rf public resources && hugo --gc --minify`
Expected: exit code 0, zero `WARN`/`ERROR` lines.

- [ ] **Step 2: Verify the full expected file set exists**

Run:
```bash
for f in public/index.html public/about/index.html public/posts/hello-world/index.html public/posts/on-the-directorship/index.html public/tags/index.html public/tags/meta/index.html public/tags/engineering/index.html public/tags/leadership/index.html public/index.xml public/sitemap.xml public/css/style.css public/fonts/SpecialElite-Regular.ttf; do
  test -f "$f" && echo "OK $f" || echo "MISSING $f"
done
```
Expected: every line printed as `OK ...`, none as `MISSING ...`.

- [ ] **Step 3: Spot-check for broken internal links**

Run:
```bash
grep -o 'href="/[^"]*"' public/index.html | sort -u
```
Expected: every path printed corresponds to a real route (`/`, `/about/`, `/tags/`, `/posts/<slug>/`) — visually confirm none are malformed (e.g. no doubled slashes, no empty `href=""`).

- [ ] **Step 4: Confirm git state is clean**

Run: `git status`
Expected: `nothing to commit, working tree clean` (all prior tasks' commits already cover every file; `public/` and `resources/` are gitignored).

- [ ] **Step 5: Final manual walkthrough (not automatable — do this yourself)**

Run: `hugo server -D`, click through Home → a post → Tags → a tag page → About → RSS link, in a browser. Confirm everything reads correctly and the parchment/typewriter look is applied consistently. Stop the server with Ctrl+C.

No commit for this task — it verifies, it doesn't change anything.
