---
description: Generate a new VitePress docs page or full section (index + pages + sidebar + nav) following the learning-portal conventions. Usage: /vitepress-page "topic description" or /vitepress-page "new section: Section Name with pages: page1, page2, page3"
argument-hint: "[topic or 'new section: Name with pages: p1, p2, p3']"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# VitePress Page / Section Generator

You are generating content for a VitePress-based learning portal.
The user's request is: **$ARGUMENTS**

---

## Step 1 — Detect Mode

Parse $ARGUMENTS to determine which mode to run:

- **Single page mode**: argument describes a topic (e.g. "Docker networking", "Java records")
  → Create one `.md` content file and wire it into an existing section sidebar
- **New section mode**: argument starts with "new section:" (e.g. "new section: DevOps with pages: ci-cd, monitoring, logging")
  → Create: section folder, index.md, all listed page .md files, sidebar .js file, and update config.mjs

If the mode is ambiguous, ask the user before proceeding.

---

## Step 2 — Explore the Project

Before writing anything, read these files to understand the current state:

1. `docs/.vitepress/config.mjs` — find existing nav sections and sidebar routes
2. The relevant sidebar file in `docs/.vitepress/sidebars/` — understand existing pages
3. One or two existing content `.md` files in the target section — match the writing style and depth

Use Glob to discover existing sections: `docs/.vitepress/sidebars/*.js`

---

## Step 3 — Apply Conventions

### Frontmatter (every content page)

```yaml
---
title: Page Title
description: One-sentence description for SEO and tooltips
category: section-name
pageClass: layout-section-name
difficulty: beginner | intermediate | advanced
tags: [tag1, tag2, tag3]
related:
  - /section/related-page-1
  - /section/related-page-2
estimatedMinutes: 15
---
```

### Page Structure (always in this order)

```markdown
# Page Title

<DifficultyBadge level="intermediate" />

One-paragraph intro: what this is and why it matters.

---

## First Major Section

Theory first — explain the concept clearly.

```java
// Code example immediately follows theory
// Use real, runnable examples — not pseudocode
// Include comments explaining non-obvious lines
```

---

## Second Major Section

...repeat Theory → Code pattern...

---

## Common Pitfalls / Comparison Table (if applicable)

| | Option A | Option B |
|--|---------|---------|
| **Speed** | O(1) | O(log n) |

---

## Quiz

→ [Test your knowledge](/quizzes/mixed-review)
```

### Section Index Page (index.md)

```markdown
# Section Title

One-paragraph overview of what this section covers and why it matters.

## Why Developers Need This

- **Key point** — brief explanation
- **Key point** — brief explanation

---

## Section Map

| Page | What You'll Learn |
|------|------------------|
| [Page Title](/section/page) | Topics covered |

---

## Key Concepts Glossary (optional)

| Term | Definition |
|------|-----------|
| **Term** | Definition |

---

## Learning Path

```
Topic A  →  Topic B  →  Topic C
```

## Prerequisites

- List what the reader should know first
```

### Sidebar File (`docs/.vitepress/sidebars/<section>.js`)

```js
export const <section>Sidebar = [
  {
    text: 'Section Display Name',
    items: [
      { text: 'Overview', link: '/<section>/' },
      { text: 'Page Title', link: '/<section>/page-slug' },
    ],
  },
]
```

### config.mjs Changes (new section only)

Add import at the top:
```js
import { <section>Sidebar } from './sidebars/<section>.js'
```

Add sidebar route in the `sidebar:` object:
```js
'/<section>/': <section>Sidebar,
```

Add nav item in the appropriate dropdown in the `nav:` array:
```js
{ text: 'Section Name', link: '/<section>/' },
```

---

## Step 4 — Content Quality Rules

Apply these rules to every page generated:

- **Depth**: Match the depth of existing pages in the portal — comprehensive, not superficial
- **Code examples**: All Java code must be syntactically correct and realistic; use proper Spring/Java idioms
- **No filler**: Never write "In this section we will..." — start directly with the concept
- **Containers**: Use VitePress containers where appropriate:
  - `::: tip` — best practice or helpful shortcut
  - `::: info` — neutral supplemental info
  - `::: warning` — common mistake or gotcha
  - `::: danger` — security issue or data-loss risk
  - `::: details Summary text` — collapsible detail
- **Tables**: Use markdown tables for comparisons (e.g. ArrayList vs LinkedList)
- **Dividers**: Use `---` between every major `##` section
- **Language tags**: Always specify language on code blocks — `java`, `sql`, `yaml`, `bash`, `json`, `xml`, `js`
- **DifficultyBadge**: Always include `<DifficultyBadge level="..." />` on line 3 of every content page (not index pages)
- **Quiz link**: Always end content pages with `## Quiz\n\n→ [Test your knowledge](/quizzes/mixed-review)`
- **Related links**: Fill the `related:` frontmatter with 2–4 real existing pages

---

## Step 5 — Output

For **single page mode**:
1. Create `docs/<section>/<slug>.md`
2. Add an entry to `docs/.vitepress/sidebars/<section>.js`

For **new section mode**:
1. Create `docs/<section>/` folder with `index.md` + all listed pages
2. Create `docs/.vitepress/sidebars/<section>.js`
3. Edit `docs/.vitepress/config.mjs` — add import, sidebar route, and nav entry

After all files are written, confirm:
- Run `npm run docs:build` to verify no build errors
- List every file created/modified
