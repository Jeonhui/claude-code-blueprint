---
name: fix
description: "Fix an audit issue — apply targeted code changes, archive the issue, and update the ledger"
---

Fix a specific audit issue. The fixer agent reads the issue report, implements the fix following its role specification, archives the resolved issue, and updates the ledger.

Parse from $ARGUMENTS:
- `--id <id>` or `-i <id>` (optional): The issue ID to fix (e.g., `AUDIT-20260220-001`)
- `--next` or `-n` (optional): Automatically pick the highest-priority open issue from the backlog
- `--all` or `-a` (optional): Fix ALL open issues in priority order (critical → high → medium → low)
- `--by <agent>` or `-b <agent>` (optional): Fixer agent name. Defaults to `audit-fixer`.
- `--verify` or `-v` (optional): After fixing, run the `audit-qa` agent to verify the fix.
- `--dry-run` (optional): Show what would be fixed without making changes.

One of `--id`, `--next`, or `--all` is required. If none is provided, report: "Specify `--id <ID>`, `--next`, or `--all` to select issues."

Do NOT use MCP tools — use native file tools only (Read, Write, Edit, Glob, Grep, Bash).

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
   - Create default agent files (audit-logic.md, audit-style.md, audit-fixer.md, audit-qa.md) as defined in the "Default Agent Definitions" section of `/blueprint-audit:review` (`blueprint-audit/commands/review.md`).
   - Report: "Audit environment initialized."
3. If `.claude/audit/` already exists, skip initialization silently.

## Role Specification Protocol

**The fix command is a generic executor. ALL fix behavior is defined by the agent's role spec file.**

When executing a fix, you MUST:
1. Read the agent file from `.claude/agents/<agent>.md`
2. Parse the YAML frontmatter for metadata
3. Read the full markdown body as the agent's behavioral instructions
4. Follow the agent's "Fix Principles" / "Process" sections EXACTLY
5. The role spec is the single source of truth for:
   - **How to approach the fix** ("Fix Principles" or equivalent)
   - **What steps to follow** ("Process" section)
   - **Severity-aware behavior** (if defined)

## Issue Selection

Issues are stored inside consolidated review files (`REVIEW-*.md`) in `.claude/audit/backlog/`.

### With `--id <id>`:
1. Search all `.md` files in `.claude/audit/backlog/` for a section heading matching `## <id>:`
2. If not found, also search `.claude/audit/archive/` — if found there, report: "Issue `<id>` is already resolved." and stop.
3. If not found anywhere, report: "Issue `<id>` not found. Use `/blueprint-audit:status` to see open issues." and stop.
4. Read the issue section (from `## <id>:` until the next `---` or `## AUDIT-` heading)
5. Verify the issue's `status` is `TO_FIX`

### With `--next`:
1. Scan all `.md` files in `.claude/audit/backlog/`
2. Search for all issue sections (`## AUDIT-YYYYMMDD-NNN:`) with `status: TO_FIX`
3. Parse each issue's `priority` and the review file's `date`
4. Sort by priority (critical > high > medium > low), then by date (oldest first)
5. Pick the first issue
6. If no `TO_FIX` issues exist, report: "No open issues in backlog. Run `/blueprint-audit:review` to scan for issues." and stop.
7. Report which issue was auto-selected: "Auto-selected: <id> (<priority>) — <title>"

### With `--all`:
1. Scan all `.md` files in `.claude/audit/backlog/`
2. Collect ALL issue sections (`## AUDIT-YYYYMMDD-NNN:`) with `status: TO_FIX`
3. Sort by priority (critical > high > medium > low), then by date (oldest first)
4. If no `TO_FIX` issues exist, report: "No open issues in backlog. Run `/blueprint-audit:review` to scan for issues." and stop.
5. Report: "Fixing all N open issues in priority order..."
6. Process each issue sequentially through the Fix Execution steps (steps 1–4)
7. If a fix fails for any issue, log the failure and continue to the next issue — do NOT stop the entire batch

## Agent Name Resolution

When `--by <agent>` is provided, resolve the agent file using this fallback:

1. Try `.claude/agents/<agent>.md` (exact match — e.g., `--by audit-fixer` → `audit-fixer.md`)
2. If not found, try `.claude/agents/audit-<agent>.md` (auto-prefix — e.g., `--by fixer` → `audit-fixer.md`)
3. If neither exists, report error: "Agent '<agent>' not found. Available agents:" then list optimizer-type agents.

This allows users to use the short form (`-b fixer`) or the full form (`-b audit-fixer`) interchangeably.

## Fix Execution

1. **Read the issue:**
   - Parse the YAML frontmatter: id, status, reviewer, priority, target (file:line)
   - Read the Description and Improvement Suggestion sections
   - If status is not `TO_FIX`, report: "Issue `<id>` has status `<status>`, not TO_FIX. Skipping." and stop.

2. **Load the fixer agent:**
   - Resolve agent name using Agent Name Resolution (default: `audit-fixer`)
   - Read the resolved agent file

3. **Execute the fix following the role spec:**
   - Follow the agent's Process section step by step
   - Read the target file specified in the issue
   - Apply the fix using Edit tool with minimal changes
   - The agent's Fix Principles define constraints on how to make changes
   - If `--dry-run`, describe the planned changes without editing files

4. **On success — update status and log** (skip if `--dry-run`):

   a. **Update issue status in the review file:** In the issue section, change `- **status:** TO_FIX` → `- **status:** FIXED`. Also add:
      ```
      - **fixed_by:** <agent-name>
      - **fixed_date:** YYYY-MM-DD
      ```

   b. **Update the summary table** in the same review file: Change the issue's `Status` column from `TO_FIX` to `FIXED`.

   c. **Check if all issues in the review file are resolved:** If every issue in the file has `status: FIXED`, move the entire file to archive: `mv .claude/audit/backlog/REVIEW-*.md .claude/audit/archive/`

   d. **Append to LEDGER.md:** Add a new row to the table:
      ```
      | <id> | FIXED | <agent-name> | <target> | <verified: yes/no> | YYYY-MM-DD |
      ```

5. **Optional QA verification** (if `--verify` flag):
   - Load `.claude/agents/audit-qa.md`
   - Follow the QA agent's role spec to verify the fix
   - Update the "Verified" column in LEDGER.md to `yes` if verification passes

6. **Report:**

#### Single issue (`--id` or `--next`):
```
# Audit Fix Report

**Issue:** <id>
**Priority:** <priority>
**Status:** FIXED
**Fixed By:** <agent-name>
**Target:** <file:line>
**Verified:** <yes/no/skipped>
**Date:** YYYY-MM-DD

## Changes Made
- <summary of what was changed>

## Files Modified
- <list of files that were edited>

> Run `/blueprint-audit:status` to see updated dashboard.
> Run `/blueprint-audit:fix --next` to fix the next issue.
```

#### Batch (`--all`):
```
# Audit Batch Fix Report

**Date:** YYYY-MM-DD
**Fixed By:** <agent-name>
**Total Attempted:** N
**Succeeded:** N
**Failed:** N

## Results
| ID | Priority | Target | Status | Summary |
|:---|:---|:---|:---|:---|
| AUDIT-YYYYMMDD-NNN | high | file:line | FIXED | Short summary of change |
| AUDIT-YYYYMMDD-NNN | medium | file:line | FAILED | Reason for failure |

## Files Modified
- <deduplicated list of all files edited across all fixes>

> Run `/blueprint-audit:status` to see updated dashboard.
```

If `--dry-run`, prefix with `**[DRY RUN]** No changes were made.`

$ARGUMENTS
