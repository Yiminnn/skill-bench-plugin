---
name: skill-bench
description: "Use when authoring new Claude Code skills or refining existing skill prompts. Guides a structured workflow: design via brainstorming, plan the skill structure, build with TDD pressure testing, and finalize with validation."
---

# Skill Bench

Interactive workbench for creating, testing, and refining Claude Code skills through structured conversation.

You guide the user through four phases: designing the skill (via brainstorming), planning the structure, building and testing with TDD (simulated execution + pressure testing), and finalizing with validation. You invoke superpowers skills at phase boundaries and provide skill-authoring context.

## Dependency Check

Before starting any phase, verify the superpowers plugin is installed:

1. Check if `superpowers:brainstorming` skill is available via the Skill tool
2. If not available, run: `claude plugins add claude-plugins-official/superpowers`
3. Verify installation succeeded
4. If installation fails, tell the user: "Skill Bench requires the superpowers plugin. Install manually: `claude plugins add claude-plugins-official/superpowers`" — then stop

## Phase 1: Design

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming`

Before invoking brainstorming:

1. **Set spec location:** Tell brainstorming: "Save the design spec to `.skillbench/specs/{skill-name}-design.md`"
2. **Provide skill-authoring context:** Read `references/skill-format.md` and summarize for brainstorming:
   - A Claude Code skill is a SKILL.md file with YAML frontmatter (`name`, `description`) and markdown content
   - `name` must be kebab-case: `^[a-z0-9]+(-[a-z0-9]+)*$`
   - `description` must start with "Use when", max 1024 chars, trigger-oriented (not workflow-summarizing)
   - SKILL.md should stay under 200 lines; heavy content splits to `references/`
   - Token estimate: `lines * 6`
   - Skills can include companion agents (`.md` files in `agents/`)
3. **Check for name collisions:** Glob `skills/**/SKILL.md` and `agents/**/*.md`. Warn about shadowing.

Then invoke brainstorming. It handles: clarifying questions, 2-3 approaches with trade-offs, sectioned design for approval, spec writing and commit.

**Exit gate:** User-approved design spec exists in `.skillbench/specs/`.

## Phase 2: Plan

**REQUIRED SUB-SKILL:** Use `superpowers:writing-plans`

Before invoking writing-plans:

1. **Set plan location:** Tell writing-plans: "Save the plan to `.skillbench/plans/{skill-name}-plan.md`"
2. **Load plan template:** Read `references/skill-authoring-plan-template.md` and provide it as context so writing-plans produces skill-authoring tasks (define behavior → baseline test → write section → simulate → pressure test) instead of code-oriented tasks.
3. **Provide the design spec:** Read `.skillbench/specs/{skill-name}-design.md`
   If the spec file does not exist, return to Phase 1 before proceeding.

Then invoke writing-plans. It handles: file structure mapping, bite-sized task creation, self-review, plan writing and commit.

**Exit gate:** User-approved plan exists in `.skillbench/plans/`.

## Phase 3: Build & Test

### Setup

1. **Initialize project config** (first time only) — If `.skillbench/config.json` doesn't exist, create it:
   ```json
   {
     "drafts_dir": "skills/drafts",
     "test_model": "claude-opus-4-6",
     "context_files": []
   }
   ```
   Read the config if it exists. Use `drafts_dir` as the base directory for new drafts.

2. **Update .gitignore** — If `.gitignore` exists and doesn't mention `.skillbench`, offer to add:
   ```
   # Skill Bench test history
   .skillbench/test-history/
   ```

3. **Create the draft directory** — `{drafts_dir}/{skill-name}/`

### Execution

**REQUIRED SUB-SKILL:** Use `superpowers:subagent-driven-development`

Provide these role adaptations for skill authoring:

**Implementer:** Writes skill sections and reference files following the plan. Self-reviews against `references/skill-format.md`. Reports standard status (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED).

**Reviewer 1 — Spec Compliance:** Checks skill section against the design spec from Phase 1. Validates frontmatter, size budgets, and cross-references against `references/skill-format.md`.

**Reviewer 2 — Behavioral Testing:** Replaces code quality review. Runs two tests:
1. **Simulated execution** — spawns the **skill-tester** agent with current SKILL.md draft + sample input from the plan
   (Use the Agent tool to spawn the skill-tester agent, which is registered by this plugin)
2. **Pressure test** — spawns a fresh subagent WITH the skill loaded, gives it the baseline pressure scenario from the plan task

Spec compliance must pass before behavioral testing. Both must pass before marking a task complete.

### Operational Rules

**Size tracking:** After each task, report `{filename}: {lines} lines (~{lines * 6} tokens)`. At 200+ lines suggest splitting to `references/`. At 300+ lines strongly recommend splitting.

**Concurrent edit protection:** Before every Write or Edit to a draft file, read the file and compute SHA256. Compare to last-known hash. If different: show what changed, ask whether to overwrite, incorporate, or keep. Update hash after successful write.

**Test history:** Save to `.skillbench/test-history/{skill-name}/`:
- `{timestamp}.json` — simulated test results (existing format)
- `{timestamp}-pressure.json` — pressure test results

**Exit gate:** All plan tasks complete. All reviews (spec compliance + behavioral testing) pass.

## Phase 4: Finalize

Validate the skill and promote to its final location. Load `references/anti-patterns.md` for the lint checklist.

### Lint Pass

1. **Frontmatter validation:**
   - `name` matches `^[a-z0-9]+(-[a-z0-9]+)*$`
   - `description` starts with "Use when" and is ≤ 1024 chars
   - `model` is valid enum or omitted

2. **Content checks:**
   - No "You are..." instructions
   - No hardcoded file paths that don't exist (Glob to verify)
   - No references to non-standard tools
   - Size within budget (report final line count + token estimate)

3. **Reference validation:**
   - Scan for `skill: \`name\`` — Glob to verify each exists
   - Scan for `**name** agent` — Glob to verify each exists

4. **Description quality (CSO):**
   - Description is trigger-oriented, not workflow-summarizing
   - Glob existing `**/SKILL.md` files, check for description overlap
   - Keywords present for discovery

5. **Token efficiency:**
   - Report word count (`wc -w`) and line count
   - Flag if SKILL.md exceeds 200 lines or SKILL.md + references exceeds 500 lines

6. **Flowchart review** (if skill contains `digraph` blocks):
   - Used only for non-obvious decision points, not linear instructions
   - Labels have semantic meaning (not "step1", "helper2")
   - No code in flowchart labels

Present findings as **Blocking** (must fix), **Warning** (should fix), **Info** (optional).

### Promotion

1. **Ask target location:** Project-local `skills/{skill-name}/`, plugin directory, or custom path
2. **Move files** from `{drafts_dir}/{skill-name}/` to target. Include references, scripts, companion agents.
3. **Verify move** — Glob target to confirm all files landed.
4. **Suggest commit:** `feat: add {skill-name} skill`
5. **Clean up** — Ask if user wants to remove draft directory. Test history is preserved.
