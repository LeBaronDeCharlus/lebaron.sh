# lebaron.sh — Blog Design Spec

## Purpose

A tiny, simple static blog for a writer known online as "Le Baron de Charlus" (after the Proust character). The author has spent 10+ years in IT (dev, devops, SRE, engineering manager, now director) and writes a mix of tech/engineering essays and literary/cultural musings, in a persona-driven voice. Domain: `lebaron.sh`.

## Non-goals

- No comments system, no dark-mode toggle, no search, no pagination beyond what Hugo gives for free.
- No CMS, no JS framework, no build step beyond Hugo itself.
- Not setting up the actual GitHub repo, DNS, or GitHub Pages hosting as part of this build — the author will do that themselves. This spec only prepares the workflow file so that step is a simple push once ready.

## Tech stack

- **Static site generator:** Hugo.
- **Theme:** custom, hand-built minimal theme (no third-party Hugo theme dependency) — kept intentionally small: one CSS file, no JS framework, no unused features.
- **Version control:** git, initialized locally as part of this build.
- **Hosting (future, not built now):** GitHub Pages. A `.github/workflows/hugo.yml` GitHub Actions workflow is included in the scaffold, ready to activate once the author creates a GitHub repo and pushes this code. It is not wired to any live repo/secrets during this build.

## Site structure

```
content/
  posts/           # blog posts (markdown)
  about.md         # persona/bio page
layouts/
  _default/        # list.html, single.html
  index.html       # homepage template
  partials/        # header, footer
static/
  fonts/           # self-hosted Special Elite font files
  css/             # single stylesheet
hugo.toml          # site config
.github/workflows/hugo.yml   # dormant CI workflow for future Pages deploy
.gitignore         # excludes /public, hugo cache dirs
```

### Content model

- **Posts** (`content/posts/*.md`): front matter includes `title`, `date`, `tags`, `draft`. Rendered in reverse-chronological order on the homepage.
- **About page** (`content/about.md`): introduces the Le Baron de Charlus persona. Seeded with clearly marked placeholder text for the author to replace.
- **Tags**: enabled as a Hugo taxonomy, with a browsable `/tags/` index and per-tag pages.
- **RSS**: Hugo's default `index.xml` feed generation is enabled (no custom work needed beyond default config).
- **Seed content**: two sample posts, clearly marked as placeholders, included so templates have real content to render during development. Not intended as real posts to publish.

### Pages/features at launch

- Homepage: reverse-chronological list of posts (title, date, tags).
- About page.
- Tag index and per-tag pages.
- RSS feed.
- No pagination controls beyond Hugo defaults, no search, no comments, no dark-mode toggle.

## Visual design

- **Font:** Special Elite (typewriter-style), self-hosted as a static font file under `static/fonts/` (not loaded from Google Fonts CDN) — keeps the site fully self-contained with no third-party requests. Applied everywhere: site title, headings, and body text.
- **Palette:** vintage typewriter/parchment. Cream/off-white background (approx. `#f4ecd8`), dark ink-brown text (approx. `#2b1d0e`). Flat color, no paper-texture image — keeps page weight minimal in line with "tiny and simple." A texture can be layered in later without restructuring anything.
- **Layout:** single centered column, generous margins, no sidebar.
- **Header:** site title reads "Le Baron de Charlus".

## Deployment (prepared, not activated)

- `.github/workflows/hugo.yml`: builds the Hugo site and deploys to GitHub Pages on push to the default branch. Included in the repo now but has no effect until the author creates the actual GitHub repository, pushes this code, and enables Pages (with `lebaron.sh` as the custom domain) themselves.

## Testing / verification

- `hugo server -D` runs locally and renders homepage, an individual post, the tags index, and the about page without errors.
- `hugo` (production build) completes without errors or warnings and produces a `public/` directory.
- Visual check in a browser: font renders, parchment colors applied, layout is a single readable column at both desktop and mobile widths.
