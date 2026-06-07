# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- AUTO-MANAGED: project-description -->
## Overview

Personal collection of Claude Code skills for Ondřej Javorský. Each skill is a reusable AI behavior packaged as a directory with a `SKILL.md` file. Skills are invoked via Claude Code's skill system using the `name` field from the frontmatter.

Current skills:
- **form** — Formats/sorts project config files (ignore files, Dockerfile, Docker Compose, package.json, Taskfile) into canonical order
- **git** — Generates Conventional Commits-formatted commit messages from staged changes

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: build-commands -->
## Build & Development Commands

No build system. Skills are markdown files — edit directly.

```sh
# View all skills
ls */SKILL.md

# Check git status
git status

# Commit (use the git skill)
git add <files> && git commit
```

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: architecture -->
## Architecture

```
skills/
├── form/
│   └── SKILL.md        # File formatting/sorting skill
├── git/
│   └── SKILL.md        # Commit message generation skill
└── CLAUDE.md           # This file
```

Each skill directory contains exactly one `SKILL.md`. Adding a new skill = create `<name>/SKILL.md`.

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: conventions -->
## Code Conventions

**SKILL.md frontmatter** (required):
```yaml
---
name: skill-name
description: One-sentence trigger description used by Claude to decide when to invoke this skill.
---
```

**Description field**: Must be a precise trigger sentence — Claude reads it to decide when to invoke. Vague descriptions cause wrong invocations.

**Sections in skill body** (recommended order):
1. `## When to use` — explicit trigger conditions
2. `## Instructions` — step-by-step behavior

**Git commits**: Conventional Commits format (`docs:`, `feat:`, `fix:`, `refactor:`, `chore:`). No `Co-authored-by` trailer. See `git` skill.

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: patterns -->
## Detected Patterns

- **Opinionated ordering**: Skills define a canonical ordering for file sections/keys, not just alphabetical sort. Order encodes meaning (e.g. Taskfile task sections follow development → deployment → test workflow).
- **Critical rule callouts**: Rules that are easy to violate but have high impact are marked with `⚠️ CRITICAL` in skill docs. Current critical rules:
  - Never sort `scripts` in package.json
  - Never sort `vars`/`env` in Taskfile
  - Docker Compose volume names must be prefixed with service name (`<service-name>-<purpose>`)
  - No blank lines between keys within a single Taskfile task
  - Docker Compose nested mappings (`environment`, `ports`, `volumes`) preserve original order — not sorted
- **Filesystem-aware classification**: Ignore-file entries without trailing `/` are classified as folder vs file by checking the actual project filesystem, not just the pattern syntax.
- **Comment anchoring**: In Docker Compose, inline comments attach to the entry below them and move with it during reordering; group-header comments stay at the top of their block.
- **Example-driven specs**: Each formatting rule includes a before/after example block.
- **Scope discipline**: Skills operate on existing content only — never add or remove entries, only reorder/reformat.

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: git-insights -->
## Git Insights

All commits use `docs:` type — this repo is documentation-only (skills are markdown). Commit style: short imperative summary, no body bullets needed for single-file doc changes.

<!-- END AUTO-MANAGED -->

<!-- MANUAL -->
## Custom Notes

Add project-specific notes here. This section is never auto-modified.

<!-- END MANUAL -->
