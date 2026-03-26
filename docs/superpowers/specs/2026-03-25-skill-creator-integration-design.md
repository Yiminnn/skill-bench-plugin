# Skill-Creator Integration Design

Integrate Anthropic's official `skill-creator` (from `anthropics/skills/skill-creator/`) and `superpowers:writing-skills` methodology into the skill-bench full workflow pipeline.

## Decisions

| Decision | Choice |
|----------|--------|
| Skill-creator role | Primary build-and-test engine (Phase 3) |
| Writing-skills role | Enriches plan template + lightweight summary as Phase 3 context |
| Express path | N/A (removed) |
| Consistency-tester (Phase 4) | Stays as separate sequential phase |
| Description optimization | End of Phase 3, skill-creator's natural exit step |
| Integration approach | Invoke installed skill-creator via Skill tool with structured handoff |

## Section 1: Dependency Management

Skill-creator is a skill repo (`anthropics/skills/skill-creator/`), not a plugin. Auto-install before Phase 3.

**Install check (same pattern as superpowers):**

1. Check if `skill-creator` skill is available via Skill tool
2. If not: clone/install from `anthropics/skills` repo to `~/.claude/skills/skill-creator/`
3. Verify skill and internal components are accessible:
   - Agents: `grader.md`, `comparator.md`, `analyzer.md`
   - Scripts: `aggregate_benchmark`, `run_loop`, `generate_review.py`
4. If install fails: tell user to install manually, provide instructions, stop

Skill-creator is a Phase 3 dependency alongside superpowers.

## Section 2: Phase 3 Restructure — Skill-Creator Handoff

Replace `superpowers:subagent-driven-development` (with dual reviewers) with a structured handoff to skill-creator.

### Handoff Context

Skill-bench passes to skill-creator:

1. **Design spec** — from `.skillbench/specs/{skill-name}-design.md` (Phase 1 output)
2. **Implementation plan** — from `.skillbench/plans/{skill-name}-plan.md` (Phase 2 output)
3. **Skill-authoring constraints** — from `references/skill-format.md`: frontmatter schema, size budgets, reference splitting rules
4. **Writing-skills summary** — from `references/writing-skills-summary.md`: iron law, RED-GREEN-REFACTOR, CSO, rationalization bulletproofing, token efficiency (~50 lines)
5. **Override instructions:**
   - "Skip interview/intent capture — design spec and plan are already approved"
   - "Write skill draft to `{drafts_dir}/{skill-name}/`"
   - "Write evals to `.skillbench/evals/{skill-name}/evals.json`"
   - "Write workspace to `.skillbench/workspace/{skill-name}/`"
   - "Run description optimization before exiting"
   - "End with a structured handoff summary: final skill path, eval results path, description optimization results"

### Skill-Creator Loop

Skill-creator runs its natural loop guided by the plan:

1. Writes skill from plan tasks
2. Creates `evals.json` with assertions derived from plan's behavioral capabilities
3. Runs with-skill + without-skill (baseline comparison)
4. Grades via its grader agent
5. Launches eval viewer for human review
6. Iterates based on feedback
7. Runs description optimization loop
8. Returns structured handoff summary

**Exit gate:** Skill-creator returns handoff summary. Skill-bench reads the final skill draft and proceeds to Phase 4.

### Setup (unchanged from current Phase 3)

- Initialize `.skillbench/config.json` if first time
- Update `.gitignore` if needed
- Create draft directory `{drafts_dir}/{skill-name}/`

## Section 3: Artifact Layout

### New Directories

| Directory | Purpose | Git Track? |
|-----------|---------|------------|
| `.skillbench/evals/{skill-name}/` | `evals.json` + assertion definitions | Yes |
| `.skillbench/workspace/{skill-name}/` | Skill-creator workspace: `iteration-N/eval-*/with_skill/`, `without_skill/`, `grading.json`, `benchmark.json`, `feedback.json` | No |

### Updated .gitignore

```
# Skill Bench artifacts
.skillbench/test-history/
.skillbench/workspace/
```

### Config Update

```json
{
  "drafts_dir": "skills/drafts",
  "evals_dir": ".skillbench/evals",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

### Existing Directories (unchanged)

| Directory | Purpose |
|-----------|---------|
| `.skillbench/specs/` | Design specs from Phase 1 |
| `.skillbench/plans/` | Implementation plans from Phase 2 |
| `.skillbench/test-cases/` | Multirun test case libraries for Phase 4 |
| `.skillbench/test-history/` | Consistency-tester results + refinement records |

## Section 4: Plan Template Enhancement

The existing `references/skill-authoring-plan-template.md` is enriched with writing-skills methodology so skill-creator receives better-structured tasks.

### Additions

**Pressure scenario design** — Step 2 (baseline pressure scenario) gains guidance on combining pressure types:
- Time pressure ("the user is waiting")
- Sunk cost ("you've already written most of the code")
- Authority ("the senior engineer said to skip this")
- Exhaustion ("this is task 7 of 12")
- Require at least 2 combined pressures per scenario for discipline-enforcing skills

**Rationalization capture** — Step 3 (baseline test) adds: "Record exact verbatim rationalizations the agent uses to skip or shortcut. These become the rationalization table in the skill."

**Skill type testing strategies** — new section mapping skill types to test approaches:
- Discipline skills -> pressure scenarios + rationalization tables
- Technique skills -> application + variation + gap scenarios
- Pattern skills -> recognition + counter-example scenarios
- Reference skills -> retrieval + application scenarios

**Loophole closing** — Step 7 (iterate) adds: "For each new rationalization discovered, add an explicit counter to the skill. Build a red flags list if 3+ rationalizations target the same rule."

### Unchanged

The overall RED-GREEN-REFACTOR structure, task decomposition by behavioral capability, size budget checkpoints, concurrent edit protection — these already align with writing-skills.

## Section 5: Writing-Skills Summary Reference

New file: `references/writing-skills-summary.md` (~50 lines). Distillation of `superpowers:writing-skills` injected as Phase 3 context. Not the full 656-line skill.

Contents:

1. **The Iron Law** — no skill without a failing test first. Write skill before testing -> delete, start over. Applies to new skills AND edits.
2. **RED-GREEN-REFACTOR mapping** — baseline fails without skill (RED) -> write minimal skill addressing specific violations (GREEN) -> close loopholes while maintaining compliance (REFACTOR)
3. **CSO principles** — description = when to use, NOT what the skill does. Never summarize workflow in description. Trigger-oriented, third person, "Use when..." prefix.
4. **Rationalization bulletproofing** — close every loophole explicitly, address "spirit vs letter" arguments, build rationalization tables from baseline testing, create red flags lists.
5. **Token efficiency** — move details to tool help, use cross-references, compress examples, eliminate redundancy.
6. **Generalize over overfit** — skills must work across millions of future invocations, not just test examples. Explore broader metaphors and patterns rather than fiddly narrow changes.

## Section 6: Phase 4 and Phase 5 Adjustments

### Phase 4 (Validate)

Consistency-tester unchanged. Entry point adjustment:

- Phase 3 (skill-creator) exits with handoff summary
- Skill-bench reads final skill path from summary
- Phase 4 reads skill draft from that path

**Eval-to-test-case bridge:** Skill-bench offers to convert skill-creator's `evals.json` cases into multirun test cases if the user doesn't already have `.skillbench/test-cases/{skill-name}.json`.

### Phase 5 (Finalize)

Description optimization removed (now in Phase 3). CSO check in lint becomes verification-only — confirms description is trigger-oriented and non-overlapping, but doesn't re-run the optimization loop unless the skill changed during Phase 4.

**New lint check:** eval coverage — verify `evals.json` exists and covers the skill's main behavioral capabilities from the plan. Warning level (not blocking).

Updated lint checklist:

| Check | Level | Change |
|-------|-------|--------|
| Frontmatter validation | Blocking | Unchanged |
| Content checks | Blocking | Unchanged |
| Reference validation | Blocking | Unchanged |
| Description quality (CSO) | Warning | Verification-only, not optimization |
| Token efficiency | Warning | Unchanged |
| Flowchart review | Info | Unchanged |
| Eval coverage | Warning | **New** |

## Section 7: File Changes

### Modified

| File | Change |
|------|--------|
| `skills/skill-bench/SKILL.md` | Phase 3 rewritten: skill-creator handoff replaces subagent-driven-development. Dependency check adds skill-creator. Phase 5 lint adds eval coverage, CSO becomes verification-only. |
| `skills/skill-bench/references/skill-authoring-plan-template.md` | Pressure scenario design, rationalization capture, skill type testing strategies, loophole closing. |
| `skills/skill-bench/references/anti-patterns.md` | Add eval coverage lint check. |
| `CLAUDE.md` | Update Phase 3 description, add skill-creator to dependencies, update artifact table. |
| `README.md` | Add skill-creator to prerequisites, update components table, update Phase 3 description. |

### New

| File | Purpose |
|------|---------|
| `skills/skill-bench/references/writing-skills-summary.md` | ~50-line writing-skills distillation for Phase 3 context injection. |

### Unchanged

| File | Why |
|------|-----|
| `agents/consistency-tester.md` | Phase 4 unchanged. |
| `agents/skill-refiner.md` | Phase 4 unchanged. |
| `agents/skill-tester.md` | Still used by consistency-tester. No longer used in Phase 3. |
| `agents/skill-explorer.md` | Read-only scanner. |
| `docs/quickstart.md` | Full workflow quick reference. |
