---
name: claudemd-generate
description: "Analyze project and generate CLAUDE.md"
---

Analyze the project structure and generate a comprehensive CLAUDE.md file. Do NOT use MCP tools.

**IMPORTANT: All generated file content and output must be written in English.**

Options in $ARGUMENTS:
- `--target root` or `--target claude-dir` (default: claude-dir)
- `--dry-run` to preview without writing

## Steps

1. **Analyze `package.json`**: Extract name, description, scripts, dependencies, devDependencies
2. **Detect package manager**: Check for `bun.lockb` (bun), `pnpm-lock.yaml` (pnpm), `yarn.lock` (yarn), else npm
3. **Detect frameworks** from dependencies: Next.js, React, Vue, Express, Fastify, NestJS, Prisma, Vitest, Jest, Tailwind, etc.
4. **Scan directories**: Check for src/, app/, pages/, lib/, components/, test/, etc.
5. **Detect monorepo**: Check for workspaces in package.json, pnpm-workspace.yaml, lerna.json, nx.json, turbo.json
6. **Check TypeScript**: Look for tsconfig.json

7. **Generate content** with sections:
   - `# <project-name>` + description
   - `## Tech Stack` — Language, package manager, frameworks by category
   - `## Project Structure` — Directory listing
   - `## Monorepo` (if applicable)
   - `## Common Commands` — dev, build, start, test, lint, type-check
   - `## Coding Conventions` — Based on detected stack

8. **Write the file using block preservation**:
   - Target path: `--target root` → `./CLAUDE.md`, `--target claude-dir` (default) → `.claude/CLAUDE.md`
   - Wrap ALL generated content between markers:
     ```
     <!-- CCB:GENERATED:BEGIN -->
     (generated content here)
     <!-- CCB:GENERATED:END -->
     ```
   - **If file exists AND contains `<!-- CCB:GENERATED:BEGIN -->`**: replace ONLY the content between `BEGIN` and `END` markers. Preserve everything outside the markers untouched.
   - **If file exists but does NOT contain the markers**: append the marker block to the end of the file. Do NOT modify existing content.
   - **If file does not exist**: create the file with the marker block.
   - If `--dry-run`, show preview only.

9. Report what was generated (created new / updated existing block / appended to existing).

$ARGUMENTS
