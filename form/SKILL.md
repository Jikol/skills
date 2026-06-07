---
name: form
description: Use this skill whenever the user asks to format, sort, reorder, organize, clean up, or tidy any of these project files - ignore files (.gitignore, .dockerignore, .prettierignore, etc.), Dockerfile, Docker Compose (compose.yml, docker-compose.yml), package.json, or Taskfile (Taskfile.yml, taskfile.yaml). Also use proactively after editing or modifying any of these files to keep them in canonical order. Applies opinionated section ordering, alphabetical sorting within sections, and consistent blank-line separation. Only reorders existing content - never adds or removes entries.
---

# form

## When to use

When user asks to format or sort any of:

- `*ignore` files (`.gitignore`, `.dockerignore`, `.prettierignore`, etc.)
- `Dockerfile`
- Docker Compose file (`compose.yml`, `docker-compose.yml`)
- `package.json`
- Taskfile (`Taskfile.yml`, `Taskfile.yaml`, `taskfile.yml`, `taskfile.yaml`)

Or after modifying any such file.

## Instructions

Format all relevant files found in the project root. Skip any file that does not exist. Each file type has its own rules below.

---

## Ignore file sorting

Applies to all `*ignore` files found in the project root.

### Sections (in this order)

Classify each non-empty, non-comment line into one of four sections:

1. **Dot-folders** — start with `.`, refer to a directory (e.g. `.cache/`, `.next/`, `.git`)
2. **Dot-files** — start with `.`, refer to a file (e.g. `.env`, `.DS_Store`, `.gitignore`)
3. **Folders** — do not start with `.`, refer to a directory (e.g. `node_modules/`, `dist`, `build/`)
4. **Files** — do not start with `.`, refer to a file (e.g. `*.log`, `build.js`, `README.md`)

### Classifying folder vs file

Ignore-file patterns often omit the trailing `/` even when targeting directories (e.g. `.git`, `node_modules`, `dist`). To classify correctly:

1. If entry ends with `/` → folder.
2. Else check the project filesystem — if a directory with that name exists at project root → folder; otherwise → file.
3. For glob patterns (`*.log`, `**/*.local`, `*.tmp`) and negations referring to globs → treat as file unless they explicitly end with `/`.
4. If in doubt and filesystem check is inconclusive → treat as file.

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

## Dockerfile formatting

Target file: `Dockerfile` in the project root.

### Blocks

Block = optional comment line immediately above + `FROM` line + all instructions until the next `FROM` or end of file.

Separate blocks from each other with exactly one blank line.

### Instructions within a block

**Do not reorder instructions** — their order is cache-sensitive and must be preserved.

Formatting rules (apply to **every** instruction including the first one after `FROM`):

- Consecutive instructions with the **same keyword** (e.g. multiple `COPY`, multiple `RUN`) → no blank line between them (grouped together).
- Instructions with a **different keyword** than the previous one → one blank line before them. This includes the first instruction after `FROM` if its keyword is not `FROM`.
- No trailing blank line at the end of a block.

### Example

```dockerfile
# base stage
FROM node:20 AS base

WORKDIR /app

COPY package.json .
COPY package-lock.json .

RUN npm ci

# builder stage
FROM base AS builder

COPY . .

RUN npm run build

# runner stage
FROM node:20-alpine AS runner

COPY --from=builder /app/dist ./dist
COPY package.json .

RUN npm ci --omit=dev

CMD ["node", "dist/index.js"]
```

### General rules

- Preserve all instruction arguments and here-docs exactly.
- Do not add or remove instructions — only reformat blank lines.
- Do not sort blocks or instructions.

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

### `volumes` — naming convention

> ⚠️ **CRITICAL: Volume names must be prefixed with the service name that uses them.** Format: `<service-name>-<purpose>`. Both the top-level volume key and its `name` field must follow this pattern.

Example:

```yaml
services:
  my-service:
    volumes:
      - my-service-data:/data/path

volumes:
  my-service-data:
    name: ${DOCKER_NAME:?error}-my-service-data
    external: true
```

Rule: top-level key = `<service-name>-<purpose>`, `name` value = `${DOCKER_NAME:?error}-<service-name>-<purpose>`.

### `volumes` — key order within each volume

Volume names are **not sorted** — preserve their order. Separate volumes from each other with one blank line.

Within each volume definition, sort keys in this order (omit missing keys, no blank lines between them):

1. `name`
2. `driver`
3. `external`

### Comment handling

Comments inside nested mappings (e.g. inside `environment`, `volumes`, `ports`) stay attached to the entry **below** them. When sorting or reordering, the comment moves with its attached entry. Standalone group-header comments at the very top of a block (e.g. `# PROGRAM ENVS` at the start of `environment:`) stay at the top of that block.

### General rules

- Preserve all values, nested structures, comments, and anchors/aliases exactly.
- Do not add or remove keys — only reorder them.
- Do not sort the contents of nested mappings (e.g. `environment`, `ports`, `volumes`) — preserve their order unless explicitly stated.
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

> ⚠️ **CRITICAL: NEVER sort `scripts`.** Preserve script order EXACTLY as written by the user. Script names like `dev`, `prod`, `test`, `form`, `lint` follow a workflow order, not alphabetical. Reordering them is a bug.

Sort values alphabetically (case-insensitive) **only** inside these keys:

- `dependencies` — scoped packages (`@scope/pkg`) first, then unscoped, each group alphabetically
- `peerDependencies` — same rule as `dependencies`
- `devDependencies` — same rule as `dependencies`
- `optionalDependencies` — same rule as `dependencies`

All other nested objects/arrays (including `scripts`, `engines`, `exports`, etc.): preserve existing order exactly.

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

Separate every **top-level** root key from the next with exactly one blank line.

**Do not insert blank lines between nested entries** (e.g. between individual keys inside `env`, `vars`, `dotenv`, or any other root key's contents). Blank line rule applies only at the top level.

> ⚠️ **CRITICAL: Do NOT sort `vars` or `env` entries alphabetically.** Preserve their order exactly as written. Variable declaration order often encodes dependency (a later var may reference an earlier one) or intentional grouping.

Example (correct):

```yaml
version: 3

dotenv: [".env.local"]

env:
  DEFAULT_DOCKER_NAME: local
  DEFAULT_DOCKER_IMAGE: amqp-simulator
  DEFAULT_DOCKER_TAG: latest

tasks:
  ...
```

### `tasks` — task ordering

If section comments group tasks (see below), reorder the **sections** in this fixed order:

1. `## development tasks ##`
2. `## deployment tasks ##`
3. `## test tasks ##`

Within each section, reorder tasks in this fixed order (omit missing ones):

- **development**: `init` → `docker:dev`
- **deployment**: `docker:build` → `docker:compose`
- **test**: preserve original order

If no section comments exist, preserve all task order entirely.

Separate tasks from each other with exactly one blank line.

### `tasks` — key order within each task

> ⚠️ **CRITICAL: NO blank lines between keys within a single task.** Every key (`desc`, `summary`, `aliases`, `platforms`, `env`, `vars`, `cmds`, etc.) inside a task MUST be on **consecutive lines** without any blank line separators. This rule overrides any other formatting instinct. If you see a blank line between two task-level keys, REMOVE IT.

**Wrong:**

```yaml
  init:
    desc: >
      Initialize project

    aliases: [i]

    platforms: [linux]

    cmds:
      - cmd: bun install
```

**Correct:**

```yaml
  init:
    desc: >
      Initialize project
    aliases: [i]
    platforms: [linux]
    cmds:
      - cmd: bun install
```

#### Section comments

Tasks may be grouped into named sections. Each section has:

- **Header** `## <label> ##` — immediately before the first task in the section, no blank line between header and task
- **Separator** `## <dashes> ##` — immediately after the last task in the section, no blank line between task and separator; separator closes the section and its dash count matches the header of **that same section**

```yaml
## development tasks ##
init: ...

docker:dev: ...
## ----------------- ##
## deployment tasks ##
docker:build: ...

docker:compose: ...
## ---------------- ##
```

Dash count = number of characters (including spaces) in `<label>`:

- `## foo bar ##` → "foo bar" = 7 chars → `## ------- ##`
- `## fizz fizz bazz ##` → "fizz fizz bazz" = 14 chars → `## -------------- ##`
- `## development tasks ##` → "development tasks" = 17 chars → `## ----------------- ##`
- `## deployment tasks ##` → "deployment tasks" = 16 chars → `## ---------------- ##`

When reformatting, recalculate and correct the separator dash count to match the header of its section.

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
