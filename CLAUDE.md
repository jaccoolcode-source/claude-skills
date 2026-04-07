2# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of Claude Code **skills** — reusable slash commands that users can invoke as `/skill-name` in any Claude Code session. Each skill lives in its own subdirectory as a `SKILL.md` file.

## Skill file format

Every `SKILL.md` starts with YAML frontmatter, then a freeform prompt body:

```markdown
---
description: One-line description shown in skill picker
argument-hint: "[hint shown to user when typing the command]"
allowed-tools: Read, Write, Edit, Glob, Bash, WebSearch, WebFetch, ...
---

# Skill Name

Prompt body — this is what Claude receives when the skill is invoked.
$ARGUMENTS is replaced with whatever the user typed after the command.
```

- `description` — shown in `/` command picker; keep it concise and include usage syntax
- `argument-hint` — displayed inline as the user types; describe expected input
- `allowed-tools` — comma-separated list of tools the skill is permitted to use; only list what the skill actually needs

## Adding a new skill

1. Create a new subdirectory: `<skill-name>/`
2. Add `SKILL.md` with the frontmatter above and a prompt body
3. The skill becomes available as `/<skill-name>` in Claude Code sessions that load this skills directory

## Existing skills

- **`syllabus-collector`** — researches Polish school curriculum (MEN official sources + educational portals), builds a detailed `konspekt-{slug}.md` outline, and generates a `CLAUDE.md` configured for a VitePress learning portal. Designed around a pedagogical tone for sensitive children.
- **`vitepress-page`** — generates a single VitePress content page or a full section (index + pages + sidebar + nav wired into `config.mjs`) inside an existing VitePress docs project. Reads existing pages to match writing style before generating.
- **`dockerize`** — analyzes a project (Node.js, Python, Go, Java, static) and generates a production-ready `Dockerfile`, `docker-compose.yml` with Traefik reverse proxy labels, and `.env.example`. Detects companion services (Postgres, Redis) automatically.
