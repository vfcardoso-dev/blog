# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal blog (pt-br) at https://vfcardoso.dev, built with [Hugo](https://gohugo.io/) and the [`hugo-theme-anubis2`](https://github.com/hugo-theme-anubis2/hugo-theme-anubis2) theme, deployed to Azure Static Web Apps. There is no test or lint tooling.

## Commands

- `hugo server -D` — local dev server with drafts (the `archetypes/default.md` template creates new posts with `draft: true`).
- `hugo new posts/<slug>.md` — create a post from the archetype.
- `hugo --minify` — production build into `public/`.

The theme is pulled as a **Hugo Module** (see `go.mod` and `module.imports` in `config.yaml`), so Hugo (extended, >= 0.167) and Go must both be installed; it is not a git submodule. Update it with `hugo mod get -u`.

## Architecture

- `config.yaml` is the single source of site config: menus (main and footer), social links, and theme params. Menu labels and content are in Portuguese. Theme options live under `params.anubis2` with lowercase keys; the old top-level `params.*` keys of the previous theme raise errors.
- The language key is `pt` (not `pt-br`) so the theme's `i18n/pt.yaml` is picked up; `locale: pt-BR` sets the `<html lang>`.
- The theme provides all layouts and has no head/footer extension hooks; this repo only overrides:
  - `layouts/home.html` renders only `content/_index.md` (the "Sobre mim" page is the home; `/about/` is an alias). The post list lives at `/posts/` (`content/posts/_index.md`, menu label "Blog").
  - `layouts/_partials/head/head.html` is a copy of the theme's head partial plus an Open Graph / Twitter Cards block (marked "LOCAL ADDITION"); re-sync it when updating the theme. The default share image is `static/images/og-default.png` (`params.images`).
  - `static/staticwebapp.config.json` sets security headers (CSP etc.), cache headers and the 404 page for Azure Static Web Apps. The CSP allows only `'self'` scripts plus the sha256 of the theme's inline TOC script; if a theme update changes that script, the hash must be recomputed (the TOC then falls back to the native `<details>` behavior).
  - `assets/css/overrides.css` is loaded via `params.anubis2.customcss` (path relative to `assets/`).
  - Mastodon profile verification (`rel="me"`) is added by the theme to every `socialicons` link. The Fediring webring links are the `footer` menu in `config.yaml`. If the Mastodon instance changes, update the `mastodon` entry in `params.anubis2.socialicons` (it has been migrated twice).
- Content lives in `content/` (`_index.md` = home/about, `contact.md`, `projects.md`, `posts/`). Posts use front matter with `author`, `title`, `date`, `description`, `tags`, `draft`. Raw HTML in Markdown is enabled (`markup.goldmark.renderer.unsafe`).
- `resources/_gen/` (Hugo's generated SCSS cache) and `.hugo_build.lock` are committed; there is no `.gitignore`.

## Deployment

`.github/workflows/azure-static-web-apps-*.yml` builds with Hugo extended (version pinned in `HUGO_VERSION`; the theme needs >= 0.167) and Go, then deploys `public/` on every push to `main` (and previews on PRs) via `Azure/static-web-apps-deploy` with `skip_app_build: true`. Pushing to `main` publishes the site.
