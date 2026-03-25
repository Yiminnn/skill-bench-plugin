# Quick Start: Creating Skills from Reference Documents

This guide is for domain experts who want to turn guidance documents, regulations, and procedures into reusable Claude Code skills.

## What You Need

- Claude Code installed and working
- This plugin installed: `claude plugins add https://github.com/Yiminnn/skill-bench-plugin`
- URLs to your source documents (guidance PDFs, regulations, government notices)

## Step 1: Create the Skill

In Claude Code, type one message with:
- What the skill should do
- Your source documents as URLs, with a note explaining what each one is for

Example:

```
/skill-bench-express create a new skill for [topic] assessment,
here is the guidance document that you should follow step by step
using the decision tree in the document as the main guide for workflow:
[URL to primary guidance PDF]

For the [specific sub-topic] step, follow the recommendations in
this guidance and make this a separate reference:
[URL to supporting guidance]

The regulations that underlie this framework should serve as the
ground truth and be a separate reference:
[URL to regulations]
```

**Tips for describing your documents:**
- Say "main guide" or "step by step" for the primary document the skill should follow
- Say "ground truth" or "regulations" for the underlying rules
- Say "separate reference" for supporting documents on specific topics
- Say "background" or "rationale" for context documents

## Step 2: Review the Triage

Claude will read your documents and show a summary of what it found — decision tree branches, key criteria, regulatory sections. Check that:

- All major steps from the guidance are listed
- No important criteria were missed
- The proposed skill name makes sense

Type **y** if it looks right, or tell Claude what it missed.

## Step 3: Review the Draft

Claude generates the skill files and asks you to review them in your editor. Open the files and check:

- **Decision tree** — Are all branches from the guidance captured?
- **Regulatory citations** — Do they match the source documents?
- **Terminology** — Are domain terms used correctly?

Tell Claude what to change, or type **looks good** to continue.

## Step 4: Test with a Sample Case

Claude offers to run the skill against a sample case. Say **yes** to see how it performs. Review the output for correctness.

## Step 5: Test with Real Cases (Recommended)

For thorough validation, say **full validation** when prompted. Claude will ask you to describe your test cases. For each case, provide:

- A name (e.g., "standard-bone-graft")
- The input prompt (what a user would ask the skill)
- Any reference files the skill would need

Claude runs each case 5 times and shows you:
- What was **consistent** across all runs
- What **varied** between runs

For each run, mark it **pass** or **fail** and describe what went wrong for failures. Example:

> "Run 3 failed — it skipped the eligibility check in step 2"

Claude analyzes the failure patterns and proposes specific fixes. Review the proposed changes and approve the ones that make sense. Then re-run to verify.

Repeat until all cases pass consistently.

## Step 6: Finalize

Claude checks the skill for formatting issues and asks where to save it. Choose your project's skills directory and commit.

## Quick Reference

| You want to... | Say... |
|---|---|
| Create a skill from documents | `/skill-bench-express [description + URLs]` |
| Skip the triage summary | "skip" |
| Approve a step | "y" or "looks good" |
| Fix something in the draft | Describe what's wrong, or edit the file yourself and say "I edited it" |
| Run a quick test | "yes" (when offered sample run) |
| Run thorough testing | "full validation" |
| Mark a test run as failed | "run 3 failed — [what went wrong]" |
| Approve all proposed fixes | "approve all" |
| Approve some fixes | "approve fix 1 and 3" |
| Stop testing | "validation complete" |
