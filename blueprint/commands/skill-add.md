---
name: skill-add
description: "Install a built-in skill package into .claude/skills/"
---

Install a built-in skill package. The skill name is provided in $ARGUMENTS.

Do NOT use MCP tools — use native file tools (Write, Bash mkdir) only.

**IMPORTANT: All generated file content and output must be written in English.**

## Available Skills

### rest-api-guide
Create `.claude/skills/rest-api-guide/SKILL.md`:
```markdown
# REST API Guide

## Core Principles
- Use plural nouns for resource endpoints (e.g., /users, /posts)
- Use HTTP methods correctly: GET (read), POST (create), PUT (replace), PATCH (update), DELETE (remove)
- Return appropriate HTTP status codes (200, 201, 204, 400, 401, 403, 404, 500)
- Use consistent error response format

## Naming Conventions
- Use kebab-case for URLs: `/api/user-profiles`
- Use camelCase for JSON fields: `{ "firstName": "John" }`
- Version your API: `/api/v1/users`

## Pagination
- Use query parameters: `?page=1&limit=20`
- Return pagination metadata in response

## Authentication
- Use Bearer tokens in Authorization header
- Implement proper CORS headers
```

Create `.claude/skills/rest-api-guide/resources/error-codes.md`:
```markdown
# Standard Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad Request - Invalid input |
| 401 | Unauthorized - Missing/invalid auth |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource already exists |
| 422 | Unprocessable Entity - Validation failed |
| 429 | Too Many Requests - Rate limited |
| 500 | Internal Server Error |
```

### test-convention
Create `.claude/skills/test-convention/SKILL.md`:
```markdown
# Test Conventions

## Structure
- Place tests next to source files or in __tests__/ directories
- Name test files: `*.test.ts` or `*.spec.ts`
- Use describe/it blocks for clear test organization

## Naming
- Describe behavior, not implementation: "should return user when valid ID provided"
- Use AAA pattern: Arrange, Act, Assert

## Coverage
- Aim for meaningful coverage, not 100%
- Focus on edge cases and error paths
- Mock external dependencies, not internal logic

## Best Practices
- Each test should be independent
- Avoid test interdependencies
- Use factory functions for test data
- Clean up side effects in afterEach/afterAll
```

### typescript-strict
Create `.claude/skills/typescript-strict/SKILL.md`:
```markdown
# TypeScript Strict Mode Guide

## Type Safety
- Enable strict mode in tsconfig.json
- Avoid `any` type - use `unknown` when type is uncertain
- Use explicit return types for public functions
- Prefer interfaces over type aliases for object shapes

## Patterns
- Use discriminated unions for state management
- Prefer `const` assertions for literal types
- Use template literal types for string patterns
- Leverage `satisfies` operator for type checking without widening

## Error Handling
- Use Result types instead of throwing
- Define custom error classes for domain errors
- Always handle Promise rejections

## Imports
- Use type-only imports: `import type { Foo } from "./foo.js"`
- Prefer named exports over default exports
```

### git-workflow
Create `.claude/skills/git-workflow/SKILL.md`:
```markdown
# Git Workflow Guide

## Commit Messages
- Use conventional commits: type(scope): description
- Types: feat, fix, docs, style, refactor, test, chore
- Keep subject line under 72 characters
- Use imperative mood: "add feature" not "added feature"

## Branching
- main: production-ready code
- feature/*: new features
- fix/*: bug fixes
- release/*: release preparation

## Pull Requests
- Keep PRs small and focused
- Include description of changes and testing done
- Request reviews from relevant team members
- Squash commits when merging
```

## Steps

1. Parse $ARGUMENTS to get the skill name
2. Validate: only `rest-api-guide`, `test-convention`, `typescript-strict`, `git-workflow` are valid. If invalid, report error listing available skills and stop.
3. Check if `.claude/skills/<name>/` already exists — if so, report it's already installed
4. Create the directory: `.claude/skills/<name>/`
5. Write the SKILL.md file with the content above
6. If the skill has resources (only `rest-api-guide`), create `resources/` dir and write files
7. Report success with file paths

$ARGUMENTS
