---
name: sort
description: Sort ignore files, Docker Compose YAML keys, and package.json keys into canonical order
---

# sort

## When to use

When user asks to sort, or after modifying any of:
- `*ignore` files (`.gitignore`, `.dockerignore`, `.prettierignore`, etc.)
- Docker Compose file (`compose.yml`, `docker-compose.yml`)
- `package.json`
- Taskfile (`Taskfile.yml`, `Taskfile.yaml`, `taskfile.yml`, `taskfile.yaml`)

## Instructions

Sort all relevant files found in the project root:
- All `*ignore` files (`.gitignore`, `.dockerignore`, `.prettierignore`, `.eslintignore`, `.npmignore`, etc.)
- Docker Compose file (`compose.yml` or `docker-compose.yml`)
- `package.json`
- Taskfile (`Taskfile.yml`, `Taskfile.yaml`, `taskfile.yml`, `taskfile.yaml`)

Skip any file that does not exist. Each file type has its own sorting rules below.

---

## Ignore file sorting

Applies to all `*ignore` files found in the project root.

### Sections (in this order)

Classify each non-empty, non-comment line into one of four sections:

1. **Dot-folders** — start with `.`, end with `/` (e.g. `.cache/`, `.next/`)
2. **Dot-files** — start with `.`, do not end with `/` (e.g. `.env`, `.DS_Store`)
3. **Folders** — do not start with `.`, end with `/` (e.g. `node_modules/`, `dist/`)
4. **Files** — do not start with `.`, do not end with `/` (e.g. `*.log`, `build.js`)

Within each section, sort entries alphabetically (case-insensitive).

Separate sections with exactly one blank line. No blank line before the first section or after the last.

### Comments and special lines

- Lines starting with `#` → collect, place at the very top, preserve order.
- Negation patterns (`!foo`) → classify by the same rules (strip `!` for classification, keep `!` in output).

### Output format

```
# comments (if any)

.dot-folder-a/
.dot-folder-b/

.dot-file-a
.dot-file-b

folder-a/
folder-b/

file-a
file-b
```

Do not add or remove entries — only reorder and reformat whitespace between sections.

---

## Docker Compose sorting

Target files: `compose.yml`, `docker-compose.yml` (whichever exists in the project root).

### Root key order

```
name

services

networks

volumes
```

Separate each root section with exactly one blank line. Only include root keys that exist in the file.

### `services` — key order within each service

Service names are **not sorted** — preserve their order. Separate services from each other with one blank line.

Within each service, sort keys in this order (omit missing keys, no blank lines between them):

1. `container_name`
2. `image`
3. `hostname`
4. `profiles`
5. `build`
6. `environment`
7. `ports`
8. `networks`
9. `extra_hosts`
10. `volumes`
11. `restart`
12. `tty`
13. `entrypoint`
14. `command`
15. `healthcheck`
16. `depends_on`

### `networks` — key order within each network

Network names are **not sorted** — preserve their order. Separate networks from each other with one blank line.

Within each network definition, sort keys in this order (omit missing keys, no blank lines between them):

1. `name`
2. `driver`
3. `external`

### `volumes` — key order within each volume

Volume names are **not sorted** — preserve their order. Separate volumes from each other with one blank line.

Within each volume definition, sort keys in this order (omit missing keys, no blank lines between them):

1. `name`
2. `driver`
3. `external`

### General rules

- Preserve all values, nested structures, comments, and anchors/aliases exactly.
- Do not add or remove keys — only reorder them.
- Preserve YAML indentation style (2 spaces).

---

## package.json sorting

Target file: `package.json` in the project root.

### Top-level key order

```
name
version
private
author
description
type
license
readme
keywords
homepage
repository
bugs
exports
main
module
files
workspaces
scripts
dependencies
peerDependencies
peerDependenciesMeta
devDependencies
optionalDependencies
publishConfig
[tool config keys — alphabetically]
```

Only include keys that exist in the file — omit missing ones.

**Unknown keys** (not in the canonical list above):
- If clearly a tool config (e.g. `prettier`, `eslint`, `jest`, `bun`, `lint-staged`, `husky`, `babel`, `nyc`, etc.) → place after `publishConfig`, sorted alphabetically among other tool keys.
- Otherwise → place just before tool config keys.

### Nested key sorting

Sort values alphabetically (case-insensitive) inside these keys:

- `scripts`
- `dependencies` — scoped packages (`@scope/pkg`) first, then unscoped, each group alphabetically
- `peerDependencies` — same rule as `dependencies`
- `devDependencies` — same rule as `dependencies`
- `optionalDependencies` — same rule as `dependencies`

All other nested objects/arrays: preserve existing order exactly.

### General rules

- Do not add or remove keys — only reorder.
- Preserve all values, formatting, and JSON structure exactly.
- Use 2-space indent.

---

## Taskfile sorting

Target files (whichever exists): `Taskfile.yml`, `Taskfile.yaml`, `taskfile.yml`, `taskfile.yaml`.

### Root key order

```
version
includes
dotenv
vars
env
set
shopt
silent
output
run
tasks
```

### `tasks` — key order within each task

Task names are **not sorted** — preserve their order. Separate tasks from each other with one blank line. Comments above a task (e.g. `## development tasks ##`) stay in place.

Within each task, sort keys in this order (omit missing keys, no blank lines between them):

1. `desc`
2. `summary`
3. `prompt`
4. `aliases`
5. `internal`
6. `platforms`
7. `run`
8. `watch`
9. `dir`
10. `vars`
11. `env`
12. `silent`
13. `interactive`
14. `ignore_error`
15. `sources`
16. `generates`
17. `preconditions`
18. `deps`
19. `requires`
20. `status`
21. `cmds`

### General rules

- Preserve all values, multiline strings, anchors, aliases, and inline comments exactly.
- Do not add or remove keys — only reorder.
- Preserve YAML indentation style (2 spaces).
