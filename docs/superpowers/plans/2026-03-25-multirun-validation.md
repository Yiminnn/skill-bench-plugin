# Multirun Validation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Phase 4 (Validate) to skill-bench — multirun testing with consistency analysis, user judgment, failure pattern analysis, skill refinement, and re-run loop.

**Architecture:** New `consistency-tester` agent orchestrates the full validation loop by spawning `skill-tester` repeatedly. SKILL.md gets ~30 lines of Phase 4 instructions. Existing Phase 4 (Finalize) renumbers to Phase 5. Supporting docs (CLAUDE.md, README.md, plugin.json) update to match.

**Tech Stack:** Markdown (SKILL.md, agent .md files with YAML frontmatter), JSON (plugin.json)

---

### Task 1: Create consistency-tester agent

**Files:**
- Create: `agents/consistency-tester.md`

This is the core deliverable — the Opus agent that orchestrates the entire multirun validation loop.

- [ ] **Step 1: Create the agent file with frontmatter**

Create `agents/consistency-tester.md` with the following content:

```markdown
---
name: consistency-tester
description: "Use when validating a skill draft through multirun testing. Takes a test case library, runs the skill-tester agent N times per case, summarizes consistency across runs, collects user pass/fail judgments with annotations, analyzes failure patterns across cases, proposes targeted skill edits, and manages the re-run loop until the user declares validation complete."
model: opus
tools: Read, Write, Edit, Glob, Grep, Agent, Bash
---

# Consistency Tester

You validate a skill draft by running it multiple times against real test cases, analyzing consistency, collecting user feedback, and refining the skill until the user is satisfied.

## What You Receive

1. **Skill draft path** — path to the SKILL.md being validated (and its references/ directory)
2. **Test case library path** — path to `.skillbench/test-cases/{skill-name}.json`
3. **Test history directory** — path to `.skillbench/test-history/{skill-name}/`

## Test Case Library Format

```json
{
  "run_count": 5,
  "cases": [
    {
      "name": "case-name",
      "prompt": "The input prompt for this test case",
      "reference_files": ["path/to/file.pdf"],
      "description": "What this case tests"
    }
  ]
}
```

## The Validation Loop

Execute this loop until the user declares validation complete:

### Step 1: Run Collection

For each case in the library:

1. Read the current SKILL.md draft and its reference files
2. For each run (1 to `run_count`):
   - Use the Agent tool to spawn the **skill-tester** agent with:
     - **Skill content:** the full SKILL.md + reference files
     - **Sample input:** the case's `prompt`
     - **Context files:** the case's `reference_files` (if any)
   - Save the skill-tester's full output to `{test-history-dir}/{case-name}-round-{r}-run-{n}.json` where `r` is the current validation round and `n` is the run number
3. Runs are sequential — do not spawn multiple skill-tester agents in parallel

### Step 2: Consistency Summary

For each case, compare all N runs and produce this report:

```
## Case: {case-name} ({run_count} runs)

### Consistent Across Runs
[List elements that were identical or functionally equivalent in ALL runs]

### Variations
[List elements that differed between runs, noting which runs diverged and how]

### Run Summaries
| Run | Status | Key Differences |
|-----|--------|-----------------|
| 1   | —      | baseline         |
| 2   | —      | [how it differs from run 1] |
| ...
```

The Status column stays blank (`—`) until the user provides judgment.

Present the summary to the user. The full simulated outputs are available in test-history if the user wants to inspect individual runs.

### Step 3: User Judgment

For each case, ask the user:

> "Review the runs for `{case-name}`. Mark each run as pass or fail, with a note on what's wrong for failures."

The user responds in natural language. Parse their response into structured judgments:

```json
{
  "case": "{case-name}",
  "round": 1,
  "judgments": [
    { "run": 1, "status": "pass", "note": null },
    { "run": 2, "status": "fail", "note": "user's annotation text" }
  ]
}
```

Save to `{test-history-dir}/{case-name}-round-{r}-judgment.json`.

Confirm your understanding of the judgments with the user before proceeding. Do NOT second-guess user judgments — the user is the domain expert.

If all runs across all cases passed, skip to Step 6.

### Step 4: Failure Analysis

Collect all failure annotations across all cases and group by root cause:

```
## Failure Pattern Analysis

### Pattern 1: [Pattern Name] ({count} failures across {case_count} cases)
- {case-name}: run {n} ({user annotation})
- {case-name}: run {n} ({user annotation})
- Root cause: [your analysis of why these failures share a common cause]

### Proposed Skill Edits
1. [Specific edit tied to Pattern 1]
2. [Specific edit tied to Pattern 2]
```

Present the analysis and proposed edits to the user. The user approves, modifies, or rejects each proposed edit.

### Step 5: Apply Refinements

For each approved edit:

1. Read the target file (SKILL.md or reference file)
2. Compute SHA256 hash and compare to last-known hash
3. If hash differs from expected: show what changed, ask overwrite/incorporate/keep
4. Apply the edit using the Edit tool
5. Update the last-known hash
6. Commit: `fix: address multirun failure pattern — {description}`

### Step 6: Re-run Recommendation

Recommend which cases to re-run:

```
## Re-run Recommendation

- {case-name}: RE-RUN ({reason — e.g., "2 failures linked to Pattern 1, now fixed"})
- {case-name}: SKIP ({reason — e.g., "all 5 passed"})

Re-run all, just the recommended ones, or pick your own?
```

Recommendation logic:
- All runs passed → SKIP
- Failures matched a pattern that was fixed → RE-RUN
- Failures ambiguous or unfixed → RE-RUN

User decides: all, recommended, or custom selection.

If user selects cases to re-run, increment the round number and return to Step 1 for those cases only. Previous round results are preserved.

If user declares validation complete, report final status and exit.

## Important

- Never modify the skill without user approval for the proposed edits
- Runs must be sequential to avoid context interference between skill-tester invocations
- The user is the domain expert — their pass/fail judgment is final
- Keep summaries focused on differences, not repetition — the user needs to see what varies
- Previous round results are always preserved, never overwritten
```

- [ ] **Step 2: Verify the agent file**

Run: `cat -n agents/consistency-tester.md | head -5`
Expected: frontmatter with `name: consistency-tester`, `model: opus`

Run: `wc -l agents/consistency-tester.md`
Expected: approximately 130-140 lines

- [ ] **Step 3: Commit**

```bash
git add agents/consistency-tester.md
git commit -m "feat: add consistency-tester agent for multirun validation"
```

---

### Task 2: Add Phase 4 (Validate) to SKILL.md

**Files:**
- Modify: `skills/skill-bench/SKILL.md:1-151`

Two changes: (1) insert Phase 4 between current Phase 3 and Phase 4, (2) renumber current Phase 4 → Phase 5, (3) update the overview and description.

- [ ] **Step 1: Update frontmatter description**

Change the description in SKILL.md frontmatter from:

```yaml
description: "Use when authoring new Claude Code skills or refining existing skill prompts. Guides a structured workflow: design via brainstorming, plan the skill structure, build with TDD pressure testing, and finalize with validation."
```

to:

```yaml
description: "Use when authoring new Claude Code skills or refining existing skill prompts. Guides a structured workflow: design via brainstorming, plan the skill structure, build with TDD pressure testing, validate with multirun consistency testing, and finalize."
```

- [ ] **Step 2: Update overview paragraph**

Change line 10 from:

```
You guide the user through four phases: designing the skill (via brainstorming), planning the structure, building and testing with TDD (simulated execution + pressure testing), and finalizing with validation. You invoke superpowers skills at phase boundaries and provide skill-authoring context.
```

to:

```
You guide the user through five phases: designing the skill (via brainstorming), planning the structure, building and testing with TDD (simulated execution + pressure testing), validating with multirun consistency testing, and finalizing. You invoke superpowers skills at phase boundaries and provide skill-authoring context.
```

- [ ] **Step 3: Insert Phase 4 (Validate) before current Phase 4**

Insert the following content between the Phase 3 exit gate (line 105) and the current `## Phase 4: Finalize` heading (line 107):

```markdown

## Phase 4: Validate

**Multirun consistency testing** — run the skill draft against real test cases multiple times, compare outputs, collect user judgment, and refine.

### Setup

1. **Check for test case library** — Look for `.skillbench/test-cases/{skill-name}.json`
   - If it exists, read it and confirm the cases with the user
   - If it doesn't exist, prompt the user to create it:
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
```

- [ ] **Step 4: Renumber Phase 4 → Phase 5**

Change `## Phase 4: Finalize` to `## Phase 5: Finalize`.

- [ ] **Step 5: Verify SKILL.md**

Run: `grep -n "## Phase" skills/skill-bench/SKILL.md`
Expected output showing 5 phases:
```
Phase 1: Design
Phase 2: Plan
Phase 3: Build & Test
Phase 4: Validate
Phase 5: Finalize
```

Run: `wc -l skills/skill-bench/SKILL.md`
Expected: approximately 180-185 lines (was 151, added ~30)

- [ ] **Step 6: Commit**

```bash
git add skills/skill-bench/SKILL.md
git commit -m "feat: add Phase 4 (Validate) to skill-bench workflow"
```

---

### Task 3: Bump plugin version

**Files:**
- Modify: `.claude-plugin/plugin.json:4`

- [ ] **Step 1: Update version**

Change `"version": "0.2.0"` to `"version": "0.3.0"` in `.claude-plugin/plugin.json`.

- [ ] **Step 2: Verify**

Run: `cat .claude-plugin/plugin.json | grep version`
Expected: `"version": "0.3.0"`

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "chore: bump version to 0.3.0"
```

---

### Task 4: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md:1-63`

Three changes: (1) update "4-phase" to "5-phase", (2) add Phase 4 to the mapping table and add consistency-tester to architecture, (3) update runtime artifacts.

- [ ] **Step 1: Update "4-phase" to "5-phase"**

Change line 7 from:

```
A Claude Code plugin for interactive skill authoring. It provides a `skill-bench` skill (4-phase workflow: Design → Plan → Build & Test → Finalize) plus two companion agents (`skill-tester`, `skill-explorer`).
```

to:

```
A Claude Code plugin for interactive skill authoring. It provides a `skill-bench` skill (5-phase workflow: Design → Plan → Build & Test → Validate → Finalize) plus three companion agents (`skill-tester`, `consistency-tester`, `skill-explorer`).
```

- [ ] **Step 2: Add consistency-tester to architecture tree**

Change the architecture tree from:

```
agents/
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

to:

```
agents/
  consistency-tester.md      # Opus agent: multirun validation — runs skill-tester N times, compares, collects judgment, refines
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

- [ ] **Step 3: Add Phase 4 to mapping table**

Change the phase mapping table from:

```
| 3. Build & Test | `superpowers:subagent-driven-development` | Execute tasks with spec + behavioral review |
| 4. Finalize | (skill-bench native) | Lint, CSO check, promote |
```

to:

```
| 3. Build & Test | `superpowers:subagent-driven-development` | Execute tasks with spec + behavioral review |
| 4. Validate | (skill-bench native via consistency-tester) | Multirun testing, user judgment, pattern analysis, refinement |
| 5. Finalize | (skill-bench native) | Lint, CSO check, promote |
```

- [ ] **Step 4: Update runtime artifacts**

Add after the `.skillbench/test-history/{skill-name}/` line:

```
- `.skillbench/test-cases/{skill-name}.json` — test case library for multirun validation
```

- [ ] **Step 5: Update agent model list**

Change the last line from:

```
- The skill-tester agent uses model `opus`; skill-explorer uses `haiku`
```

to:

```
- The skill-tester and consistency-tester agents use model `opus`; skill-explorer uses `haiku`
```

- [ ] **Step 6: Verify**

Run: `grep -c "consistency-tester" CLAUDE.md`
Expected: 3 (architecture tree, phase mapping, model list)

Run: `grep "5-phase" CLAUDE.md`
Expected: matches the updated description line

- [ ] **Step 7: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md for multirun validation phase"
```

---

### Task 5: Update README.md

**Files:**
- Modify: `README.md:1-98`

Three changes: (1) add Phase 4 to workflow description, (2) add Phase 4 to the phase-to-skill table, (3) add test case library to artifact locations, (4) add consistency-tester to components table.

- [ ] **Step 1: Add Phase 4 to the workflow list**

Change the 4-phase list:

```markdown
1. **Design** — Brainstorm what your skill should do, explore approaches, produce a design spec
2. **Plan** — Generate a structured implementation plan with TDD-style tasks
3. **Build & Test** — Execute the plan with simulated testing and pressure testing
4. **Finalize** — Lint, validate references, and promote to your target location
```

to:

```markdown
1. **Design** — Brainstorm what your skill should do, explore approaches, produce a design spec
2. **Plan** — Generate a structured implementation plan with TDD-style tasks
3. **Build & Test** — Execute the plan with simulated testing and pressure testing
4. **Validate** — Run the skill against real test cases multiple times, compare consistency, refine based on user feedback
5. **Finalize** — Lint, validate references, and promote to your target location
```

- [ ] **Step 2: Add Phase 4 to the phase table**

Change:

```markdown
| Build & Test | `superpowers:subagent-driven-development` | Execute with spec review + behavioral testing |
| Finalize | skill-bench native | Lint pass + promotion |
```

to:

```markdown
| Build & Test | `superpowers:subagent-driven-development` | Execute with spec review + behavioral testing |
| Validate | skill-bench native (consistency-tester agent) | Multirun consistency testing + user-guided refinement |
| Finalize | skill-bench native | Lint pass + promotion |
```

- [ ] **Step 3: Add consistency-tester to components table**

Change:

```markdown
| `skill-tester` | Agent | Simulated skill execution with structured evaluation |
| `skill-explorer` | Agent | Browse and summarize existing skill drafts |
```

to:

```markdown
| `skill-tester` | Agent | Simulated skill execution with structured evaluation |
| `consistency-tester` | Agent | Multirun validation — consistency analysis, user judgment, pattern-based refinement |
| `skill-explorer` | Agent | Browse and summarize existing skill drafts |
```

- [ ] **Step 4: Add test case library to artifact locations**

Add a row to the artifact locations table:

```markdown
| `.skillbench/test-cases/` | Test case libraries for multirun validation | Yes |
```

- [ ] **Step 5: Update "four phases" text**

Change line 27:

```
Skill Bench guides you through creating Claude Code skills in four phases:
```

to:

```
Skill Bench guides you through creating Claude Code skills in five phases:
```

- [ ] **Step 6: Verify**

Run: `grep -c "Validate" README.md`
Expected: at least 2 (workflow list + phase table)

Run: `grep "five phases" README.md`
Expected: matches updated text

- [ ] **Step 7: Commit**

```bash
git add README.md
git commit -m "docs: update README for multirun validation phase"
```
