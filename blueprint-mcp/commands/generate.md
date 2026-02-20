---
name: generate
description: "Generate a custom MCP server based on detected tech stack"
---

Generate a custom MCP server tailored to the project's tech stack. Analyzes the project first, then creates an MCP server with framework-specific tools.

Parse from $ARGUMENTS:
- `project_path` (optional): Project root path. Defaults to current working directory.
- `--dry-run` (optional): Show what would be generated without writing files.

Do NOT use MCP tools — use native file tools only (Read, Write, Glob, Bash).

**IMPORTANT: All generated file content and output must be written in English.**

## Steps

1. **Analyze the project** — follow the same analysis logic as `/blueprint-mcp:analyze` (read package.json, detect frameworks, databases, API patterns, directory structure).

2. **Determine server name**: `mcp-<project-name>` (from package.json `name` field)

3. **Select tools based on detected tech stack** (skip detections with confidence < 60%):

   **Next.js detected →** generate:
   - `nextjs-routes`: Scan and list Next.js routes (App Router / Pages Router)
   - `nextjs-api-test`: Test a Next.js API route via HTTP request

   **Express / Fastify / Hono detected →** generate:
   - `express-routes`: Scan source files for route definitions (`app.get`, `router.post`, etc.)
   - `express-api-test`: Test API endpoints via HTTP request

   **Prisma detected →** generate:
   - `prisma-schema`: Read and analyze Prisma schema (models, enums, relations)
   - `prisma-studio`: Launch Prisma Studio for database browsing

   **GraphQL detected →** generate:
   - `graphql-schema`: Find and analyze `.graphql`/`.gql` schema files
   - `graphql-query`: Execute GraphQL queries against local dev server

   **Always add generic tools:**
   - `<project-name>-env-check`: Check environment variables vs `.env.example`
   - `<project-name>-deps`: Check for outdated dependencies via `npm outdated`

4. **Check if server already exists** at `.claude/mcp-internal/<server-name>/`. If so, report and stop.

5. **If `--dry-run`**, output a preview and stop:
   ```
   # Dry Run: <server-name>

   **Would create:** .claude/mcp-internal/<server-name>

   ## Tools to Generate (N)
   - **<tool-name>**: <description> [<category>]

   ## Files to Create
   - package.json
   - index.js
   - tools/<tool-name>.js
   - README.md
   ```

6. **Generate files** in `.claude/mcp-internal/<server-name>/`:

   **`package.json`**:
   ```json
   {
     "name": "<server-name>",
     "version": "0.1.0",
     "description": "Auto-generated MCP server for <project-name>",
     "type": "module",
     "main": "index.js",
     "scripts": { "start": "node index.js" },
     "dependencies": {
       "@modelcontextprotocol/sdk": "^1.12.1",
       "zod": "^3.24.2"
     },
     "_comment": "Dependency versions are baselines. Run `npm outdated` after install to check for updates."
   }
   ```
   > **Note:** The dependency versions above are baselines as of the command's creation date. After generating, consider running `npm view @modelcontextprotocol/sdk version` and `npm view zod version` to check for latest compatible versions.

   **`index.js`**: MCP server entry point using `@modelcontextprotocol/sdk` with StdioServerTransport. Import and register all generated tool files from `./tools/<name>.js`.

   **`tools/<name>.js`**: Each tool file exports a `register(server)` function that calls `server.tool()` with:
   - Tool name and description
   - Zod parameter schema
   - Handler implementing the tool's logic

   Generate the actual tool implementation code — not stubs. Each tool should be a working implementation based on the templates described in step 3.

   **`README.md`**: Document the generated server name, tools list, setup instructions, and customization guide.

7. **Report results**:
   ```
   # Generated: <server-name>

   **Location:** .claude/mcp-internal/<server-name>
   **Tools:** N

   ## Files Created
   - <list>

   ## Next Steps
   1. Run `/blueprint-mcp:setup` to install dependencies and register in .mcp.json
   2. Or manually: cd .claude/mcp-internal/<server-name> && npm install
   3. Customize tools in tools/
   ```

$ARGUMENTS
