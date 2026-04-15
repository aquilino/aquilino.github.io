# AGENTS.md - aquilino.github.io

## Project Type
Static GitHub Pages site with Markdown rendering via Marked.js (client-side). No build step required.

## Architecture

### Single-Page Application (SPA)
- `index.html` is the only HTML entry point
- Content loads dynamically via URL parameter: `?md=path/to/file.md`
- Uses Marked.js to render Markdown files client-side
- Uses js-yaml to parse frontmatter metadata
- Navigation menu is centralized in `index.html` (no duplication)
- Search system uses `index.json` for metadata lookup
- Theme toggle (dark/light) persisted in localStorage

### Content Structure
- **About**: `About.md` (Whoami page)
- **CTF Writeups**: `Facil/*.md` (5 files)
- **Articles**: `Articulos/*.md` (2 files)
- **Images**: `images/` and `images/retoN/` subdirectories
- **Index**: `index.json` contains metadata for all articles

### Frontmatter Format
Every `.md` file must include YAML frontmatter:
```yaml
---
title: Article Title
date: 2024-01-15
author: h1dr0
tags: [tag1, tag2, tag3]
category: CTF|Articulo|About
difficulty: Facil|Medio|Dificil  # only for CTF
---
```

## Adding New Content

### ⚠️ IMPORTANT: 3 Steps Required
When adding a new article, you MUST update 3 files:

1. **Create `.md` file** in appropriate directory with frontmatter
2. **Add entry to `index.json`**:
   ```json
   {
     "file": "Articulos/NewArticle.md",
     "title": "New Article Title",
     "date": "2024-04-15",
     "author": "h1dr0",
     "tags": ["tag1", "tag2"],
     "category": "Articulo",
     "excerpt": "Short description"
   }
   ```
3. **Add entry to menu in `index.html`**:
   ```html
   <li><a href="#" data-md="Articulos/NewArticle.md">New Article Title</a></li>
   ```

### New CTF Writeup
Same 3 steps, but use `Facil/` (or `Medio/`, `Dificil/`) directory and include `difficulty` in frontmatter.

### New Category
1. Create directory
2. Add menu section in `index.html`

## Key Technical Details

### Search System
- Search input filters by title, tags, author, excerpt
- Filter buttons: Todos, CTF, Artículos
- Results sorted by date (newest first)
- Click result loads the markdown file

### Theme System
- Toggle button (🌙/☀️) in top-right corner
- Persists preference in localStorage
- Uses CSS custom properties (--bg-primary, --text-primary, etc.)

### Path Handling
- Root `index.html` uses `assets/...` and `images/...`
- Markdown files in subdirectories use `../images/...`

### Navigation Links
- All menu links use `data-md` attribute with path to `.md` file
- Clicking updates URL: `?md=Facil/Oclat.md`
- Browser history (back/forward) works correctly

### Home Page
- Shows last 5 articles sorted by date
- Displays title, excerpt, and tags
- Click to load article

## Deployment
GitHub Pages auto-deploys on push to `main`. No build or compilation required.
