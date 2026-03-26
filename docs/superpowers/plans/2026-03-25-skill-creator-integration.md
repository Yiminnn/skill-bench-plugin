# Skill-Creator Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Integrate Anthropic's official `skill-creator` as the primary build-and-test engine in Phase 3, and enrich the plan template with `writing-skills` TDD methodology.

**Architecture:** Skill-bench remains the orchestrator. Phase 3 hands off to skill-creator (installed from `anthropics/skills`) via structured context. Writing-skills methodology enriches plan templates (Phase 2) and is distilled as a lightweight reference for Phase 3 context injection.

**Tech Stack:** Markdown (SKILL.md with YAML frontmatter), Claude Code plugin system

---

## File Structure

| File | Action | Responsibility |
|------|--------|---------------|
| `skills/skill-bench/references/writing-skills-summary.md` | Create | ~50-line distillation of writing-skills principles for Phase 3 context |
| `skills/skill-bench/references/skill-authoring-plan-template.md` | Modify | Add pressure scenarios, rationalization capture, skill types, loophole closing |
| `skills/skill-bench/references/anti-patterns.md` | Modify | Add eval coverage lint check |
| `skills/skill-bench/SKILL.md` | Modify | Dependency check + Phase 3 rewrite + Phase 5 updates |
| `CLAUDE.md` | Modify | Phase 3 description, dependencies, artifact table |
| `README.md` | Modify | Prerequisites, components, Phase 3 description |

---

### Task 1: Create writing-skills-summary.md

**Files:**
- Create: `skills/skill-bench/references/writing-skills-summary.md`

- [ ] **Step 1: Write the file**

Create `skills/skill-bench/references/writing-skills-summary.md` with this content:

```markdown
# Writing-Skills Principles Summary

Lightweight distillation of `superpowers:writing-skills` for context injection during skill-creator handoff. Not the full skill — just the principles that matter at build time.

## The Iron Law

No skill without a failing test first. Write skill before testing → delete, start over. This applies to new skills AND edits to existing skills. No exceptions — not for "simple additions", not for "just adding a section", not for "documentation updates".

## RED-GREEN-REFACTOR for Skills

| Phase | What Happens |
|-------|-------------|
| **RED** | Run pressure scenario WITHOUT skill. Document exact behavior and rationalizations (verbatim). |
| **GREEN** | Write minimal skill addressing those specific violations. Run same scenario WITH skill — agent should comply. |
| **REFACTOR** | Agent found new rationalizations? Add explicit counters. Re-test until bulletproof. |

## CSO (Claude Search Optimization)

**Description = when to use, NOT what the skill does.**

- Start with "Use when..." — trigger conditions only
- Never summarize the skill's workflow in the description
- Third person, specific scenarios, not vague categories
- Descriptions that summarize workflow create a shortcut Claude will take — the skill body becomes documentation Claude skips

## Rationalization Bulletproofing

- Close every loophole explicitly — don't just state the rule, forbid specific workarounds
- Address "spirit vs letter" arguments early: "Violating the letter of the rules is violating the spirit of the rules"
- Build rationalization tables from baseline testing — every excuse agents make goes in the table
- Create red flags lists for easy self-checking when under pressure

## Token Efficiency

- Move implementation details to tool `--help` references
- Use cross-references to other skills instead of repeating content
- Compress examples: one excellent example beats many mediocre ones
- Eliminate redundancy — don't repeat what's in referenced skills

## Generalize Over Overfit

Skills must work across millions of future invocations, not just the test examples. When issues persist after 2+ fix attempts, explore broader metaphors and patterns rather than fiddly narrow changes. Avoid overfitting to specific test case wording.
```

- [ ] **Step 2: Verify**

Read the file. Confirm ~50 lines, covers all 6 principles from the design spec.

- [ ] **Step 3: Commit**

```bash
git add skills/skill-bench/references/writing-skills-summary.md
git commit -m "feat: add writing-skills summary reference for Phase 3 context injection"
```

---

### Task 2: Enhance plan template with writing-skills methodology

**Files:**
- Modify: `skills/skill-bench/references/skill-authoring-plan-template.md`

- [ ] **Step 1: Enhance Step 2 pressure scenario guidance**

In the Task Step Template section, replace the current Step 2 content:

```markdown
- [ ] **Step 2: Write baseline pressure scenario**

A prompt that tests this behavior WITHOUT the skill. Must:
- Create realistic pressure (time, complexity, sunk cost)
- NOT mention the skill or its rules
- Be specific enough to observe compliance/violation

[Show exact prompt text]
```

With:

```markdown
- [ ] **Step 2: Write baseline pressure scenario**

A prompt that tests this behavior WITHOUT the skill. Must:
- Combine at least 2 pressure types for discipline-enforcing skills:
  - **Time:** "the user is waiting", "this is urgent"
  - **Sunk cost:** "you've already written most of the code"
  - **Authority:** "the senior engineer said to skip this"
  - **Exhaustion:** "this is task 7 of 12"
- NOT mention the skill or its rules
- Be specific enough to observe compliance/violation

[Show exact prompt text]
```

- [ ] **Step 2: Enhance Step 3 with rationalization capture**

In the Task Step Template section, replace the current Step 3 content:

```markdown
- [ ] **Step 3: Run baseline test (RED)**

Spawn subagent WITHOUT skill. Give it the pressure scenario. Document:

Use the Agent tool to spawn a fresh subagent. Do NOT load the skill being tested — the point is to observe natural behavior without it.
- What the agent did
- Rationalizations used (verbatim)

Expected: Agent violates or skips the target behavior.
```

With:

```markdown
- [ ] **Step 3: Run baseline test (RED)**

Spawn subagent WITHOUT skill. Give it the pressure scenario. Document:

Use the Agent tool to spawn a fresh subagent. Do NOT load the skill being tested — the point is to observe natural behavior without it.
- What the agent did
- Rationalizations used (**record exact verbatim quotes** — these become the rationalization table in the skill)
- Which pressure types triggered which violations

Expected: Agent violates or skips the target behavior.
```

- [ ] **Step 3: Enhance Step 7 with loophole closing**

In the Task Step Template section, replace the current Step 7:

```markdown
- [ ] **Step 7: Iterate if needed**

If Step 5 or 6 failed: identify gap, edit skill section, re-run failing test.
```

With:

```markdown
- [ ] **Step 7: Iterate if needed**

If Step 5 or 6 failed: identify gap, edit skill section, re-run failing test.
- For each new rationalization discovered, add an explicit counter to the skill
- If 3+ rationalizations target the same rule, build a red flags list
- Close loopholes explicitly — don't just restate the rule, forbid specific workarounds
```

- [ ] **Step 4: Add skill type testing strategies section**

After the "Task Decomposition" section (before "Size Budget Checkpoints"), add:

```markdown
## Skill Type Testing Strategies

Different skill types need different pressure scenarios and test approaches:

| Skill Type | Examples | Test With | Success Criteria |
|-----------|----------|-----------|-----------------|
| **Discipline** | TDD, verification gates | Pressure scenarios with combined pressures + rationalization tables | Agent follows rule under maximum pressure |
| **Technique** | Step-by-step guides, workflows | Application + variation + missing info scenarios | Agent applies technique correctly to new scenarios |
| **Pattern** | Mental models, design patterns | Recognition + application + counter-example scenarios | Agent identifies when/how to apply (and when NOT to) |
| **Reference** | API docs, format specs | Retrieval + application + gap scenarios | Agent finds and correctly uses reference info |

When writing plan tasks, match pressure scenario design to the skill type. Discipline skills need heavy rationalization testing. Reference skills need retrieval accuracy testing.
```

- [ ] **Step 5: Update plan header template**

Replace the current Plan Header section:

```markdown
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
```

With:

```markdown
## Plan Header

Use this header for skill-authoring plans:

```
# [Skill Name] Skill Implementation Plan

> **For agentic workers:** This plan is executed by the `skill-creator` skill
> during Phase 3 of skill-bench. Skill-creator handles the eval loop:
> write → test (with/without baseline) → grade → viewer → iterate → optimize description.

**Goal:** Create the {skill-name} skill that [one sentence]

**Architecture:** Skill-bench orchestrated. SKILL.md + references/ for heavy content.
Build and eval via skill-creator. Multirun validation via consistency-tester (Phase 4).

**Tech Stack:** Markdown (SKILL.md with YAML frontmatter), Claude Code plugin system
```
```

- [ ] **Step 6: Verify**

Read the full file. Confirm:
- Step 2 lists 4 pressure types with examples
- Step 3 has bold "record exact verbatim quotes" instruction
- Step 7 has loophole closing with red flags threshold
- Skill type testing strategies table has 4 types
- Plan header references skill-creator, not subagent-driven-development

- [ ] **Step 7: Commit**

```bash
git add skills/skill-bench/references/skill-authoring-plan-template.md
git commit -m "feat: enhance plan template with writing-skills TDD methodology"
```

---

### Task 3: Add eval coverage lint check to anti-patterns

**Files:**
- Modify: `skills/skill-bench/references/anti-patterns.md`

- [ ] **Step 1: Update header reference**

Replace:

```markdown
Use this checklist during Phase 4 (Finalize) to catch common issues before promoting a skill draft.
```

With:

```markdown
Use this checklist during Phase 5 (Finalize) to catch common issues before promoting a skill draft.
```

- [ ] **Step 2: Add eval coverage section**

After the "Structural Issues" section, add:

```markdown
## Eval Coverage

- [ ] **No evals.json** — Skills built via the full workflow should have an `evals.json` in `.skillbench/evals/{skill-name}/` with assertions covering the skill's main behavioral capabilities. (Warning — not blocking for express-path skills.)
- [ ] **Evals don't cover plan capabilities** — Cross-reference eval cases against the implementation plan's behavioral capabilities. Each major capability should have at least one eval case.
```

- [ ] **Step 3: Verify**

Read the file. Confirm Phase 5 reference is correct and eval coverage section exists with 2 check items.

- [ ] **Step 4: Commit**

```bash
git add skills/skill-bench/references/anti-patterns.md
git commit -m "feat: add eval coverage lint check to anti-patterns"
```

---

### Task 4: Rewrite SKILL.md — Dependency Check, Phase 3, Phase 5

**Files:**
- Modify: `skills/skill-bench/SKILL.md`

This is the core integration task. Three sections change in the same file.

- [ ] **Step 1: Expand Dependency Check**

Replace lines 12-19 (the entire Dependency Check section):

```markdown
## Dependency Check

Before starting any phase, verify the superpowers plugin is installed:

1. Check if `superpowers:brainstorming` skill is available via the Skill tool
2. If not available, run: `claude plugin install claude-plugins-official/superpowers`
3. Verify installation succeeded
4. If installation fails, tell the user: "Skill Bench requires the superpowers plugin. Install manually: `claude plugin install claude-plugins-official/superpowers`" — then stop
```

With:

```markdown
## Dependency Check

Before starting, verify required dependencies:

### Superpowers (Phases 1-2)

1. Check if `superpowers:brainstorming` skill is available via the Skill tool
2. If not available, run: `claude plugin install claude-plugins-official/superpowers`
3. Verify installation succeeded
4. If fails: "Skill Bench requires the superpowers plugin. Install manually: `claude plugin install claude-plugins-official/superpowers`" — stop

### Skill-Creator (Phase 3)

1. Check if the `skill-creator` skill is available
2. If not available, install from `anthropics/skills`:
   ```bash
   git clone --depth 1 --filter=blob:none --sparse https://github.com/anthropics/skills.git /tmp/anthropics-skills && \
   cd /tmp/anthropics-skills && git sparse-checkout set skills/skill-creator && \
   cp -r skills/skill-creator ~/.claude/skills/skill-creator && \
   rm -rf /tmp/anthropics-skills
   ```
3. Verify `~/.claude/skills/skill-creator/SKILL.md` exists and internal agents are accessible
4. If fails: "Phase 3 requires skill-creator. Install from https://github.com/anthropics/skills" — stop
```

- [ ] **Step 2: Rewrite Phase 3**

Replace lines 56-105 (the entire Phase 3 section):

```markdown
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
```

With:

```markdown
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
```

- [ ] **Step 3: Update Phase 4 entry to support eval-to-test-case bridge**

In the Phase 4 Setup section, after the test case library check (the "If it doesn't exist" branch), add an alternative path. Replace:

```markdown
   - If it doesn't exist, prompt the user to create it:
     > "Create your test case library at `.skillbench/test-cases/{skill-name}.json`. Format:"
```

With:

```markdown
   - If it doesn't exist:
     - If `{evals_dir}/{skill-name}/evals.json` exists (from Phase 3), offer to convert: "I can seed your test case library from the Phase 3 evals. Convert?"
       - If yes: read `evals.json`, map each eval case to a test case (eval prompt → test prompt, eval assertions → test description), write to `.skillbench/test-cases/{skill-name}.json`
     - Otherwise, prompt the user to create it:
```

- [ ] **Step 4: Update Phase 5 CSO check**

In the Phase 5 Lint Pass, replace item 4:

```markdown
4. **Description quality (CSO):**
   - Description is trigger-oriented, not workflow-summarizing
   - Glob existing `**/SKILL.md` files, check for description overlap
   - Keywords present for discovery
```

With:

```markdown
4. **Description quality (CSO verification):**
   - Verify description is trigger-oriented, not workflow-summarizing (optimized in Phase 3)
   - Glob existing `**/SKILL.md` files, check for description overlap
   - If skill changed during Phase 4, re-run description optimization via skill-creator
```

- [ ] **Step 5: Add eval coverage to Phase 5 lint**

In the Phase 5 Lint Pass, after item 5 (Token efficiency) and before item 6 (Flowchart review), add:

```markdown
6. **Eval coverage:**
   - Verify `{evals_dir}/{skill-name}/evals.json` exists
   - Check that evals cover the skill's main behavioral capabilities from the plan
   - Warning if missing (not blocking — express-path skills may not have evals)
```

Then renumber the existing item 6 (Flowchart review) to item 7.

- [ ] **Step 6: Verify**

Read the full SKILL.md. Check:
- Dependency Check has two subsections (Superpowers, Skill-Creator)
- Phase 3 references skill-creator, not subagent-driven-development
- Config JSON includes `evals_dir`
- .gitignore includes `.skillbench/workspace/`
- Phase 3 creates 3 directories (draft, evals, workspace)
- Phase 3 handoff has 5 read steps + 7 override instructions
- Phase 4 has eval-to-test-case bridge
- Phase 5 item 4 says "CSO verification" and references Phase 3
- Phase 5 has eval coverage check (item 6) before flowchart review (item 7)
- Total line count stays under 200 lines

- [ ] **Step 7: Commit**

```bash
git add skills/skill-bench/SKILL.md
git commit -m "feat: integrate skill-creator as Phase 3 build-and-test engine"
```

---

### Task 5: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Update requires line**

Replace:

```markdown
**Requires:** [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin (auto-installed on first use).
```

With:

```markdown
**Requires:** [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin + [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill (both auto-installed on first use).
```

- [ ] **Step 2: Add writing-skills-summary.md to architecture tree**

In the architecture tree, after the `skill-authoring-plan-template.md` line, add:

```
    writing-skills-summary.md   # Writing-skills principles distillation (Phase 3 context)
```

- [ ] **Step 3: Update Phase 3 in workflow mapping table**

Replace the Phase 3 row:

```markdown
| 3. Build & Test | `superpowers:subagent-driven-development` | Execute tasks with spec + behavioral review |
```

With:

```markdown
| 3. Build & Test | `skill-creator` (from `anthropics/skills`) | Build skill from plan, eval with baseline comparison, grade, iterate, optimize description |
```

- [ ] **Step 4: Replace Reviewer Roles section**

Replace the entire "Reviewer Roles in Phase 3" section:

```markdown
### Reviewer Roles in Phase 3

- **Reviewer 1 (Spec Compliance):** Checks against design spec + `skill-format.md`
- **Reviewer 2 (Behavioral Testing):** Spawns `skill-tester` for simulated execution + pressure tests from writing-skills TDD
```

With:

```markdown
### Skill-Creator in Phase 3

Phase 3 delegates to Anthropic's official `skill-creator` skill with structured handoff context (design spec, plan, skill-format constraints, writing-skills principles). Skill-creator handles: write → eval (with-skill + baseline) → grade → viewer → iterate → description optimization.
```

- [ ] **Step 5: Update runtime artifacts**

Replace the runtime artifacts list:

```markdown
### Runtime Artifacts (created in user projects, not this repo)

- `.skillbench/config.json` — project config (drafts_dir, test_model, context_files)
- `.skillbench/specs/{skill-name}-design.md` — design specs from Phase 1
- `.skillbench/plans/{skill-name}-plan.md` — implementation plans from Phase 2
- `.skillbench/test-history/{skill-name}/` — simulated + pressure test results
- `.skillbench/test-cases/{skill-name}.json` — test case library for multirun validation
- `skills/drafts/` — default location for in-progress skill drafts
```

With:

```markdown
### Runtime Artifacts (created in user projects, not this repo)

- `.skillbench/config.json` — project config (drafts_dir, evals_dir, test_model, context_files)
- `.skillbench/specs/{skill-name}-design.md` — design specs from Phase 1
- `.skillbench/plans/{skill-name}-plan.md` — implementation plans from Phase 2
- `.skillbench/evals/{skill-name}/evals.json` — eval definitions from Phase 3 (git-tracked)
- `.skillbench/workspace/{skill-name}/` — skill-creator workspace: iterations, grading, benchmarks (gitignored)
- `.skillbench/test-history/{skill-name}/` — consistency-tester results + refinement records (gitignored)
- `.skillbench/test-cases/{skill-name}.json` — test case library for multirun validation
- `skills/drafts/` — default location for in-progress skill drafts
```

- [ ] **Step 6: Verify**

Read CLAUDE.md. Confirm: requires mentions both superpowers and skill-creator, architecture tree includes writing-skills-summary.md, Phase 3 row references skill-creator, reviewer roles section replaced, artifacts include evals and workspace.

- [ ] **Step 7: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md for skill-creator integration"
```

---

### Task 6: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update prerequisites**

Replace the prerequisites section:

```markdown
### Prerequisites

The full workflow (`skill-bench`) requires the [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin. It will be installed automatically on first use, or manually:

```bash
claude plugin install claude-plugins-official/superpowers
```

The express path (`skill-bench-express`) has no plugin dependencies.
```

With:

```markdown
### Prerequisites

The full workflow (`skill-bench`) requires:
- [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin (Phases 1-2)
- [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill (Phase 3)

Both are installed automatically on first use, or manually:

```bash
claude plugin install claude-plugins-official/superpowers
```

The express path (`skill-bench-express`) has no dependencies.
```

- [ ] **Step 2: Update full workflow phase descriptions**

Replace the Phase 3 line:

```markdown
3. **Build & Test** — Execute tasks with simulated + pressure testing via `superpowers:subagent-driven-development`
```

With:

```markdown
3. **Build & Test** — Build skill from plan, evaluate with baseline comparison, iterate via `skill-creator`
```

- [ ] **Step 3: Update components table**

Replace the skill-bench row:

```markdown
| `skill-bench` | Skill | — | Full 5-phase authoring workflow with superpowers orchestration |
```

With:

```markdown
| `skill-bench` | Skill | — | Full 5-phase authoring workflow with superpowers + skill-creator orchestration |
```

- [ ] **Step 4: Update project config JSON**

Replace the config JSON block:

```json
{
  "drafts_dir": "skills/drafts",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

With:

```json
{
  "drafts_dir": "skills/drafts",
  "evals_dir": ".skillbench/evals",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

And add to the config field descriptions after the `drafts_dir` line:

```markdown
- `evals_dir` — where eval definitions are stored (default: `.skillbench/evals`)
```

- [ ] **Step 5: Update artifact locations table**

Add two rows to the artifact locations table, after the `.skillbench/plans/` row:

```markdown
| `.skillbench/evals/` | Eval definitions from Phase 3 (skill-creator) | Yes |
| `.skillbench/workspace/` | Skill-creator workspace (iterations, grading, benchmarks) | No (gitignore) |
```

- [ ] **Step 6: Update shared infrastructure table**

Replace the `skill-tester` row in the shared infrastructure table:

```markdown
| `skill-tester` agent | Sample Run | Phase 3 (behavioral testing) |
```

With:

```markdown
| `skill-tester` agent | Sample Run | Phase 4 (via consistency-tester) |
```

- [ ] **Step 7: Verify**

Read README.md. Confirm: prerequisites list both dependencies, Phase 3 mentions skill-creator, config includes evals_dir, artifacts include evals and workspace, shared infrastructure table has updated skill-tester row.

- [ ] **Step 8: Commit**

```bash
git add README.md
git commit -m "docs: update README for skill-creator integration"
```

---

### Task 7: Final verification

**Files:**
- All modified files

- [ ] **Step 1: Cross-reference check**

Verify consistency across all files:
- SKILL.md Phase 3 references `references/writing-skills-summary.md` → confirm file exists
- SKILL.md Phase 5 references eval coverage → confirm anti-patterns.md has the check
- SKILL.md dependency check references `anthropics/skills` → confirm CLAUDE.md and README.md match
- Plan template header references `skill-creator` → confirm SKILL.md Phase 3 matches
- Config JSON format is consistent across SKILL.md, README.md

- [ ] **Step 2: Size check**

Read SKILL.md and count lines. Must stay under 200. If over, identify what can be moved to references.

- [ ] **Step 3: Grep for stale references**

Search all files for `subagent-driven-development` — should appear nowhere in the modified files (may still appear in the design spec, which is historical record — that's fine).

Search for `Reviewer 1` and `Reviewer 2` — should not appear in SKILL.md or CLAUDE.md (replaced by skill-creator handoff).

- [ ] **Step 4: Final commit (if any fixes needed)**

```bash
git add -A
git commit -m "fix: address cross-reference issues from skill-creator integration"
```
