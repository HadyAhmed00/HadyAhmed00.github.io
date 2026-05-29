# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal portfolio site (`hadyahmed00.github.io`). Static Jekyll site built via `github-pages` gem, deployed by GitHub Pages from `main`.

## Commands

```sh
bundle install              # install Jekyll + github-pages
bundle exec jekyll serve    # local dev server at http://127.0.0.1:4000 with live reload
bundle exec jekyll build    # build to _site/
rake export[N]              # archive snapshot of tag vN into ./vN/ (checks out tag, copies files, regenerates README, returns to master)
```

On Windows the `wdm` gem (auto-included via `Gemfile`) handles file-watching.

## Architecture

Standard Jekyll layout — content is data-driven, not page-driven.

- **`_data/*.yml`** — single source of truth for site content. `projects.yml` (items with `featured: true` render on home; all render on `/archive`), `skills.yml`, `links.yml` (sidebar), `other-projects.yml`. Edit YAML to change site content; do not hand-edit pages.
- **`pages/`** — only two routes. `index.html` (sidebar layout, featured projects + skills + team) and `archive.html` (page layout, full project list). Both pull from `_data`.
- **`_layouts/`** — `default.html` is the HTML shell; `sidebar.html` and `page.html` extend it via `layout: default` front matter.
- **`_includes/`** organized into:
  - `site/` — `head.html`, `scripts.html` (page-level partials)
  - `sections/` — composable page sections (`projects.html`, `skills.html`, `team.html`) parameterized via `{% include ... title=... source=... %}`
  - `components/` — atomic UI (`project.html`, `skill.html`, `sidebar.html`, `icon.html`)
  - `icons/` — inline SVGs. `icon.html` resolves `name` + optional `category` to a path via `slugify` (e.g. `name="LoForm" category="projects"` → `icons/projects/loform.svg`). To add a project, drop a matching slugged `.svg` into `_includes/icons/projects/` alongside the `_data/projects.yml` entry.
- **`_sass/`** — SCSS partitioned into `abstracts/` (vars, mixins), `base/` (reset, typography), `components/`, `layout/`. Entry is `_sass/main.scss`, compiled via `assets/css/style.scss` (front-matter triggers Jekyll's Sass processor).
- **`assets/js/contributions.js`** — runtime fetches the `lauripiispanen/github-top` repo's `egypt.yml`, parses the user's rank, and renders it into `#gh-rank`. Pure browser JS, no build step.

Site identity (`{{ site.github.owner.* }}`) comes from the GitHub Pages `jekyll-github-metadata` plugin — `_config.yml` only sets `repository`. OG/Twitter `title` meta are hardcoded to `HadyAhmed00` in `_includes/site/head.html` (the upstream `portfolYOU` template uses fixed strings here, not metadata).

Project-icon convention: `_includes/icons/projects/<slugified-name>.svg` must exist for every project in `_data/projects.yml`. `_includes/components/icon.html` does `{% include {{ path }} %}` with no fallback — a missing SVG fails the build. There's a `default.svg` placeholder; if you don't want to draw a custom icon, copy `default.svg` to the new slug.
