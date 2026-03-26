# Multirun Validation Design

**Goal:** Add a multirun validation phase to skill-bench that runs a skill draft against a user-provided test case library N times per case, summarizes consistency, collects user judgments with annotations, analyzes failure patterns, proposes targeted skill edits, and loops until the user is satisfied.

**Approach:** Single orchestrator agent (`consistency-tester`) handles the full validation loop. Spawns the existing skill-tester agent repeatedly. No changes to skill-tester itself.

---

## 1. Phase Structure

The workflow expands from 4 to 5 phases:

```
Phase 1: Design          (superpowers:brainstorming)
Phase 2: Plan            (superpowers:writing-plans)
Phase 3: Build & Test    (superpowers:subagent-driven-development)
Phase 4: Validate        ← NEW: multirun validation via consistency-tester agent
Phase 5: Finalize        (existing lint + promote, renumbered from Phase 4)
```

**Phase 4 entry gate:** Phase 3 complete — skill draft exists at `{drafts_dir}/{skill-name}/SKILL.md`, all plan tasks done, all Phase 3 reviews passed.

**Phase 4 exit gate:** User explicitly declares validation done. No automatic exit.

## 2. Test Case Library

User creates `.skillbench/test-cases/{skill-name}.json` before Phase 4 begins:

```json
{
  "run_count": 5,
  "cases": [
    {
      "name": "standard-code-review",
      "prompt": "Review this pull request for security issues based on the attached diff",
      "reference_files": ["samples/pr-example.diff", "samples/codebase-context.md"],
      "description": "Standard review with all sections populated"
    },
    {
      "name": "minimal-review-missing-context",
      "prompt": "Review this pull request with only the diff and no codebase context",
      "reference_files": ["samples/minimal-diff.txt"],
      "description": "Tests graceful handling of missing context"
    }
  ]
}
```

Fields:

| Field | Required | Description |
|-------|----------|-------------|
| `run_count` | Yes | Number of times to run each case (e.g., 5) |
| `cases` | Yes | Array of test cases |
| `cases[].name` | Yes | Kebab-case identifier for the case |
| `cases[].prompt` | Yes | The input prompt given to the skill |
| `cases[].reference_files` | No | Paths to files provided as context (relative to project root) |
| `cases[].description` | No | Human-readable note about what this case tests |

No `expected_patterns` or pass/fail criteria fields. The user judges outputs directly.

If the file doesn't exist when Phase 4 starts, skill-bench prompts the user to create it and provides the format.

## 3. Run Collection

The consistency-tester agent executes runs:

1. For each case in the library, use the Agent tool to spawn the skill-tester agent `run_count` times with identical inputs:
   - Current SKILL.md draft + any reference files from the skill
   - Case prompt
   - Case reference files
2. Runs are sequential (not parallel) to avoid context interference
3. Each run's full skill-tester output is saved to `.skillbench/test-history/{skill-name}/{case-name}-round-{r}-run-{n}.json`
   - `r` = validation round number (starts at 1, increments on re-runs)
   - `n` = run number within the round (1 to `run_count`)

## 4. Consistency Summary

After all runs complete for a case, the agent produces a consistency report:

```
## Case: standard-code-review (5 runs)

### Consistent Across Runs
- All runs produced 5 sections in correct order
- Severity ratings: consistent in all runs
- Summary header block: identical in 5/5

### Variations
- Section 3 (Findings): wording differs across runs, but structure consistent
- Section 4 (Recommendations): run 2 included an extra sub-bullet not in others
- Conclusion paragraph: 3 distinct phrasings observed

### Run Summaries
| Run | Status | Key Differences |
|-----|--------|-----------------|
| 1   | —      | baseline         |
| 2   | —      | extra sub-bullet in Section 5 |
| 3   | —      | same as run 1    |
| 4   | —      | same as run 1    |
| 5   | —      | shorter conclusion |
```

The Status column is blank until user provides judgment. The summary groups findings into "consistent" vs "variable" so the user knows where to focus.

Full simulated outputs from each run are available in test-history for inspection. The summary table is the primary review surface.

## 5. User Judgment

After presenting the consistency report, the agent collects user judgment:

1. For each case, agent asks the user to mark each run as pass or fail with optional annotation
2. User responds in natural language — e.g., "1-4 pass, 5 fail — missing compliance statement" or "all pass except run 2, the extra bullet in findings is wrong"
3. Agent parses into structured format and confirms understanding

Storage: `.skillbench/test-history/{skill-name}/{case-name}-round-{r}-judgment.json`:

```json
{
  "case": "standard-code-review",
  "round": 1,
  "judgments": [
    { "run": 1, "status": "pass", "note": null },
    { "run": 2, "status": "fail", "note": "extra sub-bullet in Section 4 shouldn't be there" },
    { "run": 3, "status": "pass", "note": null },
    { "run": 4, "status": "pass", "note": null },
    { "run": 5, "status": "fail", "note": "conclusion too short, missing summary statement" }
  ]
}
```

The agent does not second-guess user judgments. The user is the domain expert.

## 6. Failure Analysis & Skill Refinement

After judgments for all cases, the agent synthesizes:

**Pattern analysis:**
1. Collect all failure annotations across all cases
2. Group by root cause — failures with similar annotations or similar output differences get grouped
3. Present analysis:

```
## Failure Pattern Analysis

### Pattern 1: Unconstrained optional content (3 failures across 2 cases)
- standard-review: run 2 (extra sub-bullet), run 5 (missing summary statement)
- minimal-review: run 3 (invented context for missing information)
- Root cause: skill doesn't specify what to do when information is ambiguous —
  agent improvises differently each run

### Proposed Skill Edits
1. Add explicit constraint: "Do not add content beyond what the input data supports"
2. Add required section: "Summary statement must appear in conclusion"
3. Add missing-data rule: "For fields with no input data, write 'N/A — not provided'"
```

**Refinement:**
- Agent proposes specific edits to SKILL.md (or its reference files), tied to each failure pattern
- User approves, modifies, or rejects each proposed edit
- Agent applies approved edits using existing concurrent edit protection (SHA256 hash-check before writes)
- Each edit round is committed: `fix: address multirun failure pattern — {description}`

No autonomous editing. The agent never modifies the skill without user approval.

## 7. Re-run Loop

After edits are applied, the agent recommends which cases to re-run:

**Recommendation logic:**
- All runs passed → recommend SKIP
- Failures matched a fixed pattern → recommend RE-RUN
- Failures ambiguous or unfixed → recommend RE-RUN

```
## Re-run Recommendation

- standard-code-review: RE-RUN (2 failures linked to Pattern 1, now fixed)
- minimal-review-missing-context: RE-RUN (1 failure linked to Pattern 1)
- edge-case-empty-diff: SKIP (all 5 passed)

Re-run all, just the recommended ones, or pick your own?
```

User decides: all, recommended only, or custom selection.

Re-runs execute the same cycle: run N times → summarize → user judges → analyze → refine if needed. Round number increments. Previous round results are preserved.

The loop continues until the user declares validation complete.

## 8. Files Changed in the Plugin

| File | Action | What Changes |
|------|--------|-------------|
| `skills/skill-bench/SKILL.md` | Modify | Add Phase 4 (Validate) between current Phase 3 and Phase 4. Renumber current Phase 4 → Phase 5. ~30 lines of phase instructions. |
| `agents/consistency-tester.md` | Create | New Opus agent. Orchestrates: run collection, summarization, judgment collection, pattern analysis, skill refinement proposals, re-run recommendations. |
| `.claude-plugin/plugin.json` | Modify | Bump version to 0.3.0 |
| `CLAUDE.md` | Update | Add Phase 4 to phase mapping table, document consistency-tester agent, update artifact locations |
| `README.md` | Update | Add Phase 4 to workflow, document test case library format |

**Unchanged files:**
- `agents/skill-tester.md` — still does single simulation, returns structured eval. consistency-tester spawns it repeatedly.
- `agents/skill-explorer.md` — read-only scanner, unaffected.
- `skills/skill-bench/references/skill-format.md` — unchanged.
- `skills/skill-bench/references/anti-patterns.md` — unchanged.
- `skills/skill-bench/references/skill-authoring-plan-template.md` — unchanged. Phase 3 task structure stays the same; multirun validation is a separate phase.

**New runtime artifacts:**
- `.skillbench/test-cases/{skill-name}.json` — user-provided test case library (committable)
- `.skillbench/test-history/{skill-name}/{case-name}-round-{r}-run-{n}.json` — per-run results (gitignored)
- `.skillbench/test-history/{skill-name}/{case-name}-round-{r}-judgment.json` — user judgments (gitignored)
