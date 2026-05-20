---
name: form
description: Format files that lack a standard formatter (e.g. Dockerfile)
---

# form

## When to use

When user asks to format any of:
- `Dockerfile`

Or after modifying any such file.

## Instructions

Format all relevant files found in the project root. Skip any file that does not exist. Each file type has its own formatting rules below.

---

## Dockerfile formatting

Target file: `Dockerfile` in the project root.

### Blocks

Block = optional comment line immediately above + `FROM` line + all instructions until the next `FROM` or end of file.

Separate blocks from each other with exactly one blank line.

### Instructions within a block

**Do not reorder instructions** — their order is cache-sensitive and must be preserved.

Formatting rules:
- Consecutive instructions with the **same keyword** (e.g. multiple `COPY`, multiple `RUN`) → no blank line between them (grouped together).
- Instructions with a **different keyword** than the previous one → one blank line before them.
- No blank line immediately after the `FROM` line.
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
