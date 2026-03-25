# Role-Driven Resource Extraction Guide

When a user provides URLs with natural language descriptions, the descriptions ARE the extraction strategy. Classify each resource into a role, then extract at the depth that role demands.

## Role Classification

| Role | Language Cues | Extraction Depth |
|------|--------------|------------------|
| **Primary** | "main guide", "step by step", "follow this workflow", "decision tree", "as the main guide" | Full — every branch, criterion, step, decision point |
| **Ground truth** | "regulations", "ground truth", "underlies", "source of authority", "the rules" | Citation-driven — read only sections cited by the primary |
| **Named reference** | "for [specific topic]", "make this a separate reference", "follow the recommendations" | Targeted — deep-read only the named concept |
| **Background** | "foundational", "rationale", "context", "background", "history", "thinking" | Structural — principles, definitions, key concepts only |

**Default:** If role can't be classified, use "Named reference" and flag in Triage for user confirmation.

**Multiple URLs per role:** When several URLs share the same role annotation (e.g., three background documents listed together), group them. Their content merges in extraction.

## Read Order

Process resources in this order — each read informs the next:

```
1. Primary guide (full extraction)
   → Output: complete decision tree/workflow + citation list

2. Ground truth (citation-driven)
   → Input: citation list from step 1
   → Output: verbatim regulatory text for each cited section

3. Named references (targeted)
   → Input: topic name from user's annotation
   → Output: criteria, definitions, examples for the named concept

4. Background (structural)
   → Output: purpose/intent, key definitions, regulatory history
   → Skip: procedural, administrative, boilerplate sections
```

## Extraction Depth Details

### Full Extraction (Primary)

Read every page. Capture:
- All decision tree branches with criteria at each node
- All steps in any process or workflow
- All definitions used in the decision logic
- All cross-references to other documents (these become the citation list)
- All exceptions, exemptions, and special cases

This resource IS the skill's logic — missing a branch means the skill gives wrong answers.

### Citation-Driven Extraction (Ground Truth)

Do NOT read the entire regulation. Instead:
1. Take the citation list from the primary guide extraction
2. For each citation (e.g., "21 CFR 1271.10(a)"), locate that specific section
3. Extract the verbatim regulatory text
4. Note any cross-references within the extracted sections (one level deep only)

### Targeted Extraction (Named Reference)

The user's annotation names what to find. For example, "same surgical procedure exemption" means:
1. Scan the document's structure (TOC, headers)
2. Locate sections about the named topic
3. Deep-read those sections: criteria, definitions, examples, exceptions
4. Ignore everything else in the document

### Structural Extraction (Background)

Skim for conceptual grounding, not operational detail:
- Purpose and intent statements
- Key definitions that aren't in the primary guide
- Regulatory history and rationale for the framework
- Skip: implementation procedures, registration forms, compliance timelines

## URL Type Handling

- **PDF** (cues: `/media/`, `.pdf` in URL): Fetch via `WebFetch`. If result has < 100 readable words or > 50% non-alphanumeric characters, content is garbled — fall back to saving locally and reading with the `Read` tool (which supports PDF natively).
- **Web pages** (HTML): Fetch via `WebFetch` directly.
- **Federal Register**: Web pages with structured sections — extract by section header.
- **eCFR**: Web pages with regulatory sections — extract by part/section number.

## Cross-Document Linking

As you process the primary guide, maintain a citation list:
```
Citations found:
- 21 CFR 1271.3(z) → "same surgical procedure" definition
- 21 CFR 1271.10(a) → criteria for 361 HCT/Ps
- 21 CFR 1271.15(b) → exemptions
```

This list drives the ground truth extraction and informs the resource map in Triage.
