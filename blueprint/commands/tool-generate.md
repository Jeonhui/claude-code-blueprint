---
name: tool-generate
description: "Generate a custom MCP server template in .claude/mcp-internal/"
---

Generate a custom MCP server template for project-specific tools. Parse from $ARGUMENTS:
- `tool_name` (required): Name for the tool (lowercase, hyphens)
- `--description <text>` (optional, default: "Custom MCP tool")

Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All generated file content and output must be written in English.**

## Steps

1. Validate tool name: lowercase letters, numbers, hyphens, starts with letter
2. Check if `.claude/mcp-internal/<tool_name>/` already exists — if so, report and stop
3. Create the directory
4. Write three files:

**`package.json`**:
```json
{
  "name": "mcp-tool-<tool_name>",
  "version": "0.1.0",
  "description": "<description>",
  "type": "module",
  "main": "index.js",
  "scripts": { "start": "node index.js" },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.1",
    "zod": "^3.24.2"
  }
}
```

**`index.js`**:
```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "mcp-tool-<tool_name>",
  version: "0.1.0",
});

server.tool(
  "<tool_name>-example",
  "Example tool — replace with your implementation",
  { input: z.string().describe("Input parameter") },
  async ({ input }) => {
    return {
      content: [{ type: "text", text: `Received: ${input}` }],
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

**`README.md`**: Setup and usage instructions.

5. Report success with next steps:
   - Edit `index.js` to implement tool logic
   - Run `npm install` in the tool directory
   - Use `/blueprint:tool-link` to register in `.mcp.json`

$ARGUMENTS
