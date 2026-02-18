# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based cybersecurity blog (0xshlomil.github.io) using the Chirpy theme (`jekyll-theme-chirpy` gem), deployed via GitHub Actions to GitHub Pages.

## Build Commands

```bash
# Install dependencies
bundle install

# Local development server (http://localhost:4000)
bundle exec jekyll serve

# Build static site (output to _site/)
bundle exec jekyll build
```

Ruby version: 3.3.0 (see `.ruby-version`). Deployment is handled by GitHub Actions (`.github/workflows/pages-deploy.yml`) on push to `master`.

## Architecture

- **`_posts/`** — Blog posts in Markdown. Filename format: `YYYY-MM-DD-kebab-case-title.md`
- **`_tabs/`** — Navigation pages (about, archives, categories, tags) using Chirpy's tab layout
- **`_data/contact.yml`** — Social/contact links for the sidebar
- **`images/`** — Blog post images, organized by year subfolder
- **`index.html`** — Homepage using Chirpy's `home` layout

Layouts, includes, sass, and assets are provided by the `jekyll-theme-chirpy` gem.

## Blog Post Conventions

Front matter:
```yaml
---
title: 'Post Title'
categories: [Category]
tags: [tag1, tag2]
---
```

- No `layout:` needed (defaults to `post` via `_config.yml`)
- Categories: max 2, broad groupings (e.g., Software Exploitation, Reverse Engineering, Miscellaneous)
- Tags: specific topics, lowercase with hyphens (e.g., cve-2019-0539, linux-kernel)
- Images referenced as `/images/path/to/image.png`
- Permalinks follow `/:title/` format (set in `_config.yml`)

## Key Configuration

- **Theme:** jekyll-theme-chirpy v7.4
- **Markdown engine:** Kramdown with GFM input, Rouge syntax highlighter
- **Plugins:** jekyll-seo-tag, jekyll-archives (categories + tags)
- **Features:** Auto-generated ToC, dark/light mode toggle, search, code copy buttons
- **GA4:** G-3L25EEXQ5F
