<!--
GENERATION RULES -- read all of this before writing, then DELETE this comment block.

QUALITY BAR (superpowers:writing-plans): every step must be executable by an
engineer who has ZERO context for this codebase and cannot ask questions.
That means:

- COMPLETE, copy-pasteable code in every step -- never a description of code.
  "Add validation logic to the service" is NOT a plan step. The actual method
  body is.
- No placeholders. `// TODO`, `// implement here`, `...` and "similar to the
  step above" are all failures.
- Exact file paths from the repository root, marked (create) or (modify).
- Exact commands with expected output: the RED command must state the exact
  expected failure (assertion message / error type), the GREEN command must
  state the expected pass.
- Investigate before you write: read the real codebase first. Confirm build
  and test commands, module layout, naming conventions, existing base classes
  and utilities. Record what every step relies on under "Key facts". A plan
  written without reading the code is fiction.
- Every unchecked task ID in tasks.md maps to exactly one step below. No
  unmapped tasks, no steps without a task ID.

SIZE EXPECTATION: a plan meeting this bar is typically 5-10x larger than a
summary-level plan. If your draft has steps under ~30 lines, they are almost
certainly descriptions, not implementations. Rewrite them before saving.
-->

# Implementation Plan -- [change-name]

**Tracks:** `tasks.md` -- every unchecked task ID appears in exactly one step.
**Execution:** `openspec-superpowers run [change-name]` (strict TDD: RED -> GREEN -> refactor per step).

## Context

[2-3 sentences: what this change delivers and why. Link to proposal.md / design.md rather than repeating them.]

## Key facts for implementers

[Everything you discovered while reading the codebase that the steps depend on.
An executor reads this once instead of rediscovering it per step. Include at least:]

- Build / test / lint commands: [exact commands, e.g. `mvn -pl module-x test`, `pnpm --filter web test`]
- Module layout and where new code goes: [paths]
- Conventions that constrain this change: [naming, base classes, DTO/DO/VO patterns, route prefixes, error handling, etc.]
- Gotchas: [missing dependencies to add first, test-infra quirks (H2 vs prod DB dialect), config that must exist]

## Steps

<!-- Repeat this block per step. Keep steps small: one focused test each. -->

### Step 1: [imperative title] `[OpenSpec task: N.M]`

- [ ] Complete

**Files:**

- `exact/path/from/repo/root/FooTest.java` (create)
- `exact/path/from/repo/root/Foo.java` (create | modify)

**Test (RED):**

```[lang]
[COMPLETE test code -- imports, class, setup, one focused assertion. Copy-pasteable as-is.]
```

Run: `[exact command scoped to this test]`
Expected failure: `[exact expected error -- e.g. "NoSuchBeanDefinitionException: FooService" or "expected 429 but was 200"]`

**Implementation (GREEN):**

```[lang]
[COMPLETE implementation code -- the minimal change that makes the test pass. Copy-pasteable as-is.]
```

Run: `[same command]` -> expect pass.

**Verify:** `[project-wide check command(s) -- lint / typecheck / affected-module tests]`

**Commit:** `git add -A && git commit -m "[type]: [what this step did]"`

---

### Step 2: ...

## Final verification

- [ ] All steps above checked off
- [ ] All task IDs in `tasks.md` checked off
- [ ] Full suite clean: `[exact commands and what clean output looks like]`
- [ ] `openspec status --change [change-name] --json` shows all artifacts complete
