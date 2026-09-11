# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A hand-coded, static HTML/CSS/JS portfolio site (no framework, no build step, no dependencies). Deployed via GitHub Pages to `www.adrianaguilar.design` (see `CNAME`). There is no `package.json` — pages are plain `.html` files served as-is.

To preview locally, just serve the directory root, e.g. `python3 -m http.server 3000`, then open `http://localhost:3000/index.html`. There is no lint/build/test command.

## Site structure

- `index.html`, `people.html`, `about.html` — top-level pages.
- `work/*.html` — case study pages linked from the "Work" index (both the "Classic" and "Vibe Coded" tabs on `index.html`).
- `people/*.html` — case study pages linked from `people.html`.
- `password.html` — the site-wide password gate (see Auth below).
- `styles.css` / `tokens.css` — the entire stylesheet, imported by every page via `@import url("tokens.css")` at the top of `styles.css`. `tokens.css` holds all design tokens (OKLCH colors, type scale, spacing, motion) as CSS custom properties; `styles.css` is organized into clearly delimited `/* ---------- Section ---------- */` blocks (Header, Mobile menu, Theme toggle, Hero, Tab switch, Case study template, Case carousel, Work index, Footer, Responsive, Password gate, etc.) — grep for these when editing a specific area.
- `public/` — all images, video, PDFs, and shared assets (logo, favicon, cursors), including a per-project subfolder for each case study's screenshots (e.g. `public/usga/`, `public/tb/`, `public/nyl/`).
- Root-level folders named after each project in shorthand (`DnB/`, `Manscaped/`, `NYL/`, `TB/`, `WT/`, `AIGA/`, `CM/`, `bknets/`, `usga/`, `oh snap/`, `vibey/`) are **raw scraped source material** (original screenshots/copy used as reference when building a case study) and are excluded via `.gitignore` — they are not part of the deployed site. Do not treat them as live content; the processed/optimized versions actually used live under `public/`.

## No shared JS/HTML includes — everything is duplicated per page

There is no templating, no shared header/footer partial, and no external `.js` file. Every page inlines its own copy of:
- The password-gate check script (in `<head>`, before anything renders).
- The header markup (logo, primary nav, mobile-menu hamburger toggle, theme toggle button).
- The mobile-menu overlay markup (`<nav class="mobile-menu" data-mobile-menu>`).
- The footer markup (identity block + dynamic status line).
- Inline `<script>` blocks at the end of `<body>` for: the footer's time-of-day status text, the mobile-menu open/close behavior, the light/dark theme toggle, and (on pages with one) the image carousel.

**Consequence:** any change to shared UI (nav links, footer, theme toggle, gate redirect logic, carousel behavior) must be applied to *every* HTML file that has it — there is no single source of truth to edit once. When making such a change, grep across `*.html work/*.html people/*.html` first to find every occurrence, and prefer a small Python/sed batch script over manual per-file edits to keep them byte-for-byte consistent (watch for indentation differences between root-level pages and files one directory deep — a literal-string replace can silently miss one variant).

Root-level pages (`index.html`, `people.html`, `about.html`, `password.html`) use asset paths relative to the root (e.g. `public/logo.svg`, `styles.css`). Pages under `work/` and `people/` use `../` prefixes (e.g. `../public/logo.svg`, `../styles.css`) and link back with `../index.html` etc.

## Auth gate

The whole site is protected by a single shared password, checked client-side only (`sessionStorage`, not real security — see `password.html`'s inline script for the literal password string). Every protected page has this snippet in `<head>`, before the stylesheet loads, so the redirect happens before any content paints:

```html
<script>
if (sessionStorage.getItem("adrian-site-auth") !== "true") {
  location.replace("password.html?redirect=<page-path>");
}
</script>
```

`password.html` reads the `redirect` query param and sends the user back to the originally-requested page after a correct submit. `favicon.svg`, styles, and other static assets are not gated — only the HTML pages check this.

## Case study page template

Every page under `work/` and `people/` follows the same structure (copy an existing one — e.g. `work/usga.html` — as the starting point for a new case study rather than building from scratch):

1. `<section class="hero case-hero">` — breadcrumb (`<a class="breadcrumb-link">`) + `<h1 class="hero-description">`.
2. `<section class="case-overview">` — two-column: `.case-overview-text` (Goal/Solution blocks, each `.case-block` with a `.case-block-label`) beside `.case-credits` (a `<aside>` of `.case-credit` role/name pairs).
3. One or more `<section class="case-demo-section">` — either a single `.case-demo-frame` (static image/video via `.case-demo-media`) or a `.case-carousel` (see below) for multi-image projects.
4. Optional `<section class="case-impact">` — only include this when there's a real, honest outcome/press mention to report; don't fabricate metrics.
5. `<section class="case-back">` — a `.btn-brutal` link back to `../index.html` (Work) or `../people.html` (People).

**Carousel** (`data-carousel` / `data-carousel-track` / `data-carousel-prev` / `data-carousel-next` / `data-carousel-dots`): the accompanying inline script computes slide width as `100 / slides.length`%, builds nav dots dynamically from the slide count, and translates the track by `index * (100 / slides.length)`%. When adding/removing slides, no JS changes are needed — just add/remove `<li class="case-carousel-slide">` entries; the script re-derives everything from `slides.length`.

## Theming

Light/dark themes are driven entirely by CSS custom properties redefined under `:root[data-theme="dark"]` (explicit toggle) and `@media (prefers-color-scheme: dark)` (system default) in `tokens.css`. The toggle button (`data-theme-toggle`) just flips `document.documentElement.dataset.theme` and persists the choice to `localStorage`. When adding new colored UI, use the existing tokens (`--color-paper`, `--color-ink`, `--color-ink-soft`, `--color-accent`, `--color-accent-text`, `--color-accent-ink`, `--color-line`) rather than hardcoding hex/oklch values, so it stays theme-correct automatically. Note `--color-accent-ink` is intentionally a fixed dark value in both themes — it's the text color used *on top of* the yellow accent fill (buttons, active nav item, hover states), which must stay dark-on-yellow regardless of theme.

The logo is rendered via CSS `mask-image` (a `<span class="site-logo">`/`.gate-logo` with `background-color: var(--color-ink)` masked to `public/logo.svg`) rather than an `<img>`, specifically so it recolors correctly across themes without needing a separate dark-mode asset.
