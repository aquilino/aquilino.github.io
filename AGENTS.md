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

### Content Structure
- **CTF Writeups**: `Facil/*.md` (5 files: Oclat, PepetheFrog, FakeNews, Cryptocorp, Deathnote)
- **Articles**: `Articulos/*.md` (2 files: Docker, Podman)
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
category: CTF|Articulo
difficulty: Facil|Medio|Dificil  # only for CTF
---
```

## Adding New Content

### New Writeup/Article
1. Create `.md` file in appropriate directory (`Facil/`, `Articulos/`)
2. Add frontmatter at the top (see format above)
3. Add entry to `index.json` with metadata
4. Add entry to menu in `index.html`:
   ```html
   <li><a href="#" data-md="Facil/NewFile.md">Title</a></li>
   ```
5. Use relative image paths in Markdown: `../images/retoN/image.jpg`

### New Category
1. Create directory
2. Add menu section in `index.html`:
   ```html
   <header class="major"><h2>Category Name</h2></header>
   <li><a href="#" data-md="NewCategory/file.md">Title</a></li>
   ```

## Key Technical Details

### Search System
- Search input filters by title, tags, author, excerpt
- Filter buttons: Todos, CTF, Artículos
- Results sorted by date (newest first)
- Click result loads the markdown file

### Path Handling
- Root `index.html` uses `assets/...` and `images/...`
- Markdown files in subdirectories use `../images/...`

### Navigation Links
- All menu links use `data-md` attribute with path to `.md` file
- Clicking updates URL: `?md=Facil/Oclat.md`
- Browser history (back/forward) works correctly

### Frontmatter Rendering
- When loading markdown, frontmatter is parsed and displayed
- Shows: title, date, author, category, difficulty (CTF), tags
- Content starts after frontmatter separator

## Deployment
GitHub Pages auto-deploys on push to `main`. No build or compilation required.
