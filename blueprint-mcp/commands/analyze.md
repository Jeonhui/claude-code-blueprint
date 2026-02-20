---
name: analyze
description: "Analyze project tech stack for MCP server generation"
---

Analyze the current **Node.js/JavaScript/TypeScript** project's tech stack (frameworks, databases, API patterns, directory structure) to prepare for custom MCP server generation. Requires `package.json` at the project root.

Parse from $ARGUMENTS:
- `project_path` (optional): Project root path. Defaults to current working directory.

Do NOT use MCP tools — use native file tools only (Read, Glob, Bash).

**IMPORTANT: All output must be written in English.**

## Steps

1. **Read `package.json`** at the project root. If not found, report error and stop.

2. **Detect package manager** by checking lockfiles:
   - `yarn.lock` → yarn
   - `pnpm-lock.yaml` → pnpm
   - `bun.lockb` → bun
   - `package-lock.json` → npm
   - Also check `packageManager` field in package.json

3. **Detect language**:
   - Check for `tsconfig.json` → TypeScript
   - If tsconfig exists, check `src/` for `.ts`/`.tsx` and `.js`/`.jsx` files → mixed if both
   - No tsconfig → JavaScript

4. **Detect frameworks** — check dependencies/devDependencies for these packages, then verify config files:

   | Framework | Packages | Config Files | Category |
   |-----------|----------|-------------|----------|
   | Next.js | `next` | `next.config.{js,mjs,ts}` | fullstack |
   | Express | `express` | — | backend |
   | NestJS | `@nestjs/core` | `nest-cli.json` | backend |
   | Fastify | `fastify` | — | backend |
   | React | `react` | — | frontend |
   | Vue | `vue` | `vue.config.js`, `vite.config.{ts,js}` | frontend |
   | Svelte | `svelte` | `svelte.config.js` | frontend |
   | Hono | `hono` | — | backend |
   | Remix | `@remix-run/node`, `@remix-run/react` | `remix.config.js` | fullstack |
   | Astro | `astro` | `astro.config.{mjs,ts}` | fullstack |
   | Vite | `vite` | `vite.config.{ts,js}` | build |
   | Jest | `jest` | `jest.config.{js,ts}` | testing |
   | Vitest | `vitest` | `vitest.config.ts` | testing |

   Confidence: 100% if config file found, 80% if only package detected.

5. **Detect databases/ORMs** — check dependencies for:

   | ORM | Packages | Config Files | Database |
   |-----|----------|-------------|----------|
   | Prisma | `prisma`, `@prisma/client` | `prisma/schema.prisma` | PostgreSQL/MySQL/SQLite |
   | TypeORM | `typeorm` | `ormconfig.{json,js,ts}` | SQL |
   | Drizzle | `drizzle-orm` | `drizzle.config.{ts,js}` | PostgreSQL/MySQL/SQLite |
   | Mongoose | `mongoose` | — | MongoDB |
   | Sequelize | `sequelize` | `.sequelizerc`, `config/config.json` | SQL |
   | Knex | `knex` | `knexfile.{js,ts}` | SQL |
   | Kysely | `kysely` | — | SQL |

6. **Detect API patterns** — check dependencies for:

   | Pattern | Framework | Packages | Entry Point Hints |
   |---------|-----------|----------|------------------|
   | GraphQL | Apollo Server | `@apollo/server`, `apollo-server` | `src/graphql`, `src/schema.graphql` |
   | GraphQL | GraphQL Yoga | `graphql-yoga` | `src/graphql` |
   | GraphQL | Type-GraphQL | `type-graphql` | `src/resolvers` |
   | tRPC | tRPC | `@trpc/server` | `src/server/trpc.ts`, `src/trpc` |
   | gRPC | gRPC | `@grpc/grpc-js` | `protos/` |
   | WebSocket | Socket.IO | `socket.io` | `src/socket`, `src/ws` |
   | WebSocket | ws | `ws` | — |
   | REST | Express | `express` | `src/routes`, `src/api`, `src/controllers` |
   | REST | Fastify | `fastify` | `src/routes`, `src/api` |
   | REST | Next.js API | `next` | `pages/api`, `src/app/api`, `app/api` |
   | REST | Hono | `hono` | `src/routes`, `src/api` |

7. **Scan directory structure** — check which of these directories exist:
   `src`, `app`, `pages`, `api`, `lib`, `utils`, `server`, `client`, `components`, `services`, `models`, `controllers`, `routes`, `middleware`, `config`, `prisma`, `graphql`, `public`, `test`, `tests`, `__tests__`

8. **Output report** in this format:

```
# Project Analysis: <project-name>

**Root:** <path>
**Language:** TypeScript | JavaScript | mixed
**Package Manager:** npm | yarn | pnpm | bun

## Frameworks (N)
- **<Name>** <version> [<category>] (confidence: N%) - configs: <files>

## Databases (N)
- **<ORM>** <version> → <database> (confidence: N%)

## API Patterns (N)
- **<PATTERN>** via <framework> (confidence: N%) - entry: <paths>

## Directory Structure
**Source dir:** <src | app | pages | root>
**Relevant dirs:** <comma-separated list>
```

$ARGUMENTS
