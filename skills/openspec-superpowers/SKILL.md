---
name: openspec-superpowers
description: >
  Bridge between OpenSpec's SDD-TDD workflow and Superpowers skills.
  Provides plan, run, verify, and archive modes to drive spec-driven,
  test-driven development from proposal through to archived change.
---

# OpenSpec ↔ Superpowers Bridge

This skill orchestrates the SDD-TDD workflow by connecting OpenSpec's artifact pipeline to Superpowers' execution skills. It supports four modes: **plan**, **run**, **verify**, and **archive**.

---

## Mode: `plan`

**Purpose:** Create `implementation-plan.md` from the sdd-tdd schema template.

**Skill dependency:** `superpowers:writing-plans`

### Instructions

1. Read the current change's `tasks.md` via `openspec instructions implementation-plan --change <change>`.
2. Activate the `superpowers:writing-plans` skill.
3. Generate `implementation-plan.md` in the change directory, following the schema template structure.
4. **Map every unchecked OpenSpec task ID** from `tasks.md` to at least one plan step. If any task is unmapped, the plan is incomplete — do not proceed.
5. Each plan step must include:
   - The OpenSpec task ID(s) it satisfies
   - Exact file paths to create or modify
   - One focused failing test
   - The RED command and expected failure description
   - The minimal implementation to make the test pass
   - The GREEN command to confirm the test passes
   - Project verification commands
6. **Do NOT check off `tasks.md` checkboxes during planning.** Task completion happens only in `run` mode.

### Completion criteria
- `implementation-plan.md` exists in the change directory
- Every unchecked task in `tasks.md` is traced to at least one plan step
- `openspec status --change <change> --json` shows `implementation-plan` as complete

---

## Mode: `run`

**Purpose:** Execute the implementation plan using test-driven development.

**Skill dependency:** `superpowers:test-driven-development`

### Instructions

1. Read `implementation-plan.md` for the current change.
2. Activate the `superpowers:test-driven-development` skill.
3. For each plan step, in order:
   a. **Write the focused failing test** from the plan step.
   b. **RED:** Run the test command. Confirm it fails with the expected failure. If it passes unexpectedly, investigate — the test may not be focused enough.
   c. **Implement:** Make the minimal code change described in the plan step.
   d. **GREEN:** Run the test command. Confirm it passes. If it fails, fix the implementation (not the test) until green.
   e. **Refactor** while green — clean up without changing behavior, keep tests passing.
   f. **Mark the plan step complete** (check off in `implementation-plan.md`).
   g. **Mark the corresponding OpenSpec task(s) complete** (check off in `tasks.md`).
   h. **Run project verification** (lint, typecheck, full test suite). Fix any failures before moving to the next step.
4. After all steps are complete, run `openspec status --change <change> --json` to confirm all artifacts and tasks are done.
5. **Stop before archive.** Do not archive automatically — that is an explicit user action.

### Completion criteria
- All plan steps checked off in `implementation-plan.md`
- All OpenSpec tasks checked off in `tasks.md`
- Project verification passes (lint, typecheck, tests)
- `openspec status --change <change> --json` shows all artifacts complete

---

## Mode: `verify`

**Purpose:** Resolve any verification failures before reporting readiness.

**Skill dependency:** `superpowers:verification-before-completion`

### Instructions

1. Activate the `superpowers:verification-before-completion` skill.
2. Run all project checks:
   - Lint (`npm run lint`, or project-appropriate equivalent)
   - Type checking (`npm run typecheck`, or project-appropriate equivalent)
   - Full test suite (`npm test`, or project-appropriate equivalent)
3. Run `openspec status --change <change> --json` and check for incomplete artifacts or tasks.
4. For each failure:
   a. Diagnose the root cause.
   b. Fix it (code fix, missing test, incomplete task).
   c. Re-run verification to confirm the fix.
5. **Do not report readiness until all checks pass.** If a failure is unresolvable, report the blocker with context.

### Completion criteria
- All project checks pass
- All OpenSpec artifacts are complete
- All tasks in `tasks.md` are checked
- No unresolved blockers reported

---

## Mode: `archive`

**Purpose:** Archive the completed change using OpenSpec. This is always an **explicit user action**.

### Instructions

1. Confirm all prerequisites:
   - `verify` mode has been run and passed (all checks clean)
   - `openspec status --change <change> --json` shows everything complete
2. Run `openspec sync --change <change>` to sync specs back to the canonical store.
3. Run `openspec archive --change <change>` to archive the change.
4. Confirm archive succeeded: the change directory should be moved to `openspec/changes/archive/`.

### Completion criteria
- Change is archived in `openspec/changes/archive/`
- Specs are synced to the canonical store
- No remaining open items

---

## Usage

```text
# Plan the implementation (from tasks.md → implementation-plan.md)
openspec-superpowers plan <change>

# Execute with TDD (RED → GREEN → refactor for each step)
openspec-superpowers run <change>

# Verify everything is clean before archiving
openspec-superpowers verify <change>

# Archive (explicit user action)
openspec-superpowers archive <change>
```

## Typical Full Loop

```text
/opsx:propose <change>          # OpenSpec creates proposal → specs → design → tasks
openspec-superpowers plan <change>   # Plan: tasks → implementation-plan
openspec-superpowers run <change>    # Run: TDD execution of plan steps
openspec-superpowers verify <change> # Verify: resolve any failures
openspec-superpowers archive <change> # Archive: sync & archive (user action)
```
