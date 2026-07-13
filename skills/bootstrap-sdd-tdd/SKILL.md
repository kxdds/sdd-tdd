---
name: bootstrap-sdd-tdd
description: Use when a project needs the OpenSpec + Superpowers sdd-tdd (spec-to-plan-to-TDD) workflow checked, installed, and configured -- or cleanly removed -- in any coding agent (Codex, Claude Code, Cursor, Antigravity, OpenCode, Gemini CLI, Copilot CLI, or others).
---

# Bootstrap SDD-TDD (universal)

Build, validate, or remove the project-local OpenSpec + Superpowers workflow from any coding agent. Treat setup as idempotent: preserve valid existing work and repair only missing or incompatible framework files.

## Commands

- `bootstrap-sdd-tdd setup` (default when no command is given): check dependencies, install with user consent, scaffold the sdd-tdd schema, and place the `openspec-superpowers` bridge skill in the host tool's skills directory.
- `bootstrap-sdd-tdd clean`: remove everything this skill generated and restore OpenSpec to its default orchestration. Never uninstalls the OpenSpec CLI or the Superpowers plugin, and never deletes this bootstrap skill itself.

## Step 0: Detect the host tool

Determine which agent tool you are running inside. Use, in order:

1. Your own knowledge of your identity (you usually know which product you are).
2. Directory markers in the project and environment: `.codex/`, `.claude/`, `.cursor/`, `.opencode/`, `.agents/`, `.gemini/`.
3. If still ambiguous, ask the user one concise question.

Then resolve two lookups from the tables below: (a) where the bridge skill goes, (b) how Superpowers is installed.

### Table A -- bridge skill target directory

Rule: if the host tool has its own project-local skills directory, use it; otherwise use the cross-tool generic `.agents/skills/`.

| Host tool | Bridge skill location | Extra packaging |
| --- | --- | --- |
| Codex | `.codex/skills/openspec-superpowers/SKILL.md` | Also write `agents/openai.yaml` next to it (interface metadata) |
| Claude Code | `.claude/skills/openspec-superpowers/SKILL.md` | none |
| Cursor | `.cursor/skills/openspec-superpowers/SKILL.md` | none |
| Antigravity (IDE/CLI) | `.agents/skills/openspec-superpowers/SKILL.md` | Ensure root `AGENTS.md` mentions the skill so it is discoverable |
| OpenCode | `.opencode/skills/openspec-superpowers/SKILL.md` | none |
| Gemini CLI | `.gemini/skills/openspec-superpowers/SKILL.md` | none |
| Unknown / other | `.agents/skills/openspec-superpowers/SKILL.md` | Ensure root `AGENTS.md` mentions the skill |

### Table B -- how to get Superpowers

Rule: if the host tool has a plugin/extension/marketplace system, install Superpowers through it; if it has none, fall back to vendoring the official repo into the project.

| Host tool | Install surface |
| --- | --- |
| Claude Code | `/plugin install superpowers@claude-plugins-official` (or `/plugin marketplace add obra/superpowers-marketplace` then `/plugin install superpowers@superpowers-marketplace`) |
| Cursor | `/add-plugin superpowers` in Agent chat, or search "superpowers" in the plugin marketplace |
| Antigravity | `agy plugin install https://github.com/obra/superpowers` |
| Gemini CLI | `gemini extensions install https://github.com/obra/superpowers` |
| GitHub Copilot CLI | `copilot plugin marketplace add obra/superpowers-marketplace` then `copilot plugin install superpowers@superpowers-marketplace` |
| Codex | No marketplace. Fetch and follow `https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md` |
| OpenCode | No marketplace. Fetch and follow `https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md` |
| No plugin system at all | Vendor into the project: `git clone --depth 1 https://github.com/obra/superpowers` to a temp dir (or `npm pack`/download a release), then copy its `skills/` subfolders into the project's `.agents/skills/` so agents can discover them semantically. Record in the report that Superpowers was vendored (this matters for `clean`). |

---

## Setup

### Step 1: Check prerequisites

Run these checks before writing project files. Use the variant matching the host shell.

Unix/macOS:

```bash
openspec --version || echo "OPENSPEC_NOT_INSTALLED"
openspec schemas --json
```

Windows PowerShell:

```powershell
if (Get-Command openspec -ErrorAction SilentlyContinue) { openspec --version; openspec schemas --json } else { echo "OPENSPEC_NOT_INSTALLED" }
```

Then confirm:

1. OpenSpec supports `schema fork`, `schema validate`, `status --json`, and `instructions` (capability check, not a guessed version number).
2. These Superpowers skills are available in the host tool: `writing-plans`, `test-driven-development`, and `verification-before-completion`.

### Step 2: Missing or incompatible dependencies

Do not install anything until the user explicitly agrees. State which dependency is missing or incompatible and ask one concise yes/no question.

After approval:

- Install or upgrade OpenSpec with `npm install -g @fission-ai/openspec@latest`, then rerun the prerequisite checks.
- Install Superpowers using the host tool's row in Table B. Restart or open a new session if the tool requires it to pick up new skills, then recheck the three required skills.
- If the host tool has no plugin system, use the vendoring fallback in Table B and verify the copied skill folders exist.

Do not claim success if either prerequisite still fails. Report the remaining blocker.

### Step 3: Scaffold the OpenSpec workflow

Use the current project root. Do not create a business change. Use your tool's standard file editing capability for all edits (do not assume any specific patch tool exists).

1. If the project has no initialized OpenSpec workspace (missing `openspec/` directory, or it lacks the `openspec/specs/` and `openspec/changes/` structure that `openspec init` creates), run `openspec init --tools <tool>` non-interactively, mapping the host tool from Step 0 to an id from `openspec init --help` (e.g. `cursor`, `claude`, `codex`, `opencode`, `gemini`, `antigravity`); use `--tools none` when unsure. Never re-init an already initialized workspace.
2. If `openspec/schemas/sdd-tdd/` is missing, run `openspec schema fork spec-driven sdd-tdd`.
3. In `schema.yaml`, set the schema `description` to reflect the real workflow, e.g. `SDD-TDD workflow - proposal -> specs -> design -> tasks -> implementation-plan, executed with Superpowers TDD`. Do not leave the forked default description in place.
4. Extend `schema.yaml` with an `implementation-plan` artifact that depends on `tasks`, writes `implementation-plan.md`, and becomes the sole `apply.requires` entry, with `tracks: tasks.md` preserved. Its `instruction` must embed the writing-plans quality bar verbatim, because artifacts are often generated through OpenSpec's own commands (`/opsx:ff`, `/opsx:continue`) where no Superpowers skill is active -- the schema instruction and the template are the only guaranteed carriers of the standard. Include at least: "Write for an engineer with zero context who cannot ask questions. Every step contains complete copy-pasteable code (never a description of code), exact file paths, the RED command with its exact expected failure, and the GREEN command. Read the actual codebase before planning; record build/test commands and conventions in the plan. Steps consisting of prose like 'add validation logic' are incomplete -- rewrite them with the actual code."
5. In the `design` artifact instruction, add: for small changes, write a minimal design (Context plus one Decision) rather than skipping it -- `tasks` depends on `design`, so it must exist.
6. Install the `implementation-plan.md` template by copying `templates/implementation-plan.md` (bundled next to this SKILL.md) verbatim into `openspec/schemas/sdd-tdd/templates/`. Do not write the template from memory or summarize it -- the embedded generation rules and per-step structure are the mechanism that forces detailed plans. If the bundled file is missing, fetch it from `https://raw.githubusercontent.com/kxdds/sdd-tdd/main/skills/bootstrap-sdd-tdd/templates/implementation-plan.md`.
7. Set `schema: sdd-tdd` in `openspec/config.yaml`, preserving any existing context and rules.

Preserve an existing valid `sdd-tdd` schema; repair only what is missing or incompatible.

Portability rule: everything written into `schema.yaml` (descriptions, instructions) and templates must use ASCII-only punctuation (`->` not arrows, `--` not em-dashes). These strings are echoed by the OpenSpec CLI, and Windows consoles with legacy code pages garble multi-byte punctuation into `?` sequences.

### Step 4: Generate the bridge skill

Install the `openspec-superpowers` skill at the location resolved from Table A, if absent. Prefer copying the canonical `skills/openspec-superpowers/SKILL.md` bundled in this repo (or fetch it from `https://raw.githubusercontent.com/kxdds/sdd-tdd/main/skills/openspec-superpowers/SKILL.md`) verbatim -- do not regenerate it from this summary, which omits the quality bars. Only if neither source is reachable, write it from scratch with `plan`, `run`, `verify`, and `archive` modes:

- `plan`: use `superpowers:writing-plans` to create the `implementation-plan.md` artifact from the schema template. Investigate the codebase first; every step must contain complete copy-pasteable code, exact paths, and RED/GREEN commands with expected outcomes -- executable by a zero-context engineer. Map every unchecked OpenSpec task ID to exactly one plan step. Do not check off `tasks.md` during planning.
- `run`: use `superpowers:test-driven-development` for each mapped task -- one focused failing test, observe RED, minimal implementation, observe GREEN, refactor while green -- then mark the plan step and the corresponding OpenSpec checkbox complete. Run project checks and OpenSpec verification. Stop before archive.
- `verify`: resolve actionable verification failures before reporting readiness.
- `archive`: use OpenSpec sync and archive only after all tasks and verification are clean. Archive stays an explicit user action.

Per-tool packaging from Table A:

- Codex: also write `agents/openai.yaml` beside the SKILL.md with `interface.display_name`, `interface.short_description`, and `interface.default_prompt`.
- Antigravity / unknown tools: ensure the project root `AGENTS.md` references the skill. Follow the AGENTS.md write policy below.

#### AGENTS.md write policy

`AGENTS.md` is user-owned. When setup needs to add a reference:

- Never modify, reorder, or delete any existing content. Only append.
- Wrap everything this skill adds in marker comments so `clean` can remove it precisely:

  ```markdown
  <!-- bootstrap-sdd-tdd:begin -->
  ## SDD-TDD workflow
  Use [openspec-superpowers](.agents/skills/openspec-superpowers/SKILL.md): propose -> plan -> run -> verify -> archive (archive is an explicit user action).
  <!-- bootstrap-sdd-tdd:end -->
  ```

- Keep the block minimal -- one heading, a few lines at most. Do not duplicate the bridge skill's documentation into `AGENTS.md`.
- If a `bootstrap-sdd-tdd:begin`/`end` block already exists, update the content inside it instead of appending a second block.
- If the file does not exist, create it containing only the marker block.

Preserve a valid existing `openspec-superpowers` skill; update it only when it omits a required mode or violates these boundaries.

### Step 5: Verify setup

Run:

```text
openspec schema validate sdd-tdd
openspec schema which sdd-tdd
openspec templates --schema sdd-tdd --json
```

Validate created skills with the host tool's skill validator when one exists. Run the repository's lint, typecheck, and tests when scripts exist. Report changed files, validation evidence, how to invoke the workflow, and remaining risks.

Recommend as a closing note: exercise the full loop once on a deliberately small change (propose -> plan -> run -> verify -> archive) before relying on it for real work.

### Invocation after setup

```text
/opsx:propose <change>
openspec-superpowers run <change>
openspec-superpowers archive <change>
```

The final archive step remains an explicit user action.

---

## Clean

Removes everything generated by setup and restores OpenSpec's default orchestration. Never uninstalls the OpenSpec CLI or the Superpowers plugin, and never deletes this bootstrap skill.

First enumerate exactly what will be deleted (only items that actually exist), show the list to the user, and get one explicit confirmation before deleting anything.

Deletion checklist:

1. `openspec/schemas/sdd-tdd/` -- the entire forked schema directory.
2. In `openspec/config.yaml`, remove the `schema: sdd-tdd` line so OpenSpec falls back to the default `spec-driven` orchestration. Preserve user-authored `context` and `rules`.
3. The `openspec-superpowers` bridge skill in every tool directory where it exists: `.codex/skills/openspec-superpowers/`, `.claude/skills/openspec-superpowers/`, `.cursor/skills/openspec-superpowers/`, `.agents/skills/openspec-superpowers/`, `.opencode/skills/openspec-superpowers/`, `.gemini/skills/openspec-superpowers/`.
4. In the project root `AGENTS.md`, remove only the `<!-- bootstrap-sdd-tdd:begin -->` ... `<!-- bootstrap-sdd-tdd:end -->` block. Never touch content outside the markers. If older setups added unmarked references to these skills, remove only those specific lines and leave everything else intact. Delete the file only if it becomes empty and was created by setup.
5. Superpowers skill folders that setup vendored into the project via the no-plugin-system fallback (copies under `.agents/skills/` originating from the Superpowers repo). Do not touch Superpowers installed through a plugin/extension system -- that stays.

Keep (never delete during a normal clean):

- The initialized OpenSpec workspace itself (`openspec/specs/`, `openspec/changes/`, `openspec/config.yaml` created by `openspec init`) -- clean restores the default orchestration; it does not de-initialize OpenSpec.
- `openspec/changes/` and `openspec/changes/archive/` -- user work product, including any `implementation-plan.md` inside them. Delete these only when the user explicitly runs `clean --purge`, and only after a second, separate confirmation.
- OpenSpec's own generated integrations (`.cursor/commands/opsx-*`, per-tool `openspec-*` skills) -- they belong to the default orchestration and are still needed after clean.
- The OpenSpec CLI, the Superpowers plugin/extension, and this `bootstrap-sdd-tdd` skill.

After deleting, verify: run `openspec schemas --json` and confirm `sdd-tdd` no longer appears and the default schema is available. Report the exact list of deleted paths and anything that was kept.
