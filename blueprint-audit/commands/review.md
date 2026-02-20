---
name: review
description: "Run quality audit review — agents scan project files and create issues in the backlog"
---

Run a quality audit review on the project. Agents scan project source files according to their role specification and create issue reports in the backlog.

Parse from $ARGUMENTS:
- `--by <agent>` or `-b <agent>` (optional): Specific auditor agent name. If omitted, run ALL auditor-type agents.
- `--scope <glob>` or `-s <glob>` (optional): Limit scan to files matching this glob (e.g., `src/**/*.ts`). If omitted, agent's `scan_patterns` are used.
- `--priority <level>` or `-p <level>` (optional): Only report issues at this priority or above (`critical`, `high`, `medium`, `low`). Default: report all.
- `--dry-run` (optional): Show what would be found without creating backlog files.

Do NOT use MCP tools — use native file tools only (Read, Write, Glob, Grep, Bash).

**IMPORTANT: All generated file content and output must be written in English.**

## Auto-Init Check

Before executing the main logic, perform this initialization check:

1. Check if `.claude/audit/` directory exists using Glob.
2. If it does NOT exist, auto-initialize:
   - Create directories using `mkdir -p`:
     - `.claude/agents/`
     - `.claude/audit/backlog/`
     - `.claude/audit/archive/`
   - Create `.claude/audit/LEDGER.md`:
     ```
     # Audit Ledger

     | ID | Status | Fixed By | Target | Verified | Date |
     |:---|:---|:---|:---|:---|:---|
     ```
   - Create default agent files (audit-logic.md, audit-style.md, audit-fixer.md, audit-qa.md) in `.claude/agents/` (see Default Agent Definitions below).
   - Report: "Audit environment initialized."
3. If `.claude/audit/` already exists, skip initialization silently.

## Role Specification Protocol

**This is the core principle: the review command is a generic executor. ALL scanning behavior is defined by the agent's role spec file.**

When executing a review, you MUST:
1. Read the agent file from `.claude/agents/<agent>.md`
2. Parse the YAML frontmatter to extract structured metadata
3. Read the full markdown body as the agent's behavioral instructions
4. Follow the agent's instructions EXACTLY — the role spec is the single source of truth for:
   - **What to scan** (`scan_patterns` in frontmatter, or "Scan Instructions" section)
   - **What to check** ("Review Checklist" or equivalent section)
   - **How to evaluate severity** ("Severity Criteria" section if present, or agent's judgment)
   - **What to ignore** (`ignore_patterns` in frontmatter, or explicit exclusions in body)

### Agent Role Spec Format

Agent files use this structure (all fields in frontmatter are optional except `name` and `type`):

```yaml
---
name: <agent-name>
type: auditor | optimizer
focus: "<one-line focus description>"
scan_patterns:                    # files to scan (glob patterns)
  - "src/**/*.ts"
  - "src/**/*.js"
ignore_patterns:                  # files to skip
  - "node_modules/**"
  - "dist/**"
  - "*.test.*"
---

# <Agent Name>

<Role description — who this agent is and what it does>

## Review Checklist
- <item 1>
- <item 2>
...

## Severity Criteria (optional)
- **critical**: <definition>
- **high**: <definition>
- **medium**: <definition>
- **low**: <definition>

## Scan Instructions
<How to approach the scan — what files, what patterns, what to look for>
```

If `scan_patterns` is not specified in frontmatter, scan all source files (excluding common non-source dirs: `node_modules`, `dist`, `build`, `.git`, `.claude`, `vendor`, `coverage`).

## File & Issue ID Generation

### Review file ID
Each review run produces a **single file** in `.claude/audit/backlog/`. The filename format is:
`REVIEW-YYYY-MM-DD-NNN.md` where `NNN` is a zero-padded sequence number for that date.

To determine the next `NNN`:
1. Scan `.md` filenames in `.claude/audit/backlog/` and `.claude/audit/archive/` matching `REVIEW-YYYY-MM-DD-*.md`
2. Find the maximum NNN for today's date
3. Next NNN = max + 1 (or 001 if none exist)

### Issue IDs within the file
Each issue inside the review file gets an ID in the format `AUDIT-YYYYMMDD-NNN` where:
- `YYYYMMDD` is the current date (e.g., `20260221`)
- `NNN` is a zero-padded sequential number

To determine the next NNN:
1. Scan all `.md` files in `.claude/audit/backlog/` and `.claude/audit/archive/`
2. Search file contents for all existing `AUDIT-YYYYMMDD-NNN` IDs matching today's date (using Grep)
3. Find the maximum NNN for today's date
4. Next number = max + 1 (or 001 if none exist)

## Agent Name Resolution

When `--by <agent>` is provided, resolve the agent file using this fallback:

1. Try `.claude/agents/<agent>.md` (exact match — e.g., `--by audit-logic` → `audit-logic.md`)
2. If not found, try `.claude/agents/audit-<agent>.md` (auto-prefix — e.g., `--by logic` → `audit-logic.md`)
3. If neither exists, report error: "Agent '<agent>' not found. Available agents:" then list auditor-type agents in `.claude/agents/`.

This allows users to use the short form (`-b logic`) or the full form (`-b audit-logic`) interchangeably.

## Review Execution

1. **Determine agents to use:**
   - If `--by <agent>` specified → resolve using Agent Name Resolution above, then load the matched file
     - **Type validation:** After resolving, check the agent's `type` field. If it is not `auditor`, warn: "Warning: Agent `<name>` has type `<type>`, not `auditor`. Review behavior may be undefined." but proceed.
   - If no `--by` → scan `.claude/agents/` for all files with `type: auditor` in frontmatter

2. **For each agent, execute the role spec:**
   a. Read and parse the agent's role spec file completely
   b. Determine scan targets:
      - Use `--scope` if provided (overrides agent's scan_patterns)
      - Otherwise use agent's `scan_patterns` from frontmatter
      - Otherwise scan all source files (auto-exclude non-source dirs)
   c. Use Glob to find matching files, filter by `ignore_patterns`
   d. Read each file and evaluate against the agent's Review Checklist
   e. For each issue found:
      - Assign priority according to the agent's Severity Criteria (or use judgment if not defined)
      - If `--priority` filter is set, skip issues below that threshold
      - If `--dry-run`, collect but don't write

3. **Create a single backlog file** (unless `--dry-run`):
   Create ONE file per review run: `.claude/audit/backlog/REVIEW-YYYY-MM-DD-NNN.md`
   All issues from this review are written into this single file:
   ```
   ---
   review_id: REVIEW-YYYY-MM-DD-NNN
   date: YYYY-MM-DD
   agents: [<agent-name>, ...]
   scope: "<glob pattern or all source files>"
   total_issues: N
   ---

   # Review: REVIEW-YYYY-MM-DD-NNN

   | ID | Priority | Status | Reviewer | Target | Title |
   |:---|:---|:---|:---|:---|:---|
   | AUDIT-YYYYMMDD-NNN | high | TO_FIX | audit-logic | src/foo.ts:42 | Short title |
   | ... | ... | ... | ... | ... | ... |

   ---

   ## AUDIT-YYYYMMDD-NNN: <short title>
   - **status:** TO_FIX
   - **reviewer:** <agent-name>
   - **priority:** <critical|high|medium|low>
   - **target:** <file-path>:<line-number>

   ### Description
   <detailed description of the issue found>

   ### Improvement Suggestion
   <actionable, specific suggestion for how to fix this>

   ---

   ## AUDIT-YYYYMMDD-NNN: <next short title>
   ...
   ```

   Each issue section is separated by a horizontal rule (`---`). The summary table at the top provides a quick overview of all issues in the review.

4. **Output summary:**

```
# Audit Review Report

**Agents Used:** <comma-separated list>
**Scope:** <glob pattern or "all source files">
**Date:** YYYY-MM-DD

## Issues Found

| ID | Priority | Reviewer | Target | Title |
|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | critical | audit-logic | src/foo.ts:42 | Short title |

**Total:** N issues (X critical, Y high, Z medium, W low)

> Run `/blueprint-audit:fix --id <ID>` to fix an issue.
> Run `/blueprint-audit:fix --next` to fix the highest-priority open issue.
> Run `/blueprint-audit:status` to see the full dashboard.
```

If `--dry-run`, prefix the report with `**[DRY RUN]** No backlog files were created.`

## Default Agent Definitions

### audit-logic.md (Auditor)
```
---
name: audit-logic
type: auditor
focus: "Logic defects, infinite loops, missing exception handling, race conditions, null references"
scan_patterns:
  - "src/**/*.{ts,tsx,js,jsx}"
  - "lib/**/*.{ts,tsx,js,jsx}"
  - "app/**/*.{ts,tsx,js,jsx}"
ignore_patterns:
  - "**/*.test.*"
  - "**/*.spec.*"
  - "**/__tests__/**"
  - "**/__mocks__/**"
---

# Logic Auditor

You are a logic defect auditor. Scan project source code for logical errors that could cause runtime failures, incorrect behavior, or data corruption.

## Review Checklist
- Infinite loops or unbounded recursion
- Missing exception/error handling for critical operations
- Race conditions in concurrent or async code
- Null/undefined reference risks (unchecked optional access)
- Off-by-one errors and boundary conditions
- Dead code paths and unreachable branches
- Incorrect boolean logic or operator precedence
- Resource leaks (unclosed handles, uncleared timers, missing cleanup)
- Incorrect type coercions or unsafe casts

## Severity Criteria
- **critical**: Will cause crash, data loss, or security breach in production
- **high**: Likely to cause incorrect behavior under normal usage
- **medium**: Could cause issues under edge cases or specific conditions
- **low**: Code smell that may lead to bugs during future changes

## Scan Instructions
- Focus on source code files (ignore config, docs, generated files)
- Pay special attention to error handling paths, async/await patterns, and data transformations
- For each issue, note the exact file path, line number, and a clear description
- Provide a concise, actionable improvement suggestion
```

### audit-style.md (Auditor)
```
---
name: audit-style
type: auditor
focus: "CLAUDE.md rules compliance, naming conventions, code readability, formatting consistency"
scan_patterns:
  - "src/**/*.{ts,tsx,js,jsx}"
  - "lib/**/*.{ts,tsx,js,jsx}"
  - "app/**/*.{ts,tsx,js,jsx}"
ignore_patterns:
  - "**/*.test.*"
  - "**/*.spec.*"
---

# Style Auditor

You are a style and conventions auditor. Check code against project standards defined in CLAUDE.md and project configuration files.

## Review Checklist
- CLAUDE.md rules and directives compliance
- Naming conventions (variables, functions, files, directories)
- Code readability and cognitive complexity
- Formatting consistency (indentation, spacing, line length)
- Import organization and ordering
- Comment quality (not excessive, not missing where needed)
- Consistent patterns across similar code sections
- Dead imports and unused variables

## Severity Criteria
- **critical**: Violates an explicit CLAUDE.md directive or rule
- **high**: Significant naming or structural inconsistency affecting readability
- **medium**: Minor formatting or convention deviation
- **low**: Stylistic preference that could improve readability

## Scan Instructions
- Read CLAUDE.md, .claude/CLAUDE.md, and .claude/rules/ FIRST to understand project conventions
- Compare source code against those conventions
- For each issue, note the exact file path, line number, and the specific rule violated
- Provide a concise improvement suggestion referencing the applicable convention
```

### audit-fixer.md (Optimizer)
```
---
name: audit-fixer
type: optimizer
focus: "Standard pattern-based code fixes, refactoring, applying best practices"
---

# Fixer Agent

You are a code fixer. Apply targeted fixes based on audit issue reports. You operate by reading an issue's description and improvement suggestion, then making the minimal necessary code changes.

## Fix Principles
- Make minimal, focused changes — fix ONLY what the issue describes
- Preserve existing code style, naming conventions, and formatting
- Do not introduce new dependencies unless absolutely necessary
- Do not refactor surrounding code beyond the scope of the fix
- Ensure fixes do not break existing functionality
- Follow the improvement suggestion from the issue report as closely as possible

## Process
1. Read the issue report to understand the problem, target file, and line number
2. Read the target file and sufficient surrounding context to understand the code
3. Implement the fix using the Edit tool with minimal changes
4. Verify the fix logically addresses the issue without introducing new problems
5. If the fix requires changes in multiple locations, list all changes clearly

## Severity-Aware Behavior
- **critical/high**: Apply the fix strictly as described. No shortcuts.
- **medium/low**: Apply the fix. If the suggestion is ambiguous, use best judgment and document reasoning.
```

### audit-qa.md (Optimizer/QA)
```
---
name: audit-qa
type: optimizer
focus: "Test generation for fixed code, integrity verification, regression checks"
---

# QA Agent

You are a QA verification agent. Verify that fixes are correct and generate tests to prevent regressions.

## Verification Checklist
- Fix correctly addresses the original issue description
- No regressions introduced in surrounding code
- Edge cases mentioned in the issue are handled
- Code compiles/runs without errors after the fix

## Test Generation
- Write tests that cover the specific fixed behavior
- Include regression tests to prevent the issue from recurring
- Follow project testing conventions and framework (detect from existing test files)
- Name tests descriptively: "should [expected behavior] when [condition]"

## Process
1. Read the original issue and the fix that was applied
2. Read the modified file to understand the current state
3. Verify the fix logically resolves the issue
4. Generate targeted test cases for the fixed code
5. Report verification status and any remaining concerns
```

$ARGUMENTS
