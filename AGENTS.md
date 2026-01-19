# AGENTS.md - AI Coding Guidelines

This document provides guidelines for AI agents working on this Hugo blog project.

## Project Overview

- **Framework:** Hugo static site generator
- **Theme:** PaperMod
- **Deployment:** GitHub Pages via GitHub Actions
- **Author:** Ayush Bihani

## File Structure

```
.
├── archetypes/          # Templates for new content
│   ├── default.md       # Default post template
│   ├── til.md           # TIL (Today I Learned) template
│   └── notes.md         # Notes template
├── assets/              # Site assets (processed by Hugo)
├── content/             # All site content (Markdown)
│   ├── about.md         # About page
│   ├── posts/           # Blog posts
│   ├── til/             # TIL (Today I Learned) entries
│   └── notes/           # Notes (quick thoughts and observations)
├── data/                # Data files
├── hugo.yaml            # Main Hugo configuration
├── i18n/                # Internationalization
├── layouts/             # Custom layout overrides
│   └── partials/        # Custom partials (e.g., extend_head.html)
├── public/              # Generated site output (do not edit)
├── static/              # Static files (copied as-is)
└── themes/PaperMod/     # Theme files (do not edit directly)
```

## Content Creation

### Blog Posts (`content/posts/`)

Create new posts with:
```bash
hugo new posts/your-post-name.md
```

**Required front matter:**
```yaml
---
title: "Your Post Title"
date: 2025-01-19T12:00:00-06:00
draft: false
tags: ["tag1", "tag2"]
categories: ["category"]
description: "Brief description for SEO and previews"
---
```

**Optional front matter:**
- `author: "Name"` - Override default author
- `showToc: true` - Show table of contents
- `math: true` - Enable KaTeX math rendering
- `cover:` - Cover image configuration (PaperMod theme)
  ```yaml
  cover:
    image: "path/to/image.jpg"  # Relative to static/ or content/
    alt: "Alt text for accessibility"
    caption: "Image caption text"
  ```
- `weight: 10` - Custom ordering (lower numbers appear first)
- `lastmod: 2025-01-19T12:00:00-06:00` - Last modification date
- `publishDate: 2025-01-19T12:00:00-06:00` - Schedule for future publication

**Complete example with all optional fields:**
```yaml
---
title: "Your Post Title"
date: 2025-01-19T12:00:00-06:00
draft: false
tags: ["tag1", "tag2"]
categories: ["category"]
description: "Brief description for SEO and previews"
author: "Ayush"
showToc: true
math: true
cover:
  image: ""
  alt: ""
  caption: ""
---
```

### TIL Entries (`content/til/`)

Create new TIL entries with:
```bash
hugo new til/your-til-name.md
```

TIL posts should follow the archetype structure:
1. "What I Learned" section - Brief summary
2. "Details" section - Code examples and explanation
3. "Source/Reference" section - Links or attribution

### Notes (`content/notes/`)

Create new notes with:
```bash
hugo new notes/your-note-name.md
```

Notes are for quick thoughts, observations, and snippets that don't fit into blog posts or TIL entries. They have a simpler structure and are more flexible in format.

**Required front matter:**
```yaml
---
title: "Your Note Title"
date: 2025-01-19T12:00:00-06:00
draft: false
tags: ["notes"]
categories: ["Notes"]
description: "Brief description"
---
```

**Optional front matter:**
- Same as blog posts (author, showToc, math, etc.)

### File Naming

- Use lowercase with hyphens: `my-great-post.md`
- Be descriptive but concise
- Avoid special characters

## Coding Style

### Markdown

- Use ATX-style headers (`## Heading`)
- Use fenced code blocks with language identifiers
- Keep lines reasonably short for readability
- Use reference-style links for repeated URLs

### YAML Configuration

- Use 2-space indentation
- Quote strings containing special characters
- Keep related settings grouped together

### HTML/Templates

- Maintain compatibility with PaperMod theme
- Place custom partials in `layouts/partials/`
- Use Hugo's template functions over raw HTML when possible

## Math Support

Math rendering is enabled via KaTeX. Use:
- Inline math: `$equation$`
- Block math: `$$equation$$`

Ensure the post has `math: true` in front matter, or rely on site-wide `params.math: true`.

## Development Commands

```bash
# Start local server
hugo server

# Start with drafts visible
hugo server -D

# Full rebuild (after config changes)
hugo server --disableFastRender

# Build for production
hugo
```

## Important Notes

1. **Do not edit** files in `public/` - this is auto-generated
2. **Do not edit** theme files in `themes/PaperMod/` directly - use layout overrides
3. **Always test** locally before committing
4. **Set `draft: false`** when content is ready for publication
5. **Date format** in front matter: `2025-01-19T12:00:00-06:00` (RFC 3339)

## Theme Customization

To customize the PaperMod theme:

1. **CSS:** Add custom styles via `assets/css/extended/` or inline in `extend_head.html`
2. **Templates:** Override by creating matching files in `layouts/`
3. **Head additions:** Use `layouts/partials/extend_head.html` for scripts/meta tags

## URLs and Navigation

The site menu is configured in `hugo.yaml` under `menu.main`. Current sections:
- Home (`/`)
- Posts (`/posts/`)
- TIL (`/til/`)
- Notes (`/notes/`)
- Categories (`/categories/`)
- Tags (`/tags/`)
- About (`/about/`)

