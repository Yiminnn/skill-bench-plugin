# Express Entry Point Design Spec

**Date:** 2026-03-25
**Status:** Draft
**Scope:** New `skill-bench-express` skill — standalone fast path from (resource + prompt) to working skill

## 1. Invocation & Resource Parsing

The skill is invoked with natural language containing a skill description and embedded URLs with role annotations:

```
/skill-bench-express <natural language description with embedded URLs and role annotations>
```

### Parsing

The skill extracts from the natural language input:

1. **Skill description** — initial sentence(s) before any URLs. Becomes the seed for `name` (kebab-cased) and `description` (prefixed with "Use when") frontmatter.
2. **Resource groups** — each URL or cluster of URLs plus surrounding text. The surrounding text IS the role annotation.

### Role Classification

Natural language cues map to four role types, each with an extraction strategy:

| Role | Language Cues | Extraction Depth | Read Order |
|------|--------------|------------------|------------|
| **Primary** | "main guide", "step by step", "follow this workflow", "decision tree" | Full — every branch, criterion, step | 1st |
| **Ground truth** | "regulations", "ground truth", "underlies", "source of authority" | Citation-driven — read only sections cited by the primary | 2nd |
| **Named reference** | "for [specific topic]", "make this a separate reference" | Targeted — deep-read only the named concept | 3rd |
| **Background** | "foundational", "rationale", "context", "background", "history" | Structural — principles, definitions, key concepts only | 4th |

**Ambiguity handling:** If role can't be classified from the text, default to "Named reference" (targeted extraction) and flag in Triage for user confirmation.

**Multiple URLs per role:** When multiple URLs share the same role annotation, they're grouped and content is merged in extraction.

## 2. Fetch Phase

For each resource group, in read order (Primary → Ground truth → Named refs → Background):

1. **Detect URL type** — PDF (`/media/`, `.pdf`) vs. web page (HTML)
2. **Fetch content:**
   - Web pages: `WebFetch` directly — returns readable text
   - PDFs: `WebFetch` to download, then extract text. **Garble detection:** if the fetched content has < 100 readable words or > 50% non-alphanumeric characters, treat as garbled and fall back to saving locally and using `Read` (which supports PDF natively)
3. **Extract at role-appropriate depth** (see Role Classification table)
4. **Cross-document linking** — as each resource is processed, note citations to other resources. When reading the primary guide, collect all regulatory citations (e.g., "21 CFR 1271.10") — these become extraction targets for the ground truth phase.

### Read Order Cascade

```
Step 1: Fetch primary guide (full)
        → Collect citation list: [§1271.3(z), §1271.10(a), §1271.15(b), ...]

Step 2: Fetch ground truth (citation-driven)
        → Read ONLY sections from the citation list
        → Extract verbatim regulatory text for each cited section

Step 3: Fetch named reference (targeted)
        → Scan document for the named topic only
        → Ignore unrelated sections

Step 4: Fetch background (structural)
        → Extract: purpose/intent, key definitions, regulatory history
        → Skip procedural/administrative sections
```

### Error Handling

- URL unreachable: report which URL failed, ask for alternative or continue without
- PDF unreadable: report the issue, suggest user provide a local copy
- No content extracted: flag in Triage, ask user to clarify what to look for

No intermediate files — fetched content stays in conversation context until Generate writes artifacts.

## 3. Triage Phase

**Purpose:** Cheap checkpoint to catch silent misreading. The top failure mode for regulatory content.

**Skippable:** User can say "skip" before triage is shown to go directly to Generate.

### Triage Output Format

```
## Resource Triage

### Primary Guide: [title] ([pages] pages, full extraction)
- Decision tree: [N] branches identified
  1. [Branch 1 summary]
  2. [Branch 2 summary]
  ...
- Key criteria extracted: [summary]
- Regulatory cross-references found: [N] citations → will guide ground truth extraction

### Ground Truth: [regulation ID] (citation-driven, [N] sections extracted)
- [§section]: [definition/criterion]
- [§section]: [definition/criterion]
...

### Named Reference: [topic] (targeted)
- [N] criteria/examples found
- Links to: [cross-references]

### Background: [title] (structural, [N] documents)
- Foundational rationale: [2-3 sentence summary]
- Key definitions: [list]

### Resource Map
Primary Guide ──cites──→ Ground Truth ([N] sections)
Primary Guide ──references──→ Named Reference
Background ──provides context for──→ Primary Guide
```

### User Responses

- **"looks good" / "y"** → proceed to Generate
- **"you missed X" / "section Y is wrong"** → re-extract flagged section, update triage, re-present
- **"resource Z isn't relevant, drop it"** → remove from resource set

Triage is one message. Corrections loop back to re-extraction of the specific resource, not a full re-fetch.

## 4. Generate Phase

Produces three artifact types:

### 4a. Lightweight Design Spec (auto-generated)

Written to `.skillbench/specs/{skill-name}-design.md`. Contents:

- **Resource map** — URLs, classified roles, extraction depths
- **Cross-document citations** — primary guide citations mapped to ground truth sections
- **Skill structure outline** — planned SKILL.md sections and which resource informed each
- **Reference file plan** — which references will be generated and what each contains

~30-50 lines. Machine-generated audit trail, not a brainstormed design. Compatible with skill-bench Phase 1 output for recovery path.

### 4b. SKILL.md Draft

Written to `{drafts_dir}/{skill-name}/SKILL.md`. Structure per `skill-format.md`:

- **Frontmatter** — `name` (derived from description: extract the core noun phrase, kebab-case it, confirm with user in Triage — e.g., "361 HCT/P assessment" → propose `hctp-361-assessment`), `description` (starts with "Use when")
- **Overview** — 2-5 lines from primary guide's purpose statement
- **Workflow** — decision tree / process from full primary extraction. Every branch, criterion, step.
- **Quality gates** — regulatory checkpoints from ground truth citations
- **Reference pointers** — links to each generated reference file

### 4c. Reference Files

Written to `{drafts_dir}/{skill-name}/references/`. One file per resource role (excluding Primary, which becomes the SKILL.md workflow):

| Resource Role | File | Content |
|---|---|---|
| Named reference | `references/{topic-name}.md` | Targeted extraction of the named concept |
| Background | `references/background.md` | Merged from all background URLs |
| Ground truth | `references/{regulation-id}.md` | Cited sections with verbatim regulatory text |

Each reference file includes source URLs for traceability.

### Size Budget

After generation, report: `SKILL.md: {lines} lines (~{lines * 6} tokens)`. If SKILL.md exceeds 200 lines, auto-split heavy sections into references. Decision tree workflow stays in SKILL.md; supporting detail moves to references.

## 5. Present Phase

Hard gate — no auto-advance.

1. **Write all files** to `{drafts_dir}/{skill-name}/`
2. **Show file summary table** — filename, lines, token estimate, total
3. **Prompt with domain-specific review guidance:**
   > "Draft written to `{drafts_dir}/{skill-name}/`. Please review in your editor — especially:
   > - **Decision tree completeness** — are all branches from the guidance captured?
   > - **Regulatory citations** — do the references match the source documents?
   > - **Terminology** — are domain terms used correctly and consistently?
   >
   > When you're done, tell me what to change, or say 'looks good' to continue."
4. **Wait for user response:**
   - "looks good" → proceed to Sample Run
   - Specific feedback → edit draft, re-present affected files
   - "I edited it myself" → re-read files, acknowledge changes, proceed

## 6. Sample Run Phase (Optional)

Quick sanity check — one skill-tester invocation, not full multirun validation.

**Trigger:** After Present phase approval, offer:
> "Want me to run the skill against a sample case to see it working? (You can also skip to finalization or manual testing.)"

**If yes:**
1. Generate a sample input from the primary guide — a straightforward scenario exercising the main decision tree path
2. Spawn **skill-tester** agent with: draft SKILL.md + reference files + sample input
3. Show skill-tester's structured output (simulated output, issues, thinking trace)
4. User decides:
   - "looks good" → proceed to Finalize
   - Specific feedback → edit draft, optionally re-run
   - "run again with a different case" → new scenario, re-run
   - "I want full validation" → prompt for test case library (`.skillbench/test-cases/{skill-name}.json`), spawn **consistency-tester** agent (escalation to shared multirun infrastructure)

**If skipped:** proceed directly to Finalize.

## 7. Finalize Phase

Lint + promote. Same quality bar as skill-bench Phase 5, using shared references.

### Lint Pass

Load `skill-bench/references/anti-patterns.md` and `skill-bench/references/skill-format.md` (cross-referenced, not duplicated).

1. **Frontmatter:** name kebab-case, description starts with "Use when", model valid or omitted
2. **Content:** no "You are..." instructions, no hardcoded nonexistent paths, no non-standard tool references, size within budget
3. **References:** verify all `skill: \`name\`` and `**name** agent` references exist
4. **Description quality:** trigger-oriented, no overlap with existing skills
5. **Token efficiency:** line/token counts, flag if SKILL.md > 200 or total > 500
6. **Source traceability** (express-specific): every reference file includes source URLs, every regulatory citation traces to a ground truth reference section

Present as **Blocking** / **Warning** / **Info**.

### Promotion

1. Ask target location: `skills/{skill-name}/`, plugin directory, or custom path
2. Move files from `{drafts_dir}/{skill-name}/` to target
3. Verify move via Glob
4. Suggest commit: `feat: add {skill-name} skill`
5. Offer to clean up draft directory

Auto-generated design spec stays in `.skillbench/specs/` — already in its final location.

## 8. File Structure & Shared Infrastructure

### New Files

```
skills/skill-bench-express/
  SKILL.md                          # The express skill (~150-180 lines)
  references/
    role-extraction-guide.md        # Role classification cues + extraction depth rules
```

### Modified Files

```
.claude-plugin/plugin.json          # Version bump to 0.3.0
```

No new *plugin* dependencies — express uses standard Claude Code tools (WebFetch, Read, Write, Edit, Glob, Agent) and shared agents from this plugin. No superpowers plugin required.

### Shared Infrastructure

| Component | Location | Express | skill-bench |
|---|---|---|---|
| `skill-format.md` | `skills/skill-bench/references/` | Generate + Finalize | Phase 1, 3, 5 |
| `anti-patterns.md` | `skills/skill-bench/references/` | Finalize | Phase 5 |
| `skill-tester` agent | `agents/skill-tester.md` | Sample Run | Phase 3 |
| `consistency-tester` agent | `agents/consistency-tester.md` | Escalation | Phase 4 |
| `skill-explorer` agent | `agents/skill-explorer.md` | Listing drafts | Listing drafts |
| Config | `.skillbench/config.json` | `drafts_dir` | All phases |
| Design specs | `.skillbench/specs/` | Auto-generated | Brainstormed |
| Test cases | `.skillbench/test-cases/` | On escalation | Phase 4 |
| Test history | `.skillbench/test-history/` | Sample Run results | Phase 3-4 |

### Recovery Path

The auto-generated design spec is compatible with skill-bench Phase 1 output. If a user starts with express and later wants the full TDD treatment, they invoke `skill-bench` and it picks up the existing spec at Phase 2 (Plan).

### Graceful Degradation

If skill-bench references can't be loaded (unlikely — same plugin), Finalize skips lint checks it can't load and warns the user.

### Trade-off Accepted

~20 lines of duplicated lint/promotion logic vs. coupling cost of merging into one skill. The shared references (anti-patterns.md, skill-format.md) mitigate this.
