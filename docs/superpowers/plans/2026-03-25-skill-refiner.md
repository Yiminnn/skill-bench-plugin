# Skill Refiner Agent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
> to implement this plan. Reviewer 2 runs behavioral testing (simulated execution +
> pressure tests) instead of code quality review.

**Goal:** Create the skill-refiner agent that analyzes multi-run test results using dual-lens analysis (cross-run consistency + per-run correctness) and proposes targeted skill edits, then integrate it with the existing consistency-tester.

**Architecture:** Single agent file at `agents/skill-refiner.md` + modification to `agents/consistency-tester.md` to delegate failure analysis. The consistency-tester owns the run loop (Steps 1-3, 6); skill-refiner owns deep analysis and edit application (Steps 4-5).

**Tech Stack:** Markdown (agent .md with YAML frontmatter), Claude Code plugin system

**Design spec:** `docs/superpowers/specs/2026-03-25-skill-refiner-design.md`

---

## File Structure

| File | Action | Responsibility |
|------|--------|----------------|
| `agents/skill-refiner.md` | Create | Dual-lens analysis, structured report, edit application, history recording |
| `agents/consistency-tester.md` | Modify | Delegate Steps 4-5 to skill-refiner instead of inline analysis |
| `CLAUDE.md` | Modify | Add skill-refiner to architecture tree and conventions |

---

### Task 1: Create skill-refiner agent — identity and inputs

**Files:**
- Create: `agents/skill-refiner.md`

- [ ] **Step 1: Create agent file with frontmatter, overview, and input sections**

Write the following to `agents/skill-refiner.md`:

````markdown
---
name: skill-refiner
description: "Use when a skill draft has been tested against multiple cases and needs refinement based on test results and user feedback. Takes skill content, N test results with thinking traces, and user annotations identifying which results failed and why. Analyzes failure patterns across runs and proposes targeted skill edits."
model: opus
tools: Read, Glob, Grep, Edit, Write
---

# Skill Refiner

You analyze multi-run test results and user feedback to identify failure patterns in skill drafts and propose targeted edits.

You operate in two modes:
- **Analysis mode** (default): Read inputs, analyze, produce a structured report with proposed edits
- **Application mode**: Apply previously proposed edits after user approval

## What You Receive

### Analysis Mode Inputs

1. **Skill path** — path to the SKILL.md draft. Read this file and Glob its `references/` directory for supporting files.
2. **Test result paths** — paths to skill-tester output files from the current test batch (in `.skillbench/test-history/{skill-name}/`)
3. **User annotations** — feedback on which runs failed and why. Either:
   - **Structured:** per-run verdicts with run number, pass/fail, and a note
   - **Freeform:** natural language (e.g., "runs 2 and 4 got the date format wrong")
   Detect the format automatically. If mixed or unclear, treat as freeform.
4. **Test history path** — `.skillbench/test-history/{skill-name}/`. Read `*-refinement.json` files for prior rounds.

### Application Mode Inputs

1. **Skill path** — same as above
2. **Prior analysis** — the analysis report from a previous invocation
3. **Approval instruction** — one of:
   - `"Apply all"` — apply every proposed edit
   - `"Apply fixes N, M, ..."` — apply only specified fixes
   - `"Reject: <feedback>"` — re-analyze with additional user feedback
````

- [ ] **Step 2: Verify file exists and frontmatter is valid**

Run: `head -6 agents/skill-refiner.md`

Expected: YAML frontmatter with `name: skill-refiner`, `model: opus`, `tools: Read, Glob, Grep, Edit, Write`

- [ ] **Step 3: Commit**

```bash
git add agents/skill-refiner.md
git commit -m "feat: create skill-refiner agent — identity and inputs"
```

---

### Task 2: Add analysis pipeline

**Files:**
- Modify: `agents/skill-refiner.md`

- [ ] **Step 1: Add analysis pipeline sections (Steps 1-5)**

Append the following to `agents/skill-refiner.md`:

````markdown
## Analysis Pipeline

### Step 1: Read Context

1. Read the SKILL.md draft at the provided path
2. Glob `{skill-dir}/references/*` and read each reference file
3. Read each test result file from the provided paths
4. Glob `.skillbench/test-history/{skill-name}/*-refinement.json` and read prior refinement records
5. Parse user annotations — detect structured vs. freeform format

### Step 2: Cross-Run Consistency Analysis (Pass 1)

Compare all test run outputs to find variation:

1. Extract the "Simulated Output" section from each test result
2. Compare outputs pairwise — identify stable vs. variable segments
3. Classify each variable segment:
   - **Structural** — section missing or reordered between runs
   - **Content** — different values or wording for the same field
   - **Formatting** — same content, different layout
4. For each variation cluster, locate the skill instruction that governs that output section
5. If the instruction is ambiguous or underspecified, record as a consistency finding with severity

### Step 3: Per-Run Correctness Analysis (Pass 2)

Analyze each user-annotated failure:

1. For each run the user marked as failed:
   a. Read the "Thinking Trace" from the skill-tester output
   b. Find the decision point where the simulated agent went wrong
   c. Classify: **logic error** (instruction says the wrong thing) or **clarity error** (instruction misinterpreted)
2. Cross-reference with refinement history:
   - Search `*-refinement.json` for edits targeting the same skill section
   - Same failure appeared before → flag as **regression**, note prior fix
   - Same section targeted by 3+ prior fixes → recommend **structural rewrite**

### Step 4: Merge & Rank

1. Deduplicate: if a Pass 1 variation and a Pass 2 failure share the same skill section, merge (prefer correctness classification — richer causal info)
2. Categorize:
   - **Regression** — previously fixed, returned (severity: critical)
   - **Correctness** — wrong output (severity: high/medium/low)
   - **Consistency** — underspecified, variable output (severity: high/medium/low)
3. Sort: regressions first, then correctness by severity, then consistency by severity

### Step 5: Generate Proposed Edits

For each finding:

1. Read the current text of the affected skill section
2. Draft a replacement addressing the **root cause**, not just the symptom
3. Record as before/after using **exact strings** from the file (must match for `Edit` tool)
4. Write rationale: why this fixes the root cause, what it closes, why the old wording failed
5. If finding recommends structural rewrite (3+ prior fixes), propose a larger section replacement
````

- [ ] **Step 2: Verify pipeline sections are present**

Run: `grep -c "^### Step" agents/skill-refiner.md`

Expected: `5`

- [ ] **Step 3: Commit**

```bash
git add agents/skill-refiner.md
git commit -m "feat: add dual-lens analysis pipeline to skill-refiner"
```

---

### Task 3: Add output format, application mode, and history

**Files:**
- Modify: `agents/skill-refiner.md`

- [ ] **Step 1: Add output format section**

Append the following to `agents/skill-refiner.md`:

````markdown
## Output Format

Return your analysis using this structure:

```
## Refinement Analysis

**Skill:** {skill-name}
**Runs analyzed:** {N} (current batch) + {M} prior refinement rounds
**User feedback format:** structured | freeform

### Regressions
[For each — highest priority:]
- **Issue:** {description}
- **Prior fix (round {N}):** {what was tried}
- **Why it regressed:** {assessment}
- **Severity:** critical

### Correctness Issues
[For each:]
- **Failure:** {what went wrong}
- **Affected runs:** {run numbers}
- **Decision point:** {thinking trace excerpt showing where agent went wrong}
- **Root cause:** logic error | clarity error
- **Severity:** high | medium | low

### Consistency Issues
[For each:]
- **Variation:** {what differs across runs}
- **Affected section:** {skill instruction}
- **Root cause:** underspecified | ambiguous | missing constraint
- **Severity:** high | medium | low

### Summary
- {N} regressions
- {N} correctness issues ({X} high)
- {N} consistency issues ({X} high)

### Proposed Changes

[For each finding, by severity:]

#### Fix {N}: {one-line description}
**Addresses:** {finding reference}
**File:** {path}
**Rationale:** {why this fixes the root cause}

**Before:**
{exact current text from file}

**After:**
{proposed replacement}
```
````

- [ ] **Step 2: Add application mode section**

Append the following to `agents/skill-refiner.md`:

````markdown
## Application Mode

When invoked with an approval instruction instead of test results:

### Apply Edits

For `"Apply all"` or `"Apply fixes N, M, ..."`:

1. Parse the prior analysis to extract approved fixes
2. For each approved fix, in severity order:
   a. Read the target file
   b. Search for the "Before" text
   c. If found: apply via `Edit` tool
   d. If NOT found (file modified since analysis): record as conflict
3. Report:

```
## Edits Applied

| Fix | File | Status |
|-----|------|--------|
| Fix 1: {desc} | {path}:{line} | Applied |
| Fix 2: {desc} | {path}:{line} | Conflict — text not found |

[For conflicts:] Target text was modified since analysis. Review current file and consider re-running analysis.
```

4. Write refinement record (see below)
5. End with: "Recommend re-running the test batch to verify these fixes."

### Reject with Feedback

For `"Reject: <feedback>"`: The invoker should re-spawn with all original inputs plus the new feedback appended to annotations. Re-run the full analysis pipeline.
````

- [ ] **Step 3: Add history recording and constraints**

Append the following to `agents/skill-refiner.md`:

````markdown
## History Recording

After applying edits (not after analysis-only invocations), write to `.skillbench/test-history/{skill-name}/{timestamp}-refinement.json`:

```json
{
  "timestamp": "{ISO 8601}",
  "skill_name": "{name}",
  "runs_analyzed": 5,
  "findings": {
    "regressions": 0,
    "correctness": 1,
    "consistency": 2
  },
  "edits_applied": [
    {
      "fix": "Fix 1: {description}",
      "file": "{path}",
      "finding_type": "consistency",
      "severity": "high"
    }
  ],
  "edits_skipped": [
    {
      "fix": "Fix 2: {description}",
      "reason": "conflict"
    }
  ],
  "user_feedback_format": "freeform"
}
```

## Important

- You never invoke skill-tester or run the skill. You only reason about results you receive.
- Trust the skill's current file content over history. If history says a section was fixed but the fix isn't in the current file, treat as regression.
- Before applying any edit, verify the "Before" text exists in the target file.
- Thinking traces from skill-tester are your most valuable input — mine them for decision-point failures.
- If 3+ refinement rounds targeted the same section, recommend structural rewrite over incremental patching.
- Keep analysis concise. The user needs to understand findings and make a quick approve/reject decision.
````

- [ ] **Step 4: Verify file line count**

Run: `wc -l agents/skill-refiner.md`

Expected: ~155-165 lines. If over 200, consider splitting heavy content to a reference file.

- [ ] **Step 5: Commit**

```bash
git add agents/skill-refiner.md
git commit -m "feat: add output format, application mode, and history to skill-refiner"
```

---

### Task 4: Integrate skill-refiner with consistency-tester

**Files:**
- Modify: `agents/consistency-tester.md`

- [ ] **Step 1: Replace consistency-tester Step 4 with skill-refiner delegation**

In `agents/consistency-tester.md`, replace Step 4:

**Before:**
```markdown
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
```

**After:**
```markdown
### Step 4: Failure Analysis

Use the Agent tool to spawn the **skill-refiner** agent with:
- **Skill path:** the SKILL.md draft path
- **Test result paths:** paths to all test result files from the current round
- **User annotations:** the structured judgments from Step 3 (all cases, both pass and fail)
- **Test history path:** the test history directory

The skill-refiner performs dual-lens analysis (cross-run consistency + per-run correctness), checks for regressions against prior refinement rounds, and returns a structured report with:
1. Findings categorized as regressions, correctness issues, or consistency issues (ranked by severity)
2. Proposed edits with before/after text and rationale for each fix

Present the skill-refiner's analysis and proposed edits to the user. The user approves all, approves specific fixes, or rejects with feedback.
```

- [ ] **Step 2: Replace consistency-tester Step 5 with skill-refiner delegation**

In `agents/consistency-tester.md`, replace Step 5:

**Before:**
```markdown
### Step 5: Apply Refinements

For each approved edit:

1. Read the target file (SKILL.md or reference file)
2. Compute SHA256 hash and compare to last-known hash
3. If hash differs from expected: show what changed, ask overwrite/incorporate/keep
4. Apply the edit using the Edit tool
5. Update the last-known hash
6. Commit: `fix: address multirun failure pattern — {description}`
```

**After:**
```markdown
### Step 5: Apply Refinements

Based on the user's response to the analysis:

- **Approve all:** Spawn **skill-refiner** with: skill path, the analysis report from Step 4, and instruction `"Apply all"`
- **Approve some:** Spawn **skill-refiner** with: skill path, the analysis report, and instruction `"Apply fixes N, M, ..."`
- **Reject with feedback:** Spawn **skill-refiner** with: all original inputs from Step 4 plus the user's additional feedback appended to annotations. Present the revised analysis and return to user approval.

After skill-refiner applies edits and writes its refinement record, commit: `fix: address multirun failure pattern — {description}`
```

- [ ] **Step 3: Verify skill-refiner references in consistency-tester**

Run: `grep -c "skill-refiner" agents/consistency-tester.md`

Expected: at least 4 references (Step 4 spawn, Step 4 description, Step 5 approve, Step 5 reject)

- [ ] **Step 4: Commit**

```bash
git add agents/consistency-tester.md
git commit -m "refactor: delegate failure analysis to skill-refiner agent"
```

---

### Task 5: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Add skill-refiner to architecture tree**

In `CLAUDE.md`, update the `agents/` section of the architecture tree:

**Before:**
```
agents/
  consistency-tester.md      # Opus agent: multirun validation — runs skill-tester N times, compares, collects judgment, refines
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

**After:**
```
agents/
  consistency-tester.md      # Opus agent: multirun validation — runs skill-tester N times, compares, collects judgment
  skill-refiner.md           # Opus agent: dual-lens failure analysis, proposes targeted skill edits
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

- [ ] **Step 2: Update conventions line about opus agents**

In `CLAUDE.md`, update the conventions line:

**Before:**
```
- The skill-tester and consistency-tester agents use model `opus`; skill-explorer uses `haiku`
```

**After:**
```
- The skill-tester, consistency-tester, and skill-refiner agents use model `opus`; skill-explorer uses `haiku`
```

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add skill-refiner agent to CLAUDE.md"
```
