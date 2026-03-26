# Implementation Plan: Website Redesign

**PRD**: [docs/prds/Website Redesign.md](../prds/Website%20Redesign.md)
**Created**: 2026-03-26
**Status**: Ready for Implementation

---

## Overview

Transform `index.html` from a static scrollable page into a multi-view personal site with hash-based client-side routing, a persistent nav bar, and markdown-driven content via marked.js. All logic stays in a single `index.html` — no build step, no framework.

The redesign proceeds in four phases, each independently deployable. Phase 1 wires up routing and the nav shell. Phases 2–3 fill in the views (Home+About, then Posts). Phase 4 adds Projects. Each phase is safe to ship while the next is in progress.

---

## Codebase Context

**Existing Patterns Identified:**
- All CSS is inline in `<style>` inside `index.html` — maintain this pattern
- CSS custom properties define the full design token set (colors, fonts)
- `clamp()`-based fluid typography for responsive scaling
- Flexbox layout inside a centered 760px container
- Card components with `--surface` background and `--border` border

**Modules to Modify:**
- `index.html` — add marked.js CDN, routing JS, view switching, nav active state; restructure sections into view containers

**New Modules to Create:**
- `posts/index.json` — post manifest
- `posts/example_post.md` — example post (already exists, needs frontmatter)
- `projects/index.json` — project manifest

**Dependencies:**
- `marked.js` via CDN — markdown rendering (no install needed)

**Integration Points:**
- `window.location.hash` drives which view is visible
- `hashchange` event triggers view switches
- `fetch()` loads `.md` files and JSON manifests at runtime

**Complexity Assessment:**
- Overall complexity: Low
- Risk areas: `fetch()` of local markdown files works fine when served over HTTP (GitHub Pages), but fails when opening `index.html` directly as a `file://` URL — acceptable tradeoff given the deployment target
- Technical debt impact: None — the codebase is clean and minimal

---

## Implementation Phases

### Phase 1: Navigation & Hash Routing Shell **COMPLETED**

**Goal**: Replace the static page structure with a routed multi-view shell. Nav is present on all views; clicking a link switches which view section is visible.

**Why Now?**: Everything else depends on routing. No content work is meaningful until the view-switching skeleton exists.

**Shippable State**: All nav links work; each view renders a placeholder; active nav link is highlighted; deep-linking a hash on load goes to the correct view.

**Tasks**:
1. Add marked.js CDN script tag to `<head>`
   - File: `index.html`
   - Description: `<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>`

2. Restructure HTML body into view containers
   - File: `index.html`
   - Description: Wrap each logical view (`home`, `posts-list`, `post-detail`, `projects`, `about`) in a `<section id="view-*">` with `display:none` by default. Preserve existing hero/about/this-site content inside `#view-home`. Replace the existing flat layout.

3. Replace nav placeholder with full nav markup
   - File: `index.html`
   - Description: Site name links to `#`; three right-side links — `POSTS → #posts`, `PROJECTS → #projects`, `ABOUT → #about`. Monospace, uppercase, consistent with existing style.

4. Add CSS for nav active state and view visibility
   - File: `index.html`
   - Description: `.active` class on nav link sets color to `var(--accent)`. View containers use `display:none` / `display:block` toggled by JS.

5. Implement routing JS — `router()` function and `hashchange` listener
   - File: `index.html`
   - Description: On load and on `hashchange`, parse `window.location.hash`, show the matching view section, hide all others, set active nav link. Handle `#posts/<slug>` as a sub-route of `#posts`.

**Acceptance Criteria**:
- [ ] Clicking each nav link shows the correct (placeholder) view
- [ ] Browser back/forward navigates between views
- [ ] Loading `/?#about` directly shows the About view
- [ ] Active nav link is visually distinct

**Verification**:
```bash
# Open the site in browser and manually click each nav link
# Confirm hashchange works and correct sections appear/disappear
```

---

### Phase 2: Home and About Views **COMPLETED**

**Goal**: Populate the Home view with the existing hero content (cleaned up) and the About view by fetching and rendering `ABOUT_ME.md` with marked.js.

**Why Now?**: These two views require no new data files or fetch patterns beyond one simple markdown load. They validate that marked.js integration works before the more complex posts system is built.

**Shippable State**: Home view shows the hero and "This Site" meta-box. About view shows rendered markdown from `ABOUT_ME.md`.

**Tasks**:
1. Port existing hero and "This Site" content into `#view-home`
   - File: `index.html`
   - Description: Move the current hero section markup (name, tagline, this-site box) into the `#view-home` container. Remove now-duplicate About prose that was previously hardcoded inline — that content lives in `ABOUT_ME.md`.

2. Implement `loadAbout()` — fetch and render `ABOUT_ME.md`
   - File: `index.html`
   - Description: On `#about` route activation, `fetch('ABOUT_ME.md')` then `marked.parse(text)` and inject into `#view-about`. Cache the result to avoid re-fetching on subsequent visits.

3. Style rendered markdown output
   - File: `index.html`
   - Description: Scope CSS to `.prose` wrapper: headings use `--text`, code/pre use `--surface` background and `--mono` font, links use `--accent`, max-width 100% on images, `line-height: 1.7` on paragraphs.

**Acceptance Criteria**:
- [ ] Home view matches the existing hero design
- [ ] About view renders `ABOUT_ME.md` as styled HTML
- [ ] Code blocks in markdown use monospace surface style

---

### Phase 3: Posts System **COMPLETED**

**Goal**: Add the posts list view (cards from `posts/index.json`) and the post detail view (fetch and render a single `.md` file).

**Why Now?**: Posts are the primary content type. The manifest + fetch pattern established here will be reused for Projects in Phase 4.

**Shippable State**: `#posts` lists all posts from the manifest as clickable cards. `#posts/<slug>` renders the full post content.

**Tasks**:
1. Create `posts/index.json` with initial content
   - File: `posts/index.json`
   - Description: JSON array with one entry for the existing `example_post.md`. Shape: `[{ "slug", "title", "date", "description" }]`.

2. Add frontmatter to `posts/example_post.md`
   - File: `posts/example_post.md`
   - Description: Add YAML frontmatter block (`---` delimited) with `title`, `date`, `description`. Keep existing body content.

3. Implement `loadPostsList()` — fetch manifest and render card grid
   - File: `index.html`
   - Description: On `#posts` route, `fetch('posts/index.json')`, render one card per entry showing title, date, description. Each card links to `#posts/<slug>`. Cards match existing card style (`--surface`, `--border`). Sort descending by date.

4. Implement `loadPost(slug)` — fetch and render a single post
   - File: `index.html`
   - Description: On `#posts/<slug>` route, `fetch('posts/${slug}.md')`, strip frontmatter, `marked.parse()` body, inject into `#view-post-detail` with `.prose` wrapper. Show title and date as a header above the content.

5. Add "← Back to Posts" link in post detail view
   - File: `index.html`
   - Description: Static link at top of `#view-post-detail` pointing to `#posts`, styled with `--text-muted`.

**Acceptance Criteria**:
- [ ] `#posts` shows a card for `example_post`
- [ ] Clicking a card navigates to `#posts/example_post`
- [ ] Post detail renders markdown content with correct styling
- [ ] Back link returns to the posts list

---

### Phase 4: Projects System **COMPLETED**

**Goal**: Add the projects list view using `projects/index.json`. Projects have richer metadata (status, repo, tags) than posts.

**Why Now?**: All patterns (manifest fetch, card rendering) are established in Phase 3. This phase is straightforward application of the same approach.

**Shippable State**: `#projects` lists all projects from the manifest as cards showing name, status badge, tags, and optional repo link.

**Tasks**:
1. Create `projects/index.json` with initial content
   - File: `projects/index.json`
   - Description: JSON array. Shape: `[{ "slug", "name", "status", "repo", "tags" }]`. Start with one placeholder project entry.

2. Implement `loadProjectsList()` — fetch manifest and render cards
   - File: `index.html`
   - Description: On `#projects` route, `fetch('projects/index.json')`, render one card per project. Show: name as heading, status as a small monospace badge (color-coded: `active` → accent, `experiment` → muted, `archived` → dimmer), tags as inline labels, repo as a link if present.

3. Add status badge CSS
   - File: `index.html`
   - Description: `.badge` class for status display. Active: `--accent` text. Experiment: `--text-muted`. Archived: `--border` text.

**Acceptance Criteria**:
- [ ] `#projects` renders all entries from `projects/index.json`
- [ ] Status badges are visually distinct by status value
- [ ] Repo link opens in a new tab if present
- [ ] Tags are displayed inline

---

## Dependencies Summary

**External Dependencies to Install**:
- `marked.js` via CDN — `https://cdn.jsdelivr.net/npm/marked/marked.min.js` (no install, loaded at runtime)

**Build Order**:
1. Phase 1 must complete first — routing shell is required for all views
2. Phases 2, 3, and 4 can be done in any order after Phase 1 (they are independent views)
3. Phase 3 before Phase 4 is recommended — posts establish the manifest+fetch pattern that projects reuse

---

## Risk Assessment

**Low Risk Areas**:
- marked.js CDN: stable, widely used, version-pinned via jsDelivr
- `fetch()` for markdown: works correctly on GitHub Pages (HTTP); will fail on `file://` — not a concern for this deployment target

**Known Unknowns**:
- Frontmatter stripping: marked.js does not natively strip YAML frontmatter. A simple regex (`/^---[\s\S]*?---\n/`) suffices and avoids an additional library dependency.

---

## Success Criteria

The implementation is complete when:
1. All five routes (`#`, `#posts`, `#posts/<slug>`, `#projects`, `#about`) render correct views
2. All content is markdown-driven — no prose is hardcoded in HTML
3. The existing dark color scheme and design tokens are unchanged
4. The site deploys to GitHub Pages with no build step
5. `posts/index.json` and `projects/index.json` can be manually extended to add new content

---

## Notes

- Keep all routing logic, CSS, and JS in `index.html`. Do not extract to separate files — that belongs to the future TypeScript migration phase.
- The frontmatter strip regex is the only non-obvious implementation detail. Pattern: `/^---\n[\s\S]*?\n---\n/` applied to raw markdown text before passing to `marked.parse()`.
- When loading content, show no loading spinner — the files are tiny and latency on GitHub Pages is negligible.
