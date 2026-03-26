---
name: hctp-361-assessment
description: "Use when assessing whether a human cell, tissue, or cellular and tissue-based product (HCT/P) qualifies for regulation solely under section 361 of the PHS Act and 21 CFR Part 1271, or requires premarket approval as a drug, device, or biological product under section 351."
---

# 361 HCT/P Regulatory Assessment

Guides a structured assessment of whether an HCT/P meets the criteria for regulation solely under section 361 of the PHS Act and 21 CFR Part 1271. Follows the FDA decision tree from the "Regulatory Considerations for Human Cells, Tissues, and Cellular and Tissue-Based Products: Minimal Manipulation and Homologous Use" guidance (July 2020).

## Assessment Workflow

Follow these steps in order. Each step is a gate — do not skip ahead.

### Step 1: Is the Product an HCT/P?

Determine whether the product meets the definition in 21 CFR 1271.3(d): an article containing or consisting of human cells or tissues intended for implantation, transplantation, infusion, or transfer into a human recipient.

**Not HCT/Ps** (21 CFR 1271.3(d) exclusions):
- Vascularized human organs for transplantation
- Whole blood or blood components (including PRP)
- Secreted or extracted human products (milk, collagen, cell factors) — except semen
- Minimally manipulated bone marrow for homologous use not combined with another article
- Ancillary products used in HCT/P manufacture
- Cells, tissues, organs derived from animals
- In vitro diagnostic products
- Blood vessels recovered with an organ for organ transplantation

**If NOT an HCT/P** → 21 CFR Part 1271 does not apply. Assessment complete.
**If IS an HCT/P** → Proceed to Step 2.

### Step 2: Does the Same Surgical Procedure Exception Apply?

Assess 21 CFR 1271.15(b) BEFORE evaluating the four criteria. This exception is independent of the 1271.10(a) criteria.

The exception applies when ALL three conditions are met:
1. **Autologous use** — HCT/P removed from and implanted into the same individual
2. **Same surgical procedure** — removal and implantation within a single operation at the same establishment
3. **"Such HCT/P"** — the HCT/P remains in its original form (only rinsing, cleansing, sizing, or shaping performed)

See `references/same-surgical-procedure.md` for detailed Q&A, multi-operation exceptions (craniotomy, parathyroidectomy), and processing examples (adipose tissue).

**Key distinctions:**
- Minimal manipulation (1271.10(a)) is a DIFFERENT standard than "such HCT/P" (1271.15(b))
- Processing beyond rinsing/cleansing/sizing/shaping typically disqualifies from the exception, even if it would be considered "minimal manipulation"
- Shipping to another establishment for implantation disqualifies (constitutes distribution = manufacturing)

**If exception APPLIES** → Not subject to Part 1271 requirements. Assessment complete.
**If exception does NOT apply** → Proceed to Step 3.

### Step 3: Evaluate the Four Criteria (21 CFR 1271.10(a))

The HCT/P must meet ALL FOUR criteria to be regulated solely under section 361. Failure on ANY criterion means the product requires premarket approval.

#### Criterion 1: Minimally Manipulated (21 CFR 1271.10(a)(1))

First determine if the HCT/P is **structural tissue** or **cells/nonstructural tissue** based on characteristics in the donor (before recovery/processing):

**Structural tissue** (physically supports, serves as barrier/conduit, connects, covers, cushions):
- Minimal manipulation = processing does NOT alter original relevant characteristics relating to utility for reconstruction, repair, or replacement
- Examples: bone, skin, amniotic membrane, blood vessel, adipose tissue, cartilage, tendon/ligament

**Cells/nonstructural tissue** (metabolic or biochemical roles — hematopoietic, immune, endocrine):
- Minimal manipulation = processing does NOT alter relevant biological characteristics
- Examples: reproductive cells, hematopoietic stem cells, lymph nodes, parathyroid, peripheral nerve

See `references/minimal-manipulation.md` for detailed examples of minimal vs. more-than-minimal manipulation by tissue type.

#### Criterion 2: Homologous Use Only (21 CFR 1271.10(a)(2))

Homologous use = repair, reconstruction, replacement, or supplementation of a recipient's cells or tissues with an HCT/P that performs the **same basic function(s)** in the recipient as in the donor (21 CFR 1271.3(c)).

- Basic function = what the HCT/P does biologically/physiologically in its native state in the donor
- Does NOT need to be in the same anatomic location
- Must be reflected in labeling, advertising, or other indications of manufacturer's objective intent
- If intended for a "myriad of diseases or conditions," likely NOT homologous use

See `references/homologous-use.md` for examples by tissue type.

#### Criterion 3: Not Combined with Another Article (21 CFR 1271.10(a)(3))

Manufacture does NOT involve combination with another article, EXCEPT:
- Water
- Crystalloids
- Sterilizing, preserving, or storage agent (provided it does not raise new clinical safety concerns)

#### Criterion 4: No Systemic Effect / Not Dependent on Metabolic Activity (21 CFR 1271.10(a)(4))

Either:
- **(i)** The HCT/P does NOT have a systemic effect AND is NOT dependent on metabolic activity of living cells for its primary function; OR
- **(ii)** The HCT/P HAS a systemic effect or IS dependent on metabolic activity, AND is:
  - For autologous use; OR
  - For allogeneic use in a first- or second-degree blood relative; OR
  - For reproductive use

### Step 4: Determine Regulatory Outcome

| Result | Regulatory Status |
|--------|------------------|
| Meets ALL four criteria | Regulated solely under section 361 of PHS Act and 21 CFR Part 1271. Must register, list, and comply with Part 1271. |
| Fails ANY criterion | Regulated as drug, device, and/or biological product under FD&C Act and/or section 351 of PHS Act. Requires premarket approval (BLA/IND). |

### Step 5: Report Assessment

Present findings in this structure:

```
## 361 HCT/P Assessment: [Product Name]

### Product Description
[What the product is and its intended use]

### Step 1: HCT/P Definition
[Met/Not met] — [reasoning]

### Step 2: Same Surgical Procedure Exception
[Applies/Does not apply] — [reasoning for each of the 3 conditions]

### Step 3: Four Criteria Analysis
1. **Minimal manipulation:** [Met/Not met] — [structural or cellular classification, processing analysis]
2. **Homologous use:** [Met/Not met] — [basic function in donor vs. intended use in recipient]
3. **No combination:** [Met/Not met] — [any non-exempt articles combined]
4. **No systemic effect:** [Met/Not met] — [systemic effect analysis, or exception pathway]

### Determination
[361 HCT/P / Requires premarket approval] — [summary reasoning]

### Regulatory Citations
[List all cited CFR sections and guidance documents]
```

## Quality Gates

Before finalizing any assessment:
- [ ] Each criterion analyzed with specific regulatory citations
- [ ] Structural vs. cellular classification justified for Criterion 1
- [ ] Basic function identified from donor perspective for Criterion 2
- [ ] Processing steps enumerated and evaluated individually
- [ ] All four criteria addressed — none skipped
- [ ] Same surgical procedure exception evaluated BEFORE the four criteria
- [ ] Assessment uses FDA guidance examples as precedent where applicable

## References

- `references/same-surgical-procedure.md` — SSP exception Q&A from FDA guidance (November 2017)
- `references/minimal-manipulation.md` — Structural and cellular tissue examples with manipulation analysis
- `references/homologous-use.md` — Homologous use examples by tissue type
- `references/regulatory-text.md` — Key 21 CFR Part 1271 sections (verbatim from guidance citations)
- `references/background.md` — Foundational rationale for the tiered regulatory framework (1997 Proposed Approach)
