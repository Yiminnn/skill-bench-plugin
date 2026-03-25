# Skill Refiner Agent — Design Spec

> Date: 2026-03-25
> Status: Approved (design sections 1-6)

## Overview

A narrow-scope, single-shot reasoning agent that takes skill content, multiple test results with thinking traces, and user annotations, then analyzes failure patterns across runs and proposes targeted skill edits. It does NOT own the run@5 orchestration loop — it is invoked by the skill-bench orchestrator after test results are collected and user feedback is gathered.

## 1. Agent Identity & Inputs

### Frontmatter

```yaml
---
name: skill-refiner
description: "Use when a skill draft has been tested against multiple cases and needs refinement based on test results and user feedback. Takes skill content, N test results with thinking traces, and user annotations identifying which results failed and why. Analyzes failure patterns across runs and proposes targeted skill edits."
model: opus
tools: Read, Glob, Grep, Edit, Write
---
```

### Inputs

The orchestrator provides these when spawning the agent:

1. **Skill content** — path to the SKILL.md draft + its `references/` directory
2. **Test results** — file paths to skill-tester outputs from the current batch (stored in `.skillbench/test-history/{skill-name}/`), each containing status, simulated output, issues, and thinking trace. The agent reads these files itself.
3. **User annotations** — either:
   - Structured: per-run verdict (pass/fail + note), or
   - Freeform: natural language describing what's wrong across runs
4. **Test history path** — `.skillbench/test-history/{skill-name}/` (agent reads prior rounds on its own)

The agent reads skill content and reference files itself via `Read` and `Glob` — the orchestrator provides paths, not inline content. This keeps the orchestrator's prompt small and lets the agent navigate the skill's file structure.

## 2. Dual-Lens Analysis

The agent's core reasoning follows two passes, then merges findings.

### Pass 1 — Cross-Run Consistency Analysis

1. Extract the "Simulated Output" section from each test result
2. Diff outputs pairwise — identify segments that are stable across all runs vs. segments that vary
3. Classify variations: structural (section missing/reordered), content (different values/wording), formatting (different layout of same content)
4. For each variation cluster, trace back to the skill instruction that governs that section — if the instruction is ambiguous or underspecified, that's the root cause

### Pass 2 — Per-Run Correctness Analysis

1. For each run the user marked as failed, read the thinking trace from skill-tester
2. Identify the decision point where the agent went wrong — what instruction did it follow, and why did that instruction lead to incorrect output?
3. Check whether the failure is a skill logic error (instruction says the wrong thing) or a skill clarity error (instruction is correct but the agent misinterpreted it)
4. Cross-reference with test history: has this same failure appeared in previous refinement rounds? If so, flag as recurring and note what prior fix was attempted

### Merge

- Deduplicate findings (a variation found in Pass 1 may overlap with a failure in Pass 2)
- Categorize each finding as:
  - **Consistency** — skill is underspecified, causing run-to-run variation
  - **Correctness** — skill says/implies the wrong thing
  - **Regression** — a previously fixed issue has returned
- Rank by severity: regressions first (something broke), then correctness (wrong output), then consistency (variable output)

## 3. Output Format

The agent produces a structured report in two parts. Part 1 is always returned. Part 2 is returned alongside Part 1 but NOT applied until the orchestrator re-invokes with approval.

### Part 1 — Analysis Report

```
## Refinement Analysis

**Skill:** {skill-name}
**Runs analyzed:** {N} (batch) + {M} prior rounds from history
**User feedback format:** structured | freeform

### Regressions (highest severity)
[Issues from prior rounds that returned:]
- **Issue:** {description}
- **Prior fix:** {what was tried in round N}
- **Why it regressed:** {assessment}
- **Severity:** critical

### Correctness Issues
[For each:]
- **Failure:** {what went wrong}
- **Affected runs:** {which runs}
- **Decision point:** {thinking trace excerpt showing where the agent went wrong}
- **Root cause:** {skill logic error / clarity error}
- **Severity:** high | medium | low

### Consistency Issues
[For each:]
- **Variation:** {what differs across runs}
- **Affected section:** {which skill instruction governs this}
- **Root cause:** {underspecified instruction / missing constraint / ambiguous phrasing}
- **Severity:** high | medium | low

### Summary
- {N} consistency issues ({X} high)
- {N} correctness issues ({X} high)
- {N} regressions
```

### Part 2 — Proposed Edits

```
### Proposed Changes

[For each finding, ordered by severity:]

#### Fix {N}: {one-line description}
**Addresses:** {which finding(s) from Part 1}
**File:** {path}
**Rationale:** {why this change fixes the root cause, not just the symptom}

**Before:**
{exact current text}

**After:**
{proposed replacement}
```

The before/after blocks use exact strings from the file so the agent can apply them via `Edit` tool when approved. The rationale field lets the user judge whether the fix addresses the actual root cause or just patches the symptom.

## 4. Application Flow

After the orchestrator shows the analysis + proposed edits to the user, three outcomes:

### Approve all
Orchestrator spawns a new skill-refiner via the Agent tool with: skill path, the analysis output from the previous invocation, and instruction "Apply all proposed edits." The agent re-reads the skill file, applies each edit via `Edit` tool in severity order, reports what it changed.

### Approve some
Same as above, but instruction is "Apply fixes 1, 3, 5." The agent applies only specified edits, skips the rest.

### Reject with feedback
User provides additional feedback (e.g., "Fix 2 is wrong, the date should be ISO format"). Orchestrator spawns a new skill-refiner with all original inputs (skill path, test result paths, original annotations) plus the additional user feedback. The agent re-analyzes and produces a revised proposal.

Each invocation is stateless — the orchestrator must provide the skill path and relevant context each time.

### Concurrent Edit Protection

Before applying any edit, the agent reads the file and checks whether the "Before" text still exists. If the file has been modified since the analysis (user made manual edits):

1. Reports which proposed edits can no longer be applied (text doesn't match)
2. Applies the ones that still match
3. For broken edits, shows what changed and suggests adapted fixes

### Post-Application Report

```
## Edits Applied

| Fix | File | Status |
|-----|------|--------|
| Fix 1: {desc} | SKILL.md:42 | Applied |
| Fix 3: {desc} | SKILL.md:78 | Applied |
| Fix 5: {desc} | references/format.md:15 | Conflict — manual edit detected, skipped |

Recommend re-running test batch to verify fixes.
```

## 5. History Integration

The agent reads `.skillbench/test-history/{skill-name}/` to build a refinement timeline.

### What it reads

- `{timestamp}.json` — simulated test results (existing skill-tester format)
- `{timestamp}-pressure.json` — pressure test results (existing format)
- `{timestamp}-refinement.json` — refinement records (new artifact, written by this agent)

### Refinement Record Format

Written by skill-refiner after applying edits:

```json
{
  "timestamp": "2026-03-25T14:30:00Z",
  "skill_name": "fda-361",
  "runs_analyzed": 5,
  "findings": {
    "consistency": 2,
    "correctness": 1,
    "regressions": 0
  },
  "edits_applied": [
    {
      "fix": "Fix 1: Add explicit ISO date format constraint",
      "file": "SKILL.md",
      "finding_type": "consistency",
      "severity": "high"
    }
  ],
  "edits_skipped": [],
  "user_feedback_format": "freeform"
}
```

### How History Informs Analysis

- If the agent sees a prior refinement record that targeted the same skill section with a similar fix, it flags a regression and notes the prior attempt
- If 3+ rounds have targeted the same section, the agent suggests the section may need a structural rewrite rather than incremental patching
- History is read-only during analysis, written once after edits are applied

### Staleness

The agent trusts the skill's current file content over history. If history says "section B was fixed in round 2" but the current SKILL.md doesn't contain that fix, the agent treats it as a regression without assuming why.

## 6. Integration with skill-bench

### Invocation Point

Phase 3 (Build & Test) of skill-bench, after the user has test results and wants to refine. The orchestration sequence:

1. Orchestrator runs skill-tester N times (new SKILL.md Phase 3 capability)
2. Orchestrator diffs/summarizes results for user (new capability)
3. User reviews, annotates failures
4. Orchestrator invokes **skill-refiner** with (skill path, test results, annotations)
5. Skill-refiner returns analysis + proposed edits
6. Orchestrator shows to user
7. User approves → orchestrator re-invokes skill-refiner to apply
8. Loop back to step 1 for verification

### Out of Scope

- Multi-run orchestration (steps 1-3, 8) — SKILL.md Phase 3 enhancement
- Auto-summary/diffing of N runs (step 2) — separate agent or inline logic
- Express entry point for non-technical users — separate skill

### Registration

Agent lives at `agents/skill-refiner.md`. `plugin.json` already points to `./agents/`, so it's auto-discovered. No plugin.json changes needed.

### Scope Boundary

Skill-refiner never invokes skill-tester. It never runs the skill. It only receives results and reasons about them. This keeps it focused and testable.

## Files Changed

| File | Action | Description |
|------|--------|-------------|
| `agents/skill-refiner.md` | Create | The agent definition |
| No other files | — | Agent is self-contained; orchestration changes are out of scope |
