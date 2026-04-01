# claude-skills

A collection of custom skills (slash commands) for [Claude Code](https://claude.ai/code).

## What are skills?

Skills are reusable prompts invoked as `/skill-name` in any Claude Code session. Each skill lives in its own subdirectory as a `SKILL.md` file and can accept arguments typed after the command name.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [syllabus-collector](./syllabus-collector/SKILL.md) | `/syllabus-collector` | Researches Polish school curriculum and generates a VitePress-ready outline |
| [vitepress-page](./vitepress-page/SKILL.md) | `/vitepress-page` | Generates VitePress pages or full sections inside a docs project |

## Usage

Point Claude Code at this directory as your skills source, then invoke any skill with:

```
/syllabus-collector "klasa 5 matematyka"
/vitepress-page "new section: DevOps with pages: ci-cd, monitoring"
```

## Adding a skill

1. Create a subdirectory: `<skill-name>/`
2. Add a `SKILL.md` with frontmatter and a prompt body:

```markdown
---
description: Short description shown in the skill picker
argument-hint: "[hint for the user]"
allowed-tools: Read, Write, Bash, WebSearch, ...
---

# Skill Name

Your prompt here. Use $ARGUMENTS where the user's input should appear.
```
