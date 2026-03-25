---
name: skill-bench-express
description: "Use when creating a skill from reference documents and a prompt. Fast path: fetches resources, extracts content by role, generates SKILL.md with references, and presents for review. No brainstorming ceremony — resources plus description in, working skill out."
---

# Skill Bench Express

Fast path from (resource URLs + prompt) to working skill. The user provides a natural language description with embedded URLs and role annotations. You fetch, extract, generate, and present — no design ceremony.

## Phase 1: Parse Invocation

Extract from the user's natural language input:

1. **Skill description** — the initial sentence(s) before any URL appears. This seeds the `name` (kebab-cased core noun phrase) and `description` (prefixed with "Use when") frontmatter fields.
2. **Resource groups** — each URL or cluster of URLs plus surrounding text. The surrounding text IS the role annotation.

Read `references/role-extraction-guide.md` for role classification. Map each resource group to one of:
- **Primary** — "main guide", "step by step", "decision tree" → full extraction
- **Ground truth** — "regulations", "ground truth", "underlies" → citation-driven extraction
- **Named reference** — "for [topic]", "make this a separate reference" → targeted extraction
- **Background** — "foundational", "rationale", "context" → structural extraction

If a role can't be classified, default to Named reference and flag it in Triage.

Propose a kebab-case skill name derived from the description. Include it in the Triage output for confirmation.

## Phase 2: Fetch

Process resources in read order — each read informs the next:

### Step 1: Primary guide (full)

Fetch the URL. Read every section. Capture:
- All decision tree branches / workflow steps / process logic
- All criteria and definitions used in the decision logic
- All cross-references to other documents → **collect as the citation list**

### Step 2: Ground truth (citation-driven)

Using the citation list from Step 1, fetch the ground truth URL and extract ONLY the cited sections. Extract verbatim regulatory text for each citation.

### Step 3: Named references (targeted)

For each named reference, fetch the URL and extract only the named topic — criteria, definitions, examples. Ignore unrelated sections.

### Step 4: Background (structural)

Fetch background URLs. Extract purpose/intent, key definitions, regulatory history. Skip procedural/administrative content.

### URL handling

- PDFs (`/media/`, `.pdf`): `WebFetch` first. If result has < 100 readable words or > 50% non-alphanumeric characters, save locally and use `Read` (supports PDF natively).
- Web pages: `WebFetch` directly.

### Error handling

- URL unreachable → report which failed, ask for alternative or continue without
- PDF unreadable → suggest user provide a local copy
- No content extracted → flag in Triage

## Phase 3: Triage

**Skippable** — if the user says "skip" before seeing triage, proceed to Generate.

Present a single-message structured summary:

For each resource: title, role, extraction depth, key findings (branches/criteria/sections found). Include proposed skill name and a resource map showing cross-document citations.

**User responses:** "y" → Generate. "you missed X" → re-extract flagged section. "drop Z" → remove. Name change → update.

## Phase 4: Generate

### 4a. Lightweight design spec

Write to `.skillbench/specs/{skill-name}-design.md`:
- Resource map (URLs, roles, extraction depths)
- Cross-document citations (primary → ground truth mapping)
- Skill structure outline
- Reference file plan

This is a machine-generated audit trail (~30-50 lines), not a brainstormed design. It is compatible with `skill-bench` Phase 1 output if the user later wants the full workflow.

### 4b. Project config

If `.skillbench/config.json` doesn't exist, create it with `drafts_dir: "skills/drafts"`, `test_model: "claude-opus-4-6"`, `context_files: []`. Read it if it exists. Use `drafts_dir` as base.

### 4c. SKILL.md draft

Write to `{drafts_dir}/{skill-name}/SKILL.md`. Structure per `skill-format.md`:

- **Frontmatter** — `name` (confirmed in Triage), `description` (starts with "Use when")
- **Overview** — 2-5 lines from primary guide's purpose
- **Workflow** — decision tree / process from primary extraction. Every branch, criterion, step.
- **Quality gates** — regulatory checkpoints from ground truth
- **Reference pointers** — `See references/{name}.md for...` links

### 4d. Reference files

Write to `{drafts_dir}/{skill-name}/references/`:

| Resource Role | File |
|---|---|
| Named reference | `references/{topic-name}.md` |
| Background | `references/background.md` |
| Ground truth | `references/{regulation-id}.md` |

Each reference file includes source URLs for traceability.

Primary guide content does NOT become a reference — it becomes the SKILL.md workflow.

### Size budget

After writing, report: `SKILL.md: {lines} lines (~{lines * 6} tokens)`.
- Over 200 lines → auto-split heavy sections into references. Keep decision tree in SKILL.md.
- Report total across all files.

## Phase 5: Present

**Hard gate — no auto-advance.**

1. Show file summary table (filename, lines, tokens, total)
2. Prompt:
   > "Draft written to `{drafts_dir}/{skill-name}/`. Please review in your editor — especially:
   > - **Decision tree completeness** — are all branches from the guidance captured?
   > - **Regulatory citations** — do the references match the source documents?
   > - **Terminology** — are domain terms used correctly and consistently?
   >
   > When you're done, tell me what to change, or say 'looks good' to continue."
3. Wait for response:
   - "looks good" → offer Sample Run
   - Specific feedback → edit, re-present affected files
   - "I edited it myself" → re-read files, acknowledge, proceed

## Phase 6: Sample Run (Optional)

After draft approval, offer:
> "Want me to run the skill against a sample case? (You can skip to finalization.)"

**If yes:**
1. Generate a sample input from the primary guide — a straightforward scenario on the main decision path
2. Spawn the **skill-tester** agent with: draft SKILL.md + reference files + sample input
3. Show skill-tester output (simulated output, issues, thinking trace)
4. User decides:
   - "looks good" → Finalize
   - Feedback → edit, optionally re-run
   - "different case" → new scenario, re-run
   - "full validation" → prompt for test case library at `.skillbench/test-cases/{skill-name}.json`, spawn **consistency-tester** agent

**If skipped:** proceed to Finalize.

## Phase 7: Finalize

Lint and promote. Load `skill-bench/references/anti-patterns.md` and `skill-bench/references/skill-format.md` (shared from skill-bench, not duplicated).

### Lint pass

1. **Frontmatter:** name kebab-case, description starts "Use when", model valid or omitted
2. **Content:** no "You are..." instructions, no hardcoded nonexistent paths, no non-standard tools, size within budget
3. **References:** verify all skill/agent cross-references exist (Glob)
4. **Description quality:** trigger-oriented, no overlap with existing skills
5. **Token efficiency:** flag SKILL.md > 200 lines or total > 500 lines
6. **Source traceability** (express-specific): every reference includes source URLs, every regulatory citation traces to a ground truth section

Present as **Blocking** / **Warning** / **Info**. Fix blocking issues before promotion.

### Promotion

1. Ask target location: `skills/{skill-name}/`, plugin directory, or custom path
2. Move files from `{drafts_dir}/{skill-name}/` to target
3. Verify via Glob
4. Suggest commit: `feat: add {skill-name} skill`
5. Offer to clean up draft directory. Design spec stays in `.skillbench/specs/`.
