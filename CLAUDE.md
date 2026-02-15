# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based cybersecurity blog (0xshlomil.github.io) using the Reverie theme, hosted on GitHub Pages.

## Build Commands

```bash
# Install dependencies
bundle install

# Local development server (http://localhost:4000)
bundle exec jekyll serve

# Build static site (output to _site/)
bundle exec jekyll build
```

Ruby version: 2.4.3 (see `.ruby-version`). Production builds happen automatically via GitHub Pages on push to `master`.

## Architecture

- **`_posts/`** — Blog posts in Markdown. Filename format: `YYYY-MM-DD-kebab-case-title.md`
- **`_pages/`** — Static pages (about, archive, categories, search)
- **`_layouts/`** — Liquid templates: `default.html` (base) → `post.html` / `page.html`
- **`_includes/`** — Reusable partials (meta tags, analytics, social icons, comments)
- **`_sass/`** — SCSS partials; `_darcula.scss` for code syntax highlighting
- **`assets/`** — Compiled stylesheet entry point (`style.scss`) and search JS
- **`images/`** — Blog post images, organized by year subfolder

## Blog Post Conventions

Front matter:
```yaml
---
layout: post
title: 'Post Title'
categories: [Category1, Category2]
---
```

Images are stored in `images/` (often in year-based subdirectories) and referenced as `/images/path/to/image.png`.

Permalinks follow `/:title/` format (set in `_config.yml`).

## Key Configuration

- **Markdown engine:** Kramdown with GFM input, Rouge syntax highlighter
- **Plugins:** jekyll-sitemap, jekyll-feed, jekyll-seo-tag, jekyll-paginate
- **Pagination:** 6 posts per page on the homepage
- **Search:** Client-side via SimpleJekyllSearch (`search.json` generates the index)
