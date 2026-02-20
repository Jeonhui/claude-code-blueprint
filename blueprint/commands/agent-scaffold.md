---
name: agent-scaffold
description: "Scaffold a new Anthropic Agent SDK project"
---

Scaffold a new Anthropic Agent SDK project with TypeScript template. Parse from $ARGUMENTS:
- `project_name` (required): Name for the agent project
- `description` (optional): Project description

Do NOT use MCP tools — use native file tools only.

**IMPORTANT: All generated file content and output must be written in English.**

## Steps

1. Determine output directory: `./<project_name>` in current working directory
2. Check if directory already exists — if so, report and stop
3. Create the project structure with these files:

**`package.json`**:
```json
{
  "name": "<project_name>",
  "version": "0.1.0",
  "description": "<description>",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsc --watch"
  },
  "dependencies": {
    "@anthropic-ai/sdk": "^0.52.0"
  },
  "devDependencies": {
    "typescript": "^5.8.0",
    "@types/node": "^22.0.0"
  }
}
```

**`tsconfig.json`**:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "declaration": true
  },
  "include": ["src/**/*"]
}
```

**`src/index.ts`**:
```typescript
import Anthropic from "@anthropic-ai/sdk";
import { tools, processToolCall } from "./tools.js";

const client = new Anthropic();

async function runAgent(userMessage: string): Promise<string> {
  console.log(`\nUser: ${userMessage}`);

  const messages: Anthropic.MessageParam[] = [
    { role: "user", content: userMessage },
  ];

  let response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 4096,
    system: "You are a helpful assistant with access to tools. Use them when needed.",
    tools,
    messages,
  });

  while (response.stop_reason === "tool_use") {
    const toolUseBlocks = response.content.filter(
      (block): block is Anthropic.ToolUseBlock => block.type === "tool_use"
    );

    const toolResults: Anthropic.ToolResultBlockParam[] = [];
    for (const toolUse of toolUseBlocks) {
      console.log(`\nTool: ${toolUse.name}(${JSON.stringify(toolUse.input)})`);
      const result = await processToolCall(toolUse.name, toolUse.input as Record<string, unknown>);
      console.log(`Result: ${result}`);
      toolResults.push({
        type: "tool_result",
        tool_use_id: toolUse.id,
        content: result,
      });
    }

    messages.push({ role: "assistant", content: response.content });
    messages.push({ role: "user", content: toolResults });

    response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4096,
      system: "You are a helpful assistant with access to tools. Use them when needed.",
      tools,
      messages,
    });
  }

  const textBlock = response.content.find(
    (block): block is Anthropic.TextBlock => block.type === "text"
  );
  const answer = textBlock?.text ?? "";
  console.log(`\nAssistant: ${answer}`);
  return answer;
}

// Main
const query = process.argv[2] ?? "Hello! What tools do you have available?";
runAgent(query).catch(console.error);
```

**`src/tools.ts`**:
```typescript
import Anthropic from "@anthropic-ai/sdk";

// Tool definitions for the Claude API
export const tools: Anthropic.Tool[] = [
  {
    name: "get_weather",
    description: "Get the current weather for a given location.",
    input_schema: {
      type: "object" as const,
      properties: {
        location: {
          type: "string",
          description: "City name, e.g. 'San Francisco, CA'",
        },
      },
      required: ["location"],
    },
  },
];

// Process a tool call and return the result string
export async function processToolCall(
  toolName: string,
  toolInput: Record<string, unknown>
): Promise<string> {
  switch (toolName) {
    case "get_weather": {
      const location = toolInput.location as string;
      // Replace with actual API call in production
      return JSON.stringify({
        location,
        temperature: "72°F",
        condition: "Sunny",
      });
    }
    default:
      return JSON.stringify({ error: `Unknown tool: ${toolName}` });
  }
}
```

**`README.md`**: Setup and usage instructions.

4. Report created files and next steps:
   - `cd <project_name>`
   - `npm install`
   - `npm run build`
   - `npm start`

$ARGUMENTS
