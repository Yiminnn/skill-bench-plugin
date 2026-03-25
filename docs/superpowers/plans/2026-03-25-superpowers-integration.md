# Superpowers Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Integrate the superpowers brainstorming -> writing-plans -> subagent-driven-development pipeline into skill-bench's workflow, replacing the ad-hoc drafting loop with structured design, planning, and TDD-based skill authoring.

**Architecture:** Skill-bench becomes a thin orchestrator that invokes superpowers skills (brainstorming, writing-plans, subagent-driven-development) at each phase boundary. A new `skill-authoring-plan-template.md` reference file adapts writing-plans' code-oriented task format to skill authoring. Hard dependency on superpowers with auto-install.

**Tech Stack:** Markdown (SKILL.md with YAML frontmatter), Claude Code plugin system

**Spec:** `docs/superpowers/specs/2026-03-25-superpowers-integration-design.md`

---

## File Structure

| File | Action | Responsibility |
|------|--------|----------------|
| `skills/skill-bench/references/skill-authoring-plan-template.md` | Create | Adapts writing-plans' task format for skill authoring (RED/GREEN/REFACTOR cycle) |
| `skills/skill-bench/SKILL.md` | Rewrite | New 4-phase orchestrator: Design → Plan → Build & Test → Finalize |
| `.claude-plugin/plugin.json` | Modify | Version bump 0.1.0 → 0.2.0, add dependencies field |
| `CLAUDE.md` | Modify | Reflect superpowers dependency and new phase mapping |
| `README.md` | Modify | Update workflow description, add prerequisites section |

Unchanged: `references/skill-format.md`, `references/anti-patterns.md`, `agents/skill-tester.md`, `agents/skill-explorer.md`

---

### Task 1: Create skill-authoring-plan-template.md

**Files:**
- Create: `skills/skill-bench/references/skill-authoring-plan-template.md`

This is the critical adaptation layer. It must exist before SKILL.md can reference it.

- [ ] **Step 1: Write the template file**

Write `skills/skill-bench/references/skill-authoring-plan-template.md` with this exact content:

````markdown
# Skill-Authoring Plan Template

This template adapts writing-plans' task format for skill authoring. Provide this to writing-plans as context so it produces skill-appropriate tasks instead of code-oriented ones.

## Key Adaptation

Code plans: write test → implement → run test → commit
Skill plans: define behavior → baseline test → write section → simulate → pressure test → commit

This maps to writing-skills' RED → GREEN → REFACTOR cycle:
- Steps 1-3 = **RED** (baseline fails without skill)
- Step 4 = **GREEN** (write minimal skill to pass)
- Steps 5-7 = **REFACTOR** (test, find gaps, close loopholes)

## Task Step Template

Each task in the plan should use this step sequence:

```
### Task N: [Behavioral Capability Name]

**Files:**
- Create/Modify: `{drafts_dir}/{skill-name}/SKILL.md`
- Create (if needed): `{drafts_dir}/{skill-name}/references/{name}.md`

- [ ] **Step 1: Define expected behavior**

What should an agent do when given this skill section?
What should it NOT do?
What rationalizations might it use to skip this behavior?

- [ ] **Step 2: Write baseline pressure scenario**

A prompt that tests this behavior WITHOUT the skill. Must:
- Create realistic pressure (time, complexity, sunk cost)
- NOT mention the skill or its rules
- Be specific enough to observe compliance/violation

[Show exact prompt text]

- [ ] **Step 3: Run baseline test (RED)**

Spawn subagent WITHOUT skill. Give it the pressure scenario. Document:
- What the agent did
- Rationalizations used (verbatim)

Expected: Agent violates or skips the target behavior.

- [ ] **Step 4: Write the skill section (GREEN)**

Write the SKILL.md content addressing violations from Step 3.
[Show exact markdown content]

After writing, report: `SKILL.md: {lines} lines (~{lines * 6} tokens)`

- [ ] **Step 5: Run simulated test**

Spawn the skill-tester agent with current SKILL.md draft + sample input.
Expected: pass or partial.

- [ ] **Step 6: Run pressure test (REFACTOR)**

Spawn fresh subagent WITH skill loaded. Give it scenario from Step 2.
Expected: Agent complies with skill instructions.

- [ ] **Step 7: Iterate if needed**

If Step 5 or 6 failed: identify gap, edit skill section, re-run failing test.

- [ ] **Step 8: Commit**
```

## Task Decomposition

Break skills into tasks by **behavioral capability**, not by markdown heading:

1. Frontmatter + overview (skill identity and trigger)
2. Core workflow (main process the skill teaches)
3. Quality gates / red flags (discipline enforcement)
4. Reference files (if size budget requires splitting)
5. Companion agent (if the skill spawns agents)

Not every skill needs all 5. Simple skills may only need 1-2.

## Size Budget Checkpoints

After each task, check cumulative size:
- SKILL.md < 200 lines: OK
- SKILL.md 200-300 lines: suggest splitting to `references/`
- SKILL.md > 300 lines: strongly recommend splitting

Token estimate: `lines * 6`

## Plan Header

Use this header for skill-authoring plans:

```
# [Skill Name] Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
> to implement this plan. Reviewer 2 runs behavioral testing (simulated execution +
> pressure tests) instead of code quality review.

**Goal:** Create the {skill-name} skill that [one sentence]

**Architecture:** Skill-bench orchestrated. SKILL.md + references/ for heavy content.
Testing via skill-tester agent (simulated) and pressure testing (writing-skills TDD).

**Tech Stack:** Markdown (SKILL.md with YAML frontmatter), Claude Code plugin system
```

## Concurrent Edit Protection

Include this reminder in EVERY task that writes to draft files:

> Before every Write or Edit: read file, compute SHA256, compare to last-known hash.
> If different: show what changed, ask overwrite/incorporate/keep. Update hash after write.
````

- [ ] **Step 2: Validate the template**

Run these checks:

```bash
# File exists
ls skills/skill-bench/references/skill-authoring-plan-template.md

# Line count under 200
wc -l skills/skill-bench/references/skill-authoring-plan-template.md

# Contains key sections
grep -c "Key Adaptation" skills/skill-bench/references/skill-authoring-plan-template.md
grep -c "Task Step Template" skills/skill-bench/references/skill-authoring-plan-template.md
grep -c "Task Decomposition" skills/skill-bench/references/skill-authoring-plan-template.md
grep -c "Size Budget" skills/skill-bench/references/skill-authoring-plan-template.md
grep -c "Plan Header" skills/skill-bench/references/skill-authoring-plan-template.md
grep -c "Concurrent Edit Protection" skills/skill-bench/references/skill-authoring-plan-template.md
```

Expected: File exists, under 200 lines, all 6 section headers found (count = 1 each).

- [ ] **Step 3: Commit**

```bash
git add skills/skill-bench/references/skill-authoring-plan-template.md
git commit -m "feat: add skill-authoring plan template for writing-plans adaptation"
```

---

### Task 2: Rewrite SKILL.md

**Files:**
- Rewrite: `skills/skill-bench/SKILL.md` (currently 209 lines → target ~130 lines)

Complete replacement. The new SKILL.md is a thin orchestrator that delegates to superpowers skills.

- [ ] **Step 1: Write the new SKILL.md**

Replace the entire content of `skills/skill-bench/SKILL.md` with:

````markdown
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
````

- [ ] **Step 2: Validate the new SKILL.md**

Run these checks:

```bash
# Line count under 200
wc -l skills/skill-bench/SKILL.md
# Expected: ~130 lines

# Frontmatter valid
head -4 skills/skill-bench/SKILL.md
# Expected: --- / name: skill-bench / description: "Use when..." / ---

# All 4 phases present
grep -c "## Phase 1: Design" skills/skill-bench/SKILL.md
grep -c "## Phase 2: Plan" skills/skill-bench/SKILL.md
grep -c "## Phase 3: Build & Test" skills/skill-bench/SKILL.md
grep -c "## Phase 4: Finalize" skills/skill-bench/SKILL.md
# Expected: 1 each

# Superpowers skills referenced
grep -c "superpowers:brainstorming" skills/skill-bench/SKILL.md
grep -c "superpowers:writing-plans" skills/skill-bench/SKILL.md
grep -c "superpowers:subagent-driven-development" skills/skill-bench/SKILL.md
# Expected: 1+ each

# References to existing files
grep -c "references/skill-format.md" skills/skill-bench/SKILL.md
grep -c "references/anti-patterns.md" skills/skill-bench/SKILL.md
grep -c "references/skill-authoring-plan-template.md" skills/skill-bench/SKILL.md
grep -c "skill-tester" skills/skill-bench/SKILL.md
# Expected: 1+ each

# Dependency check present
grep -c "claude plugins add claude-plugins-official/superpowers" skills/skill-bench/SKILL.md
# Expected: 2 (check + fallback message)

# Flowchart review check present in Phase 4
grep -c "Flowchart review" skills/skill-bench/SKILL.md
# Expected: 1
```

- [ ] **Step 3: Commit**

```bash
git add skills/skill-bench/SKILL.md
git commit -m "feat: rewrite SKILL.md to orchestrate superpowers pipeline

Replaces ad-hoc 4-phase workflow with superpowers integration:
- Phase 1 (Design): delegates to superpowers:brainstorming
- Phase 2 (Plan): delegates to superpowers:writing-plans with skill-authoring template
- Phase 3 (Build & Test): delegates to superpowers:subagent-driven-development
  with adapted reviewer roles (spec compliance + behavioral testing)
- Phase 4 (Finalize): enhanced lint with CSO and token efficiency checks"
```

---

### Task 3: Update plugin.json

**Files:**
- Modify: `.claude-plugin/plugin.json`

- [ ] **Step 1: Update plugin.json**

Replace the content of `.claude-plugin/plugin.json` with:

```json
{
  "name": "skill-bench",
  "description": "Interactive skill authoring bench — create, test, and refine Claude Code skills through conversation",
  "version": "0.2.0",
  "author": {
    "name": "DeffAI"
  },
  "license": "MIT",
  "keywords": ["skills", "authoring", "testing", "workflows"],
  "dependencies": ["claude-plugins-official/superpowers"],
  "skills": "./skills/",
  "agents": "./agents/"
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -c "import json; json.load(open('.claude-plugin/plugin.json')); print('valid JSON')"
# Expected: "valid JSON"

grep '"version"' .claude-plugin/plugin.json
# Expected: "version": "0.2.0"

grep '"dependencies"' .claude-plugin/plugin.json
# Expected: "dependencies": ["claude-plugins-official/superpowers"]
```

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "chore: bump version to 0.2.0, add superpowers dependency"
```

---

### Task 4: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Write updated CLAUDE.md**

Replace the content of `CLAUDE.md` with:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin for interactive skill authoring. It provides a `skill-bench` skill (4-phase workflow: Design → Plan → Build & Test → Finalize) plus two companion agents (`skill-tester`, `skill-explorer`).

**Requires:** [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin (auto-installed on first use).

Install: `claude plugins add https://github.com/deffai/skill-bench-plugin`

## Development

No build, test, or lint commands — the entire codebase is markdown files and a JSON plugin manifest. Validation happens at skill-authoring time via the lint pass in Phase 4 of the workflow.

## Architecture

```
.claude-plugin/plugin.json   # Plugin manifest — declares skills/ and agents/ dirs
skills/skill-bench/
  SKILL.md                   # Main skill: thin orchestrator invoking superpowers at phase boundaries
  references/
    skill-format.md          # Frontmatter schema + size budgets for SKILL.md files
    anti-patterns.md         # Lint checklist used in Phase 4 (Finalize)
    skill-authoring-plan-template.md  # Adapts writing-plans' task format for skill authoring
agents/
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

`plugin.json` points to `./skills/` and `./agents/` — the plugin loader discovers contents by convention.

### Phase → Superpowers Mapping

| Phase | Superpowers Skill | What It Does |
|-------|-------------------|-------------|
| 1. Design | `superpowers:brainstorming` | Collaborative design spec with 2-3 approaches |
| 2. Plan | `superpowers:writing-plans` | Bite-sized tasks using skill-authoring template |
| 3. Build & Test | `superpowers:subagent-driven-development` | Execute tasks with spec + behavioral review |
| 4. Finalize | (skill-bench native) | Lint, CSO check, promote |

### Reviewer Roles in Phase 3

- **Reviewer 1 (Spec Compliance):** Checks against design spec + `skill-format.md`
- **Reviewer 2 (Behavioral Testing):** Spawns `skill-tester` for simulated execution + pressure tests from writing-skills TDD

### Runtime Artifacts (created in user projects, not this repo)

- `.skillbench/config.json` — project config (drafts_dir, test_model, context_files)
- `.skillbench/specs/{skill-name}-design.md` — design specs from Phase 1
- `.skillbench/plans/{skill-name}-plan.md` — implementation plans from Phase 2
- `.skillbench/test-history/{skill-name}/` — simulated + pressure test results
- `skills/drafts/` — default location for in-progress skill drafts

## Conventions

- Skill names must be kebab-case: `^[a-z0-9]+(-[a-z0-9]+)*$`
- Skill descriptions must start with "Use when" or "Use this when"
- SKILL.md files should stay under 200 lines; split heavy content into `references/`
- Token estimate for markdown: `lines * 6`
- The skill-explorer agent is read-only — it must never write files
- The skill-tester agent uses model `opus`; skill-explorer uses `haiku`
```

- [ ] **Step 2: Validate CLAUDE.md**

```bash
# Contains superpowers dependency note
grep "superpowers" CLAUDE.md | head -5
# Expected: multiple matches (Requires line, phase mapping table, etc.)

# Contains new phase names
grep "Design" CLAUDE.md
grep "Plan" CLAUDE.md
grep "Build & Test" CLAUDE.md
grep "Finalize" CLAUDE.md

# Contains new artifact locations
grep "specs/" CLAUDE.md
grep "plans/" CLAUDE.md
```

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md for superpowers integration"
```

---

### Task 5: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Write updated README.md**

Replace the content of `README.md` with:

```markdown
# Skill Bench

A Claude Code plugin for interactive skill authoring. Create, test, and refine Claude Code skills through conversation.

## Prerequisites

Skill Bench requires the [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin. It will be installed automatically on first use, or you can install it manually:

```bash
claude plugins add claude-plugins-official/superpowers
```

## Install

```bash
claude plugins add https://github.com/deffai/skill-bench-plugin
```

Or install locally for development:

```bash
claude plugins add /path/to/skill-bench-plugin
```

## What It Does

Skill Bench guides you through creating Claude Code skills in four phases:

1. **Design** — Brainstorm what your skill should do, explore approaches, produce a design spec
2. **Plan** — Generate a structured implementation plan with TDD-style tasks
3. **Build & Test** — Execute the plan with simulated testing and pressure testing
4. **Finalize** — Lint, validate references, and promote to your target location

Each phase delegates to a superpowers skill:

| Phase | Skill | Purpose |
|-------|-------|---------|
| Design | `superpowers:brainstorming` | Collaborative design with trade-off analysis |
| Plan | `superpowers:writing-plans` | Bite-sized tasks adapted for skill authoring |
| Build & Test | `superpowers:subagent-driven-development` | Execute with spec review + behavioral testing |
| Finalize | skill-bench native | Lint pass + promotion |

## Components

| Component | Type | Purpose |
|-----------|------|---------|
| `skill-bench` | Skill | Main authoring workflow — orchestrates the 4-phase conversation |
| `skill-tester` | Agent | Simulated skill execution with structured evaluation |
| `skill-explorer` | Agent | Browse and summarize existing skill drafts |

## Usage

### Start authoring a new skill

Invoke the skill-bench skill in Claude Code. It will walk you through the process.

### Explore existing drafts

Ask Claude to use the skill-explorer agent:
> "Show me my skill drafts"

### Test a draft

During the Build & Test phase, the workflow automatically runs:
- **Simulated execution** via the skill-tester agent
- **Pressure testing** via writing-skills' TDD methodology (baseline without skill → verify compliance with skill)

## Project Config

On first use, Skill Bench creates `.skillbench/config.json` in your project:

```json
{
  "drafts_dir": "skills/drafts",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

- `drafts_dir` — where draft skills are written (default: `skills/drafts`)
- `test_model` — model for simulated testing (default: `claude-opus-4-6`)
- `context_files` — glob patterns for files to include in test context

### Artifact Locations

| Directory | Purpose | Git Track? |
|-----------|---------|------------|
| `.skillbench/config.json` | Project settings | Yes |
| `.skillbench/specs/` | Design specs from Phase 1 | Yes |
| `.skillbench/plans/` | Implementation plans from Phase 2 | Yes |
| `.skillbench/test-history/` | Simulated + pressure test results | No (gitignore) |

The skill-bench workflow will offer to update `.gitignore` on first use.

## License

MIT
```

- [ ] **Step 2: Validate README.md**

```bash
# Prerequisites section exists
grep -c "Prerequisites" README.md
# Expected: 1

# Phase names updated
grep "Design" README.md | head -2
grep "Build & Test" README.md | head -2

# Superpowers references present
grep -c "superpowers" README.md
# Expected: 5+

# Artifact locations documented
grep "specs/" README.md
grep "plans/" README.md
```

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: update README for superpowers integration and new workflow"
```
