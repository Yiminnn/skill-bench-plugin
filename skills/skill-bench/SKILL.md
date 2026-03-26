---
name: skill-bench
description: "Use when authoring new Claude Code skills or refining existing ones. New skills: 5-phase workflow (Design → Plan → Build → Validate → Finalize). Existing skills: enter at Refine Mode for validation and targeted refinement."
---

# Skill Bench

Interactive workbench for creating, testing, and refining Claude Code skills through structured conversation.

Five phases: Design (brainstorming) → Plan (writing-plans) → Build & Test (skill-creator) → Validate (consistency-tester) → Finalize (lint + promote).

**Refining an existing skill?** → Jump to [Refine Mode](#refine-mode) — skips to validation and refinement.

## Dependency Check

Before starting, verify required dependencies:

### Superpowers (Phases 1-2)

1. Check if `superpowers:brainstorming` skill is available via the Skill tool
2. If not available, run: `claude plugin install claude-plugins-official/superpowers`
3. Verify installation succeeded
4. If fails: "Skill Bench requires the superpowers plugin. Install manually: `claude plugin install claude-plugins-official/superpowers`" — stop

### Skill-Creator (Phase 3)

1. Check if the `skill-creator` skill is available
2. If not available: clone `skills/skill-creator/` from `https://github.com/anthropics/skills` to `~/.claude/skills/skill-creator/`
3. Verify `~/.claude/skills/skill-creator/SKILL.md` and its internal agents are accessible
4. If fails: "Phase 3 requires skill-creator. Install from https://github.com/anthropics/skills" — stop

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
     "evals_dir": ".skillbench/evals",
     "test_model": "claude-opus-4-6",
     "context_files": []
   }
   ```
   Read the config if it exists. Use `drafts_dir` and `evals_dir` from config.

2. **Update .gitignore** — If `.gitignore` exists and doesn't mention `.skillbench`, offer to add:
   ```
   # Skill Bench artifacts
   .skillbench/test-history/
   .skillbench/workspace/
   ```

3. **Create directories:**
   - Draft: `{drafts_dir}/{skill-name}/`
   - Evals: `{evals_dir}/{skill-name}/`
   - Workspace: `.skillbench/workspace/{skill-name}/`

### Execution

**REQUIRED:** Invoke the `skill-creator` skill with structured handoff context.

Before invoking:

1. **Read design spec** from `.skillbench/specs/{skill-name}-design.md`. If missing, return to Phase 1.
2. **Read implementation plan** from `.skillbench/plans/{skill-name}-plan.md`. If missing, return to Phase 2.
3. **Read skill-authoring constraints** — summarize `references/skill-format.md` for skill-creator: frontmatter schema, size budgets, reference splitting rules.
4. **Read writing-skills principles** — read `references/writing-skills-summary.md` and include as context.
5. **Construct handoff** with override instructions:
   - "Skip interview/intent capture — design spec and plan are already approved"
   - "Build the skill following the implementation plan"
   - "Write skill draft to `{drafts_dir}/{skill-name}/`"
   - "Write evals to `{evals_dir}/{skill-name}/evals.json`"
   - "Write workspace to `.skillbench/workspace/{skill-name}/`"
   - "Run description optimization before exiting"
   - "Return handoff summary: final skill path, eval results path, description optimization results"

Skill-creator runs its eval loop: write skill → create evals with assertions → run with-skill + without-skill → grade → launch eval viewer → iterate → optimize description → return handoff summary.

### Operational Rules

**Size tracking:** After skill-creator exits, report `{filename}: {lines} lines (~{lines * 6} tokens)` for each file in the draft. At 200+ lines suggest splitting to `references/`. At 300+ lines strongly recommend.

**Artifacts:** Skill-creator writes workspace artifacts to `.skillbench/workspace/{skill-name}/`. Eval definitions are preserved in `{evals_dir}/{skill-name}/evals.json`.

**Exit gate:** Skill-creator returns handoff summary with final skill path and eval results.

## Phase 4: Validate

**Multirun consistency testing** — run the skill draft against real test cases multiple times, compare outputs, collect user judgment, and refine.

### Setup

1. **Check for test case library** — Look for `.skillbench/test-cases/{skill-name}.json`
   - If it exists, read it and confirm the cases with the user
   - If it doesn't exist:
     - If `{evals_dir}/{skill-name}/evals.json` exists (from Phase 3), offer to convert: "I can seed your test case library from the Phase 3 evals. Convert?"
       - If yes: read `evals.json`, map each eval case to a test case (eval prompt → test prompt, eval assertions → test description), write to `.skillbench/test-cases/{skill-name}.json`
     - Otherwise, prompt the user to create it:
       > "Create your test case library at `.skillbench/test-cases/{skill-name}.json`. Format:"
     > ```json
     > {
     >   "run_count": 5,
     >   "cases": [
     >     {
     >       "name": "case-name",
     >       "prompt": "input prompt for this case",
     >       "reference_files": ["path/to/reference.pdf"],
     >       "description": "what this case tests"
     >     }
     >   ]
     > }
     > ```
   Wait for the user to create the file before proceeding.

2. **Verify skill draft exists** — Confirm `{drafts_dir}/{skill-name}/SKILL.md` exists from Phase 3. If not, return to Phase 3.

### Execution

Use the Agent tool to spawn the **consistency-tester** agent with:
- Skill draft path: `{drafts_dir}/{skill-name}/SKILL.md`
- Test case library path: `.skillbench/test-cases/{skill-name}.json`
- Test history directory: `.skillbench/test-history/{skill-name}/`

The consistency-tester handles the full validation loop: run collection → consistency summary → user judgment → failure analysis → skill refinement → re-run recommendations. It loops until the user declares validation complete.

**Exit gate:** User declares validation complete.

## Refine Mode

Entry point for improving an existing skill. Skips Phases 1-3.

### Import

See `references/refine-mode.md` for the full import procedure.

1. **Locate skill** — user provides path, skill name (from drafts), or installed skill path
2. **Validate** — parse frontmatter, report identity and size
3. **Copy to drafts** if source is a published skill (preserves original)
4. **Setup workspace** — config, test-cases, test-history directories

### Validate & Refine

Execute **Phase 4** above using the imported skill's draft path. All Phase 4 mechanics apply: test case library, consistency-tester, user judgment loop, skill-refiner.

### Finalize (Optional)

Execute **Phase 5** below. If the skill was copied from a published location, promotion offers to overwrite the original or place at a new path.

## Phase 5: Finalize

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

4. **Description quality (CSO verification):**
   - Verify description is trigger-oriented, not workflow-summarizing (optimized in Phase 3)
   - Glob existing `**/SKILL.md` files, check for description overlap
   - If skill changed during Phase 4, re-run description optimization via skill-creator

5. **Token efficiency:**
   - Report word count (`wc -w`) and line count
   - Flag if SKILL.md exceeds 200 lines or SKILL.md + references exceeds 500 lines

6. **Eval coverage:**
   - Verify `{evals_dir}/{skill-name}/evals.json` exists
   - Check that evals cover the skill's main behavioral capabilities from the plan
   - Warning if missing

7. **Flowchart review** (if skill contains `digraph` blocks):
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
