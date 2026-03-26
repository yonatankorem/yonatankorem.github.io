# PRD: Website Redesign

## Overview

Redesign yonatankorem.github.io from a single scrollable page into a multi-view personal site with distinct sections for Posts, Projects, and About Me. All content is markdown-driven. The site remains a no-build, no-framework static site hosted on GitHub Pages.

---

## Goals

- Give Posts, Projects, and About Me dedicated views with clean URLs
- Render all content from markdown files (no hardcoded HTML content)
- Preserve the existing dark color scheme exactly
- Keep the site simple and easy to extend toward a TypeScript migration later

---

## Non-Goals

- No build step or bundler
- No TypeScript (yet — deferred to a future phase)
- No Mermaid diagram support (deferred)
- No server-side rendering or CMS

---

## Views & Routing

Hash-based client-side routing. All routing logic lives in `index.html`.

| Route | View |
|---|---|
| `#` or empty | Home (hero/landing) |
| `#posts` | Posts list |
| `#posts/<slug>` | Single post reading view |
| `#projects` | Projects list |
| `#about` | About Me |

---

## Content Structure

### Posts

- Location: `posts/<slug>.md`
- Slug: derived directly from filename (e.g. `my-first-post.md` → `#posts/my-first-post`)
- Manifest: `posts/index.json` — manually updated when adding a post
- Frontmatter fields: `title`, `date`, `description`

**`posts/index.json` shape:**
```json
[
  { "slug": "my-first-post", "title": "My First Post", "date": "2026-03-26", "description": "A short excerpt shown in the list." }
]
```

### Projects

- Location: `projects/<slug>.md`
- Manifest: `projects/index.json` — manually updated when adding a project
- Frontmatter fields: `name`, `status` (`active` | `experiment` | `archived`), `repo` (URL, optional), `tags` (array)

**`projects/index.json` shape:**
```json
[
  { "slug": "my-project", "name": "My Project", "status": "active", "repo": "https://github.com/...", "tags": ["agentic", "typescript"] }
]
```

### About Me

- Source: `ABOUT_ME.md` (already exists)
- Rendered as markdown, no additional structure

---

## Navigation

Single `<nav>` bar present on all views:

```
yonatan.korem          posts   projects   about
```

- Left: site name in monospace — links to `#` (home)
- Right: three links — `posts`, `projects`, `about` — monospace, uppercase, consistent with existing `h2` style
- Active link highlighted with accent color

---

## Home View

Preserves the current hero section:
- Name heading
- Tagline (Software Architect, 13+ years, agentic development focus)
- "This Site" meta-box: a place to present thoughts, experiments, and projects — maintained with agentic development workflows, configured for repo quality and project maintainability

---

## Markdown Rendering

- Library: **marked.js** via CDN (no install, no build)
- Supports: headings, paragraphs, code blocks, inline code, links, images, blockquotes, lists
- Code blocks: styled with monospace font and surface background, consistent with existing design
- Images: rendered inline, max-width 100%

---

## Footer

Unchanged from current:
- Left: `yonatan.korem@gmail.com`
- Right: `github.com/yonatankorem` (linked)

---

## Design Constraints

Preserve the existing CSS custom properties exactly:

```css
--bg: #0f1117
--surface: #1a1d27
--border: #2a2d3a
--accent: #6c8ebf
--accent-dim: #3d5a80
--text: #e2e6f0
--text-muted: #8890a4
--font: 'Inter', system-ui, -apple-system, sans-serif
--mono: 'JetBrains Mono', 'Fira Code', monospace
```

---

## File Structure

```
index.html            ← single HTML file, all routing + JS + CSS inline
ABOUT_ME.md           ← about me content
posts/
  index.json          ← post manifest
  example_post.md     ← existing example
projects/
  index.json          ← project manifest
```

---

## Future Considerations (Out of Scope)

- TypeScript extraction into separate modules
- Mermaid diagram support
- Auto-generated index.json via build script
- Tags/filtering on posts or projects list
