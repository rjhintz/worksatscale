# worksatscale.com

Source for the Works@Scale consulting site, built with [Hugo](https://gohugo.io)
using the [Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke) theme.
Deployed automatically to GitHub Pages at https://worksatscale.com.

## How it works

- Push to `main` → GitHub Actions (`.github/workflows/hugo.yml`) builds the
  site with Hugo and deploys it to GitHub Pages. No manual build or upload
  step — this replaced the old zip-to-cPanel workflow.
- Hugo version is pinned in the workflow (currently 0.165.0) because the
  Ananke theme requires Hugo 0.146.0+. If a future theme update bumps that
  requirement further, the build will fail with a clear version error in the
  Actions log — bump `HUGO_VERSION` in the workflow file to fix.
- Custom domain and DNS (A records + www CNAME) are configured at Namecheap,
  pointing to GitHub Pages. Email for the domain is separate from this repo
  entirely — it's handled via Namecheap DNS Mail Settings, not anything here.

## Editing content

Content lives in `content/*.md` as TOML front matter + Markdown:

- `content/_index.md` — homepage
- `content/about.md`, `content/solutions.md`, `content/contact.md`,
  `content/privacy.md` — the other top-level pages
- `content/notes/` — the Notes section (blog-style posts)

Edit the Markdown, commit, push to `main`. The site rebuilds automatically
within a minute or two.

## Editing styles

Custom CSS lives at `static/css/custom.css` — **not** `assets/css/`. This
matters: the Ananke theme's asset bundler only looks for custom CSS in a
specific internal path and silently fails to load anything placed under
`assets/`, with no error. Keeping it in `static/` sidesteps that entirely —
Hugo just copies it straight through, no processing required.

Two theme quirks worth knowing if page styling looks inconsistent:

- The theme applies a `.serif` class directly to body copy, and an
  `.athelas` class to page titles, on every non-home content page. Both are
  overridden in `custom.css` to keep one consistent typeface. If a new page
  type looks like it's using the wrong font, check whether it needs the same
  override.
- CSS specificity: browsers' own default link-color rule uses a
  `:link` pseudo-class internally, which outranks a plain element selector.
  Any new link-color override needs `:link`/`:visited` included, or it won't
  actually take effect despite loading correctly.

## Images

All current photography is in `static/images/` — crops/framings of the same
source photo (Olafur Eliasson's *Weather Project*, Tate Modern). A few
variants aren't currently assigned to any page; check the directory listing
if you want to swap one out.

## Local preview

```
hugo server -D
```

Requires the extended Hugo binary (for SCSS support) at the version pinned
in the workflow. Site will be at `http://localhost:1313`.

## Hosting & domain

- **Hosting**: GitHub Pages (free)
- **Domain registration + email**: Namecheap
- **DNS**: managed at Namecheap, pointing web traffic to GitHub Pages
- **SSL**: auto-issued and renewed by GitHub Pages, no manual cert management

## Renewal dates to watch

- **Domain registration (Namecheap)**: renews Sep 6, 2027. This is the one
  recurring cost tied to this site — everything else (hosting, SSL) is free.
  If the domain lapses, DNS and email both go down with it, so don't let
  this one slide.
- **Old cPanel hosting plan**: expired/expiring Sep 15, 2026, auto-renew
  off. No action needed — it's already superseded by GitHub Pages and can be
  left to lapse on its own.

## Provenance

Much of the initial setup (git/GitHub migration, GitHub Actions deploy
workflow, CSS fixes, image edits) was implemented by Claude (Anthropic)
working directly in this repo under Richard Hintz's direction, rather than
written by hand. Noted here since multiple AI tools may touch this repo over
time — useful to know which changes came from where.
