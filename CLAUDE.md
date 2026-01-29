# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog built with Hexo (v8.1.1) using the Butterfly theme. It's a static site generator project that deploys to GitHub Pages.

- **Live URL**: https://wufulin.github.io
- **Language**: Chinese (zh-CN)
- **Timezone**: Asia/Shanghai

## Common Commands

```bash
# Install dependencies
npm install

# Start local development server at http://localhost:4000
npm run server

# Build static site (outputs to `public/`)
npm run build

# Build and deploy to GitHub Pages (pushes to gh-pages branch)
npm run deploy
```

## Architecture

### Content Structure

- `source/_posts/` - Markdown blog posts with YAML frontmatter
- `source/about/` - About page
- `source/categories/` - Categories page
- `source/tags/` - Tags page
- `source/links/` - Friend links page
- `source/timeline/` - Timeline page
- `source/img/` - Static image assets

### Configuration Files

- `_config.yml` - Hexo main configuration (site metadata, URL, deployment)
- `_config.butterfly.yml` - Butterfly theme configuration (navigation, colors, features)
- `package.json` - Node.js dependencies and npm scripts

### Theme

The Butterfly theme is included as a git submodule at `themes/butterfly/` from https://github.com/jerryc127/hexo-theme-butterfly.git. When cloning this repository, use:

```bash
git clone --recursive <repo-url>
# or if already cloned:
git submodule update --init --recursive
```

### Deployment

Two deployment methods are configured:

1. **GitHub Actions Auto-Deployment** (primary): Pushes to `main` branch trigger automatic build and deployment to `gh-pages` branch via `.github/workflows/deploy.yml`

2. **Local Deployment**: `npm run deploy` builds and pushes to the gh-pages branch manually

## Key Features

- MacOS-style code blocks with copy functionality
- Mermaid diagram support (enabled in `_config.butterfly.yml`)
- Local search functionality
- RSS/Atom feed generation
- XML sitemap generation
- Word count for posts

## Content Guidelines

When creating new posts:

- Posts are Markdown files with YAML frontmatter
- Place new posts in `source/_posts/`
- Use categories and tags for organization
- Images should be placed in `source/img/` and referenced with `/img/<filename>`
- Mermaid diagrams are supported via code blocks with `mermaid` language tag
