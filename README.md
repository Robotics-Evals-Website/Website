# Robotics Evaluation Bounty — website

Single-page static site for the Robotics Evaluation Bounty: an open call funding
reproducible, open-source robotics benchmarks built on Inspect Robots.

- **Live site:** https://robotics-evals-website.github.io/Website/
- **Stack:** one hand-written `index.html` (inline CSS + vanilla JS), no build step,
  no dependencies beyond Google Fonts.
- **Social card:** `og.png` (1200×630), referenced absolutely from the meta tags.

## Local preview

Any static server works:

```bash
python3 -m http.server 8000
```

then open http://localhost:8000.

## Deploying

Pushing to the default branch publishes via GitHub Pages (`.nojekyll` is present so
files are served as-is). The canonical URL and `og:` tags in `index.html` point at
the GitHub Pages URL — update them if the site moves to a custom domain.

## Editing notes

- All styling lives in the `<style>` block at the top of `index.html`; design tokens
  (colors, type scale) are CSS custom properties on `:root`. The site ships light
  only: there is no dark theme and no toggle, and `:root` declares
  `color-scheme:light` so a dark OS preference cannot repaint form controls or
  scrollbars. See `design.md` for the full system.
- Interactive pieces (stage-funnel scroll rail, CableBench sampler, results
  registry, SO-101 arm animation, FAQ accordion, print expansion) are small IIFEs
  in `<script>` blocks at the bottom of the file. Everything degrades gracefully
  without JavaScript.
- Key dates and award figures appear in several places (hero, status bar, stages,
  timeline, FAQ, meta tags, `og.png`) — when one changes, update all of them.
