---
name: validate-plan
description: 'Use when validating an implementation plan for structural consistency before developer review. Checks AC coverage, task traceability, file justification, test coverage, and contradiction detection. Lightweight single-pass lint, not a full review. Triggers on: "validate plan", "check plan", "lint plan", or automatically after plan generation.'
---

# Validate Plan

Lightweight structural lint for implementation plans. Catches gaps and
inconsistencies before the developer sees the plan.

This is a fast, deterministic consistency check — think linter, not code review.

All artifacts are read from and written to `.github/context/task-specific/{TICKET_ID}/`.

## When to Use

- After generating a plan in the `plan-implementation` agent
- Standalone: developer asks to validate an existing plan

## Input

| Parameter | Required     | Description                                      |
| --------- | ------------ | ------------------------------------------------ |
| TICKET_ID | One of these | Ticket ID — plan path derived automatically      |
| PLAN_PATH | One of these | Direct path to plan markdown file                |

**Path resolution order:**

1. If PLAN_PATH provided, use it directly
2. If TICKET_ID provided, look in: `.github/context/task-specific/{TICKET_ID}/implementation-plan.md`

**Context resolution** (for AC source):

All artifacts live in `.github/context/task-specific/{TICKET_ID}/`:

1. Look for problem statement: `problem-statement.md`
2. Look for ticket context file: `jira-{TICKET_ID}.md`
3. Extract ACs from the plan's own Requirements section as fallback

## Procedure

### Step 1: Locate and Read Artifacts

Read the plan file. If it doesn't exist, stop with error.

Read the requirements source (problem statement, ticket context, or plan's own
Requirements section). If no external requirements source exists, use the plan's
internal requirements — note this limitation in the output.

### Step 2: Extract Structured Elements

Parse the plan to identify:

- **ACs**: Acceptance criteria (patterns: `AC-001`, `AC-1`, `- AC:`, Given/When/Then blocks)
- **Tasks**: Implementation tasks (patterns: `TASK-001`, `T001`, numbered steps with file paths)
- **Files**: Affected files table or inline file references (backtick-quoted paths)
- **Constraints**: Requirements and constraints (patterns: `REQ-001`, `CON-001`, `NFR-001`)
- **Tests**: Testing strategy entries (test type + description pairs)

### Step 3: Run Validation Passes

Execute these 5 checks in order:

#### Check 1: AC Coverage

For each acceptance criterion found:
- **PASS**: Every AC maps to at least one task
- **WARN**: AC is only partially addressed
- **FAIL**: AC has no corresponding task at all

#### Check 2: Task Traceability

For each task:
- **PASS**: All tasks reference files
- **WARN**: Task describes a code change but has no file path
- **SKIP**: Task is non-code (e.g., "review with team", "update documentation")

#### Check 3: File Justification

For each file listed in the Affected Files section:
- **PASS**: All files are referenced by tasks
- **WARN**: File listed but no task touches it (orphan file)

#### Check 4: Test Coverage

For each functional acceptance criterion:
- **PASS**: Every functional AC has a test
- **WARN**: AC exists but no test covers it
- **SKIP**: AC is non-functional (process, documentation)

#### Check 5: Contradiction Check

Scan for logical conflicts:
- Task says "create file X" but another task says "modify file X"
- Task contradicts a stated constraint
- Duplicate tasks doing the same thing
- **PASS**: No contradictions found
- **FAIL**: Contradiction detected

### Step 4: Generate Report

```markdown
## Plan Validation: {TICKET-ID}

**Plan:** `{PLAN_PATH}`
**Requirements source:** `{source file or "plan internal"}`

| #   | Check               | Status           | Detail                                                  |
| --- | ------------------- | ---------------- | ------------------------------------------------------- |
| 1   | AC Coverage         | {PASS/WARN/FAIL} | {count mapped}/{count total} ACs mapped to tasks        |
| 2   | Task Traceability   | {PASS/WARN/SKIP} | {detail or "All N tasks reference files"}               |
| 3   | File Justification  | {PASS/WARN}      | {detail or "All N files referenced by tasks"}           |
| 4   | Test Coverage       | {PASS/WARN/SKIP} | {count covered}/{count total} functional ACs have tests |
| 5   | Contradiction Check | {PASS/FAIL}      | {detail or "No contradictions found"}                   |

**Verdict:** {PASS | WARN (N warnings) | FAIL (N failures, M warnings)}

### Issues Found

| Issue | Check         | Severity | Detail                                                     |
| ----- | ------------- | -------- | ---------------------------------------------------------- |
| 1     | AC Coverage   | FAIL     | AC-003 "..." has no corresponding task                     |

{If PASS, write: "No issues found."}
```

### Step 5: Return Verdict

- **PASS**: All checks passed. Plan is structurally sound.
- **WARN**: Warnings found but no failures. Note warnings for developer awareness.
- **FAIL**: At least one failure. Fix issues before presenting plan to developer.

## Error Handling

| Error                 | Action                                                                       |
| --------------------- | ---------------------------------------------------------------------------- |
| Plan file not found   | Stop with "No plan found at {path}. Provide PLAN_PATH or TICKET_ID."         |
| No ACs found anywhere | Skip Check 1, note "No acceptance criteria found — cannot validate coverage" |
| No tasks found        | FAIL Check 2, note "Plan has no identifiable tasks"                          |

## DO

- Accept plans in multiple formats (template or free-form)
- Use fuzzy matching for AC-to-task mapping (keyword overlap, not just ID matching)
- Report the specific line or section where each issue was found
- Keep the output concise — one table row per issue, not paragraphs

## DO NOT

- Modify the plan file
- Spawn sub-agents or make external API calls
- Block on warnings — only FAIL verdicts should prevent progression
- Validate subjective quality (this skill checks structure, not merit)
