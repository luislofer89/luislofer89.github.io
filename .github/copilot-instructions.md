# al-folio Academic Website - AI Agent Instructions

## Project Overview

This is an **al-folio** Jekyll-based academic portfolio website. It's a static site generator focused on academic content with publications, projects, blog posts, CV, and research showcase features. The site is deployed to GitHub Pages via automated workflows.

## Architecture & Key Components

### Jekyll Collections System

- **Collections**: `news`, `projects`, `books`, `posts` (built-in)
- Collections defined in `_config.yml` and rendered via specific layouts in `_layouts/`
- Each collection has a landing page in `_pages/` (e.g., `_pages/projects.md`)
- Posts/items use frontmatter with `layout`, `title`, `date`, etc.

### Content Flow

1. **Markdown/Liquid** → Jekyll build → **HTML** in `_site/`
2. Pages in `_pages/` use `permalink` in frontmatter to define URLs
3. Layouts in `_layouts/` control page structure (e.g., `about.liquid`, `bib.liquid`, `post.liquid`)
4. Includes in `_includes/` are reusable components (e.g., `news.liquid`, `projects.liquid`)

### Publications System (Jekyll Scholar)

- **BibTeX source**: `_bibliography/papers.bib` - the single source of truth for publications
- **Rendering**: `_layouts/bib.liquid` renders each publication with custom fields
- **Custom BibTeX fields** supported (add to `.bib` entries):
  - `abbr`: venue abbreviation (styled via `_data/venues.yml`)
  - `pdf`, `code`, `slides`, `poster`, `website`: auto-generate buttons
  - `abstract`, `bibtex_show`: expandable sections
  - `selected={true}`: featured on homepage
  - `preview`: thumbnail image path
- **Co-authors**: Auto-linked via `_data/coauthors.yml` (keys MUST be lowercase, accent-free)
- **Filtering**: Custom plugins strip internal keywords (see `filtered_bibtex_keywords` in `_config.yml`)

### Custom Ruby Plugins (`_plugins/`)

- `hide-custom-bibtex.rb`: Removes filtered keywords and superscripts from BibTeX exports
- `google-scholar-citations.rb`, `inspirehep-citations.rb`: Fetch citation metrics
- `external-posts.rb`: Pull posts from external RSS feeds
- `file-exists.rb`, `download-3rd-party.rb`: Asset management helpers
- **Important**: Plugins run at build time, not runtime

## Development Workflow

### Local Development (Docker - REQUIRED)

```bash
# Pull latest image and start server
docker compose pull
docker compose up

# Access at http://localhost:8080 with live reload
# Changes auto-rebuild except _config.yml (requires restart)
```

**Do NOT use native Jekyll setup** - Docker is the standard approach. Alternative slim image: `docker compose -f docker-compose-slim.yml up`

### Build Process

1. **Jekyll build**: Processes Markdown, Liquid, SASS → `_site/`
2. **ImageMagick**: Generates responsive WebP images (widths: 480, 800, 1400px)
3. **PurgeCSS**: Removes unused CSS (configured in `purgecss.config.js`)
4. **Terser**: Minifies JavaScript with `drop_console: true`

### Deployment (Automatic)

- **Trigger**: Push to `main` branch (excluding docs/config-only changes)
- **Workflow**: `.github/workflows/deploy.yml`
  1. Installs Ruby 3.3.5, Python 3.13, ImageMagick
  2. Runs `JEKYLL_ENV=production bundle exec jekyll build`
  3. Purges CSS with PurgeCSS
  4. Deploys `_site/` to `gh-pages` branch
- **Result**: GitHub Pages serves from `gh-pages` branch (~45s to live)

## Critical Configuration Patterns

### \_config.yml Structure

```yaml
# Site identity (MUST match for GitHub Pages)
url: https://<username>.github.io
baseurl: # LEAVE EMPTY for user/org sites

# Collections (add new ones here + create folder + landing page)
collections:
  news:
    output: true
  projects:
    output: true

# Jekyll Scholar (publications)
scholar:
  last_name: [YourLastName] # For author highlighting
  source: /_bibliography/
  style: apa # Citation style

# Optional features (toggle on/off)
enable_darkmode: true
enable_google_analytics: false
```

### Frontmatter Conventions

**Pages** (`_pages/*.md`):

```yaml
layout: page # or: about, cv, distill, post
title: Projects
permalink: /projects/
nav: true
nav_order: 2
```

**Blog Posts** (`_posts/YYYY-MM-DD-title.md`):

```yaml
layout: post
title: My Post
date: 2024-01-15 10:00:00
description: Brief summary
tags: formatting links
categories: sample-posts
```

**Projects** (`_projects/*.md`):

```yaml
layout: page
title: Project Name
img: assets/img/project.jpg # Thumbnail
importance: 1 # Ordering
category: work # or: fun
```

## Common Customization Patterns

### Adding New Content Types

1. Add collection to `_config.yml` collections section
2. Create `_<collection_name>/` directory
3. Create landing page in `_pages/` with appropriate layout
4. (Optional) Add layout in `_layouts/` if custom rendering needed

### Styling Changes

- **Colors**: Edit `_sass/_themes.scss` → `--global-theme-color`
- **Layout**: Edit `_sass/_layout.scss`
- **Typography**: Edit `_sass/_base.scss`
- Use browser DevTools to inspect element sources

### Adding BibTeX Buttons

Edit `_layouts/bib.liquid` around line 200-300 where buttons are conditionally rendered:

```liquid
{% if entry.yourcustomfield %}
  <a href="{{ entry.yourcustomfield }}" class="btn btn-sm z-depth-0" role="button">Custom</a>
{% endif %}
```

Add field name to `filtered_bibtex_keywords` in `_config.yml` to hide from exported BibTeX.

## Known Issues & Gotchas

1. **\_config.yml changes require restart** - NOT hot-reloaded
2. **BibTeX co-author keys** - MUST be lowercase without accents (`"bach":` not `"Bach:"`)
3. **GitHub Pages repo name** - MUST be `<username>.github.io` for baseurl to be empty
4. **ImageMagick required** - Must be installed for responsive images (`imagemagick.enabled: true`)
5. **Collection output** - Set `output: true` in `_config.yml` for individual pages
6. **Liquid vs Ruby plugins** - Liquid is template language; Ruby plugins extend Jekyll at build time

## File Naming Conventions

- Blog posts: `_posts/YYYY-MM-DD-title.md` (date in filename is REQUIRED)
- Projects: `_projects/1_project.md`, `2_project.md` (prefix for ordering)
- News: `_news/announcement_1.md` (no date requirement)
- Pages: `_pages/descriptive-name.md` (permalink in frontmatter defines URL)

## Testing & Quality

- **Prettier**: Code formatting (`npm` scripts in `package.json`)
- **Broken links**: `lychee` checker in `.github/workflows/broken-links.yml`
- **Accessibility**: Manual Axe testing (see README FAQ)
- **Lighthouse**: PageSpeed insights in `lighthouse_results/`

## Key Files to Check First

When investigating issues or making changes:

1. `_config.yml` - All feature flags and site configuration
2. `Gemfile` - Ruby dependencies and Jekyll plugins
3. `_pages/about.md` - Homepage structure and what's displayed
4. `_layouts/` - How different content types are rendered
5. `.github/workflows/deploy.yml` - Build and deployment process
