---
name: status
description: "Display quality audit dashboard — issues overview, priority breakdown, and agent registry"
---

Display the current audit quality dashboard for this project.

Parse from $ARGUMENTS:
- `--brief` (optional): Show compact one-line summary only.
- `--by <agent>` or `-b <agent>` (optional): Filter issues by reviewer/fixer agent name.
- `--priority <level>` or `-p <level>` (optional): Filter to show only issues at this priority or above.

Do NOT use MCP tools — use native file tools only (Read, Glob, Grep).

**IMPORTANT: All output must be written in English.**

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
   - Create default agent files (audit-logic.md, audit-style.md, audit-fixer.md, audit-qa.md) as defined in the "Default Agent Definitions" section of `/blueprint-audit:review` (`blueprint-audit/commands/review.md`).
   - Report: "Audit environment initialized."
3. If `.claude/audit/` already exists, skip initialization silently.

## Agent Name Resolution

When `--by <agent>` is provided to filter, resolve the agent name:

1. If the name matches an agent file exactly (e.g., `audit-logic`), use it
2. If not, try with `audit-` prefix (e.g., `logic` → `audit-logic`)
3. Filter issues where `reviewer` or `fixed_by` matches the resolved name

This allows users to use the short form (`-b logic`) or the full form (`-b audit-logic`) interchangeably.

## Dashboard Generation

### 1. Scan backlog/ for open issues
- Read all `REVIEW-*.md` files in `.claude/audit/backlog/`
- Parse each issue section (`## AUDIT-YYYYMMDD-NNN:`) and extract fields from the metadata lines:
  - `- **status:** TO_FIX` → status
  - `- **reviewer:** audit-logic` → reviewer
  - `- **priority:** high` → priority
  - `- **target:** src/foo.ts:42` → target
- The issue title comes from the heading: `## AUDIT-YYYYMMDD-NNN: <title>`
- The `Created` date comes from the review file's YAML frontmatter `date:` field
- Filter to only `status: TO_FIX` issues (skip `FIXED` issues still in backlog files)
- Apply filters: `--by` filters by reviewer, `--priority` filters by minimum priority level
- Group by priority (critical → high → medium → low)

### 2. Scan archive/ for resolved issues
- Read all `REVIEW-*.md` files in `.claude/audit/archive/`
- Count total issue sections with `- **status:** FIXED`
- Also count `FIXED` issues still in backlog files (partially resolved review files)
- If `--by` is set, count only those matching the agent name

### 3. Read LEDGER.md
- Parse the table rows to extract recent activity
- Get the last 10 entries for the activity log

### 4. List registered agents
- Read all `.md` files in `.claude/agents/`
- Parse frontmatter: name, type, focus

### 5. Output

You MUST use the exact output formats specified below. Follow the table structures, headings, and spacing precisely.

**CRITICAL rendering rule:** ALL data sections (Summary, each Priority group, Recent Activity, Registered Agents) MUST be output as **markdown tables using pipe (`|`) syntax**. NEVER output issues as key-value pairs, bullet lists, or plain text. Every priority group with issues MUST use a table — no exceptions regardless of how many rows exist.

#### Brief mode (`--brief`):
```
Audit: N open (X critical, Y high) | N resolved | N% resolution rate
```

#### Full mode (default):
```
# Audit Quality Dashboard

**Date:** YYYY-MM-DD

## Summary
| Metric | Value |
|:---|:---|
| Open Issues | N |
| Resolved Issues | N |
| Total Issues | N |
| Resolution Rate | N% |

## Open Issues by Priority

### Critical (N)
| ID | Reviewer | Target | Created | Title |
|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | audit-logic | src/foo.ts:42 | YYYY-MM-DD | Short title |

### High (N)
| ID | Reviewer | Target | Created | Title |
|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | audit-logic | src/bar.ts:10 | YYYY-MM-DD | Short title |

### Medium (N)
| ID | Reviewer | Target | Created | Title |
|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | audit-style | src/baz.ts:5 | YYYY-MM-DD | Short title |

### Low (N)
| ID | Reviewer | Target | Created | Title |
|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | audit-style | src/qux.ts:1 | YYYY-MM-DD | Short title |

## Recent Activity
| ID | Status | Fixed By | Target | Verified | Date |
|:---|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | FIXED | audit-fixer | src/foo.ts:42 | yes | YYYY-MM-DD |

## Registered Agents
| Name | Type | Focus |
|:---|:---|:---|
| audit-logic | auditor | Logic defects, ... |
| audit-style | auditor | CLAUDE.md rules, ... |
| audit-fixer | optimizer | Pattern-based fixes, ... |
| audit-qa | optimizer | Test generation, ... |

---
> `/blueprint-audit:review` — scan for new issues
> `/blueprint-audit:fix --next` — fix highest-priority issue
> `/blueprint-audit:review-instruction` — print review role specs and commands
> `/blueprint-audit:fix-instruction` — print fix role specs and commands
```

**Display rules:**
- If a priority group has 0 issues, show `No <priority> issues.` instead of an empty table
- Recent Activity shows the last 10 rows from LEDGER.md. If empty, show `No activity recorded yet.`
- Resolution Rate = `Resolved / Total * 100`, rounded to nearest integer. If Total is 0, show `0%`

#### Empty state (no open AND no resolved issues):
```
# Audit Quality Dashboard

**Date:** YYYY-MM-DD

No audit activity yet. Run `/blueprint-audit:review` to start scanning.

## Registered Agents
| Name | Type | Focus |
|:---|:---|:---|
| ... | ... | ... |
```

$ARGUMENTS
