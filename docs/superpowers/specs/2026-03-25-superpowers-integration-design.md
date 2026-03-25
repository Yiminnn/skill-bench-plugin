# Superpowers Integration Design

**Goal:** Integrate the superpowers brainstorming -> writing-plans -> executing-plans pipeline into skill-bench's draft skill creation workflow, with writing-skills' TDD methodology as the testing backbone.

**Approach:** Direct Superpowers Invocation — skill-bench becomes a thin orchestrator that invokes superpowers skills at each phase boundary, providing skill-authoring context via reference files. Hard dependency on superpowers; auto-install if missing.

---

## 1. Dependency & Bootstrap

At the start of every skill-bench invocation, before entering any phase:

1. **Check for superpowers** — verify `superpowers:brainstorming` skill is available
2. **Auto-install if missing** — run `claude plugins add claude-plugins-official/superpowers` and confirm success
3. **Proceed** — once superpowers is confirmed, enter Phase 1

No fallback path. If superpowers can't be installed (network issue, etc.), skill-bench stops and tells the user to install it manually. No "degraded mode."

This is a preamble in SKILL.md (~10 lines of instructions).

## 2. Phase Mapping

The current 4-phase structure restructures into:

```
Phase 1: Design (was: Intent)
  REQUIRED SUB-SKILL: superpowers:brainstorming
  - Before invoking brainstorming, skill-bench states: "Save the design spec to .skillbench/specs/{skill-name}-design.md"
    (brainstorming respects user preferences for spec location over its default docs/superpowers/specs/)
  - Before invoking brainstorming, skill-bench loads references/skill-format.md and provides a summary
    as context: what a Claude Code skill is, frontmatter schema, size budgets, description conventions
  - Spec saved to .skillbench/specs/{skill-name}-design.md
  - Exit gate: user-approved design spec

Phase 2: Plan (new phase)
  REQUIRED SUB-SKILL: superpowers:writing-plans
  - Before invoking writing-plans, skill-bench states: "Save the plan to .skillbench/plans/{skill-name}-plan.md"
    (writing-plans respects user preferences for plan location over its default docs/superpowers/plans/)
  - Before invoking writing-plans, skill-bench loads references/skill-authoring-plan-template.md and
    provides it as context so writing-plans produces skill-authoring tasks instead of code-oriented tasks
  - Plan saved to .skillbench/plans/{skill-name}-plan.md
  - Exit gate: user-approved plan

Phase 3: Build & Test (was: Draft + Test, merged)
  REQUIRED SUB-SKILL: superpowers:subagent-driven-development
  - Per task in the plan:
    Implementer -> writes skill section + reference files
    Reviewer 1 (spec compliance) -> checks against design spec + skill-format.md
    Reviewer 2 (behavioral testing) -> skill-tester simulation + writing-skills pressure test
  - Skill-bench's Phase 2 mechanics preserved: size tracking, auto-split, concurrent edit protection
  - Exit gate: all tasks complete, all reviews pass

Phase 4: Finalize (stays, enhanced)
  - Skill-bench's existing lint pass from references/anti-patterns.md
  - Writing-skills' quality checks: CSO validation, token efficiency, flowchart review
  - Promote to target location, suggest commit
```

Key changes from current skill-bench:

- **Old Phases 2 and 3 merge** into Phase 3 (Build & Test). The old Phase 2 was "draft iteratively" and Phase 3 was "test iteratively" — now they're one TDD loop per task.
- **Phase 2 is new** — there was no planning step before.
- **Phase 1 gets deeper** — brainstorming produces a real design spec with 2-3 approaches and trade-offs, not just a one-sentence intent confirmation.

## 3. The Skill-Authoring Plan Template

New file: `references/skill-authoring-plan-template.md`. This is the critical adaptation layer that tells writing-plans how to produce tasks for skill authoring instead of code.

### Standard writing-plans task format (code-oriented):

```
- [ ] Write failing test
- [ ] Run test to verify it fails
- [ ] Write minimal implementation
- [ ] Run test to verify it passes
- [ ] Commit
```

### Skill-authoring task equivalent:

```
- [ ] Define expected behavior (what should an agent do when given this skill section?)
- [ ] Write baseline pressure scenario (a prompt that tests this behavior WITHOUT the skill)
- [ ] Run baseline test (spawn subagent without skill, document natural behavior + rationalizations)
- [ ] Write the skill section (SKILL.md content following the plan)
- [ ] Run simulated test (spawn skill-tester agent with sample input)
- [ ] Run pressure test (spawn subagent WITH skill, verify compliance)
- [ ] Iterate if simulated or pressure test fails
- [ ] Commit
```

This is writing-skills' RED -> GREEN -> REFACTOR cycle expressed as plan steps:

- Steps 1-3 = **RED** (baseline fails without skill)
- Step 4 = **GREEN** (write minimal skill to pass)
- Steps 5-7 = **REFACTOR** (test, find gaps, close loopholes)

### Task decomposition guidance

Each task maps to a logical unit of the skill — not one task per markdown heading, but one task per behavioral capability. Examples:

- Task 1: Frontmatter + overview (the skill's identity and trigger)
- Task 2: Core workflow section (the main process the skill teaches)
- Task 3: Quality gates / red flags (the discipline enforcement)
- Task 4: Reference files (if auto-split triggered by size)
- Task 5: Companion agent definition (if the skill spawns agents)

The template also includes:

- Size budget checkpoints (report line count + token estimate after each task)
- Auto-split rules (when a task produces content that pushes SKILL.md past 200 lines, split to `references/`)
- Concurrent edit protection reminder (hash-check before every write)

## 4. Reviewer Role Adaptation

Subagent-driven-development dispatches three roles per task. For skill authoring, they adapt:

### Implementer subagent (unchanged role, different output)

- Writes skill section + reference files following the plan task
- Self-reviews against skill-format.md conventions
- Commits
- Reports status: DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED

### Reviewer 1: Spec Compliance (same role, skill-specific criteria)

- Checks skill section against the design spec from Phase 1
- Validates frontmatter against `references/skill-format.md`
- Confirms size budgets respected
- Verifies cross-references resolve (skill/agent references point to real files)

### Reviewer 2: Behavioral Testing (replaces "code quality")

Runs two tests:

1. **Simulated execution** — spawns the skill-tester agent with the current SKILL.md draft + a sample input from the plan. Checks: does the skill produce correct output?
2. **Pressure test** — spawns a fresh subagent WITH the skill loaded, gives it the baseline pressure scenario from the plan task. Checks: does the agent comply with the skill's instructions under pressure?

Reports: pass/partial/fail for each test, specific issues, suggested fixes.

The pressure test is what differentiates this from the current skill-bench. Today, skill-bench only does simulated execution (skill-tester). The integrated workflow adds behavioral validation from writing-skills' TDD methodology.

### Failure handling

Same as subagent-driven-development:

- Reviewer finds issues -> implementer fixes -> reviewer re-reviews -> repeat until approved
- Both reviewers must pass before task is marked complete
- Spec compliance runs before behavioral testing (no point pressure-testing a skill that doesn't match the spec)

## 5. Artifact Locations

```
.skillbench/
  config.json                          # Project config (unchanged)
  specs/{skill-name}-design.md         # Design specs from Phase 1 (brainstorming)
  plans/{skill-name}-plan.md           # Implementation plans from Phase 2 (writing-plans)
  test-history/{skill-name}/           # Test run results (unchanged)
    {timestamp}.json                   # Simulated test results (existing format)
    {timestamp}-pressure.json          # Pressure test results (new)

{drafts_dir}/{skill-name}/            # Skill drafts from Phase 3 (from config, default: skills/drafts)
  SKILL.md
  references/
  agents/                              # Companion agent definitions if any
```

Changes from current:

- **New:** `.skillbench/specs/` — brainstorming's spec output redirected here
- **New:** `.skillbench/plans/` — writing-plans' output redirected here
- **New:** `{timestamp}-pressure.json` in test history — records pressure test results alongside simulated test results
- **Unchanged:** config.json, drafts_dir, simulated test history format

The `.gitignore` guidance updates: `.skillbench/test-history/` still gitignored (now includes pressure traces too). `.skillbench/specs/` and `.skillbench/plans/` are committable — they're useful project history.

## 6. Files Changed in the Plugin

| File | Action | What Changes |
|------|--------|-------------|
| `skills/skill-bench/SKILL.md` | Rewrite | New 4-phase flow with superpowers invocations |
| `skills/skill-bench/references/skill-authoring-plan-template.md` | Create | Adaptation layer for writing-plans |
| `skills/skill-bench/references/skill-format.md` | Unchanged | Still loaded by implementer + spec reviewer |
| `skills/skill-bench/references/anti-patterns.md` | Unchanged | Still loaded in Phase 4 finalize |
| `agents/skill-tester.md` | Unchanged | Still spawned by Reviewer 2 for simulated testing |
| `agents/skill-explorer.md` | Unchanged | Still works independently |
| `.claude-plugin/plugin.json` | Minor edit | Bump version to 0.2.0. Add `"dependencies": ["claude-plugins-official/superpowers"]` (informational — plugin loader doesn't enforce this, but documents the requirement) |
| `CLAUDE.md` | Update | Reflect new architecture and superpowers dependency |
| `README.md` | Update | Reflect new workflow, add superpowers prerequisite |
