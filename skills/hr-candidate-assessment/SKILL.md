---
name: hr-candidate-assessment
description: Assesses candidates against a role by parsing their CVs, scoring against a role-specific rubric, and producing a ranked long-list with per-candidate scorecards. Uses blind review to reduce bias. Produces PIF-styled Word document (rubric + methodology) and Excel scorecard (ranked candidates with per-dimension scores). Trigger phrases include "assess candidates for [role]", "screen CVs for [role]", "rank these candidates for [role]", "candidate assessment for [role]", or when the user asks to evaluate or shortlist applicants.
metadata:
  version: "1.3.0"
  attribution: Adapted from the CV Validation & Candidate Screening skill (MCP Market). Restructured into an interactive Claude Code flow, adapted to produce PIF-styled Word and Excel artifacts, and integrated with our established trigger-preprocessing + MCQ pattern. Standard HR practices (blind review, weighted rubrics, tiered ranking) are common to the field.
---

# HR Candidate Assessment

## Purpose
Screen and rank a set of candidate CVs against a specific role. The skill generates a role-specific scoring rubric, parses each CV, scores every candidate against the rubric using a blind-review protocol, and produces two artifacts: a Word document with the rubric and methodology, and an Excel scorecard ranking candidates into tiers.

---

## Trigger Stencils

Activate on user messages that follow patterns like:
- *"Assess candidates for [role]"*
- *"Screen these CVs for [role]"*
- *"Rank these applicants for [role]"*
- *"Candidate assessment for [role] in [division]"*
- *"Shortlist for [role]"*
- Or any explicit request to evaluate or rank candidates against a role.

---

## Workspace Convention

This skill reads from and writes to a dedicated folder:

**Base path:** `~/HR-Workspace/hr-candidate-assessment/`

**Structure:**
- `inputs/job-descriptions/` — drop the target role's JD file here
- `inputs/cvs/` — drop candidate CV files here
- `outputs/` — where the skill writes the rubric (Word) and scorecard (Excel)

**On first invocation:** create any missing folders **silently** — do NOT announce folder creation to the user.

---

## Chat-Input Mirroring Rule (Standard Behavior)

**Whenever the user provides content in chat — either pasted text or an attached file — always save a copy to the appropriate `inputs/` subfolder before processing.**

- **Text paste:** save as `YYYYMMDD_HHMMSS_[category].txt` in the correct subfolder (e.g., `inputs/job-descriptions/` or `inputs/cvs/`)
- **Attachment:** save the file as-is to the correct subfolder, preserving the original filename
- **Confirmation:** mention the saved path in chat so the user has an audit trail

Example: *"Received the JD. Saved a copy to `~/HR-Workspace/hr-candidate-assessment/inputs/job-descriptions/20260713_101452_pasted-jd.txt`. Now generating the rubric."*

This applies to both JD inputs and CV uploads.

---

## Step 1 — Preprocess: Extract Context Already in the Trigger

Before asking any questions, scan the user's trigger and extract:

- **Role name / title** — always required (e.g., *"Senior Investment Analyst"*)
- **Level / seniority** — often implied by the title (*"Senior"*, *"VP"*, *"Director"*)
- **Division / business unit** — e.g., *"Real Estate"*, *"Investments"*, *"Corporate Functions"*
- **Any JD text pasted in the trigger** — use as the requirements source if provided

**Silent behavior rules:**
- Do NOT post a "Noted: Role = X, Level = Y, Division = Z" confirmation message
- Do NOT announce workspace folder creation
- Do NOT narrate setup steps
- Just extract silently and use the values when MCQs are asked

Then proceed to Step 2 (which only asks for what wasn't extracted).

---

## Step 2 — Ask MCQ Context Questions (via `AskUserQuestion`)

Ask up to 4 questions in a single interactive call. Skip questions for anything already stated in the trigger.

### Question A — Level *(only if not clear from trigger)*
- **header:** `Level`
- **question:** *"What level is this role?"*
- **options (4):**
  - `Analyst / Associate (early career, 0–3 yrs)`
  - `Senior Analyst / Manager (mid-career, 4–8 yrs)`
  - `VP / Director (senior, 8–15 yrs)`
  - `C-suite / Executive (15+ yrs)`

### Question B — Assessment Emphasis *(always ask)*
- **header:** `Emphasis`
- **question:** *"Where should the scoring lean heaviest?"*
- **options (4):**
  - `Technical / functional depth`
  - `Leadership & people impact`
  - `Domain / sector experience`
  - `Balanced across all dimensions`

### Question C — Must-Have Threshold *(always ask)*
- **header:** `Must-haves`
- **question:** *"How strict on must-have requirements?"*
- **options (3):**
  - `Strict — any missing must-have = disqualified`
  - `Moderate — flag missing must-haves, still consider`
  - `Flexible — treat all requirements as scored dimensions`

### Question D — Division *(only if not in trigger)*
- **header:** `Division`
- **question:** *"Which division is this role in?"*
- **options (4):**
  - `Investments Division`
  - `Corporate Functions`
  - `Technology`
  - `Other divisions`

**After answers, confirm briefly:**
> *"Scoring for [Role] · [Level] · [Division] · Emphasis: [X] · Must-haves: [Y]. If you have a job description, paste or attach it next. Otherwise say 'skip' and I'll generate a rubric from the role name and level."*

Then proceed to Step 3.

---

## Step 3 — Optional JD Input (Two Input Modes)

Post this message in chat:

> **Do you have a job description to feed in?** It will make the rubric sharper.
>
> **Option A — Chat paste**
> Paste the JD text directly here, or attach a file (Word `.docx`, PDF `.pdf`, text `.txt`).
>
> **Option B — Drop into workspace folder**
> Place the JD file into:
> `~/HR-Workspace/hr-candidate-assessment/inputs/job-descriptions/`
> Then reply here with "**done**" or "**ready**".
>
> **Option C — Skip**
> Say "**skip**" and I'll generate the rubric from role + level + division alone.

**When the user replies:**
- If they pasted or attached → use that content
- If they said "done" or "ready" → scan `~/HR-Workspace/hr-candidate-assessment/inputs/job-descriptions/` and read the first file present
- If they said "skip" → proceed with role + level + division

Behavior:
- **If JD provided:** parse its must-have, should-have, and nice-to-have requirements; use them to shape the rubric
- **If skipped:** generate the rubric from role + level + division alone, using standard competency assumptions for that level

Then proceed to Step 4.

---

## Step 4 — Generate the Scoring Rubric

Build a rubric with 5 dimensions. Weights adapt to level and emphasis:

### Rubric dimensions

1. **Functional & Technical Fit** — role-specific hard skills, tools, methodologies
2. **Relevant Experience** — years, depth, progression, transaction / project scope
3. **Leadership & Ownership** — scoped to level: for IC roles this measures initiative and cross-functional influence; for management roles it measures team leadership and organizational impact
4. **Domain / Sector Alignment** — industry exposure, sector knowledge (heavier for lateral hires in specialist functions)
5. **Culture & Values Fit** — collaboration signals, values alignment, growth orientation (assessed cautiously — never inferred from protected characteristics)

### Default weights by level and emphasis

**Junior (Analyst):** Technical 40% · Experience 20% · Leadership 10% · Domain 15% · Culture 15%
**Mid (Senior / Manager):** Technical 30% · Experience 25% · Leadership 20% · Domain 15% · Culture 10%
**Senior (VP / Director):** Technical 20% · Experience 25% · Leadership 30% · Domain 15% · Culture 10%
**Executive:** Technical 15% · Experience 20% · Leadership 40% · Domain 15% · Culture 10%

Then apply the emphasis modifier:
- **Technical / functional depth** → +10 to Technical, −5 to Leadership, −5 to Domain
- **Leadership & people impact** → +10 to Leadership, −5 to Technical, −5 to Culture
- **Domain / sector experience** → +10 to Domain, −5 to Technical, −5 to Culture
- **Balanced** → no adjustment

### Must-have requirements

Extract from the JD if provided, or generate a minimum viable set based on role and level. Categorize per user's threshold choice:
- **Strict:** must-haves are pass/fail — any miss disqualifies the candidate
- **Moderate:** must-haves are pass/fail but flagged rather than auto-rejected
- **Flexible:** must-haves become weighted scoring dimensions

Confirm the rubric with the user briefly before moving on:
> *"Rubric ready: 5 dimensions with [level-adjusted weights], [N] must-have requirements. Now upload the candidate CVs you want to evaluate."*

---

## Step 5 — Collect Candidate CVs (Two Input Modes)

Post this message in chat:

> **Please provide the candidate CVs to evaluate.**
>
> Accepted formats: Word (`.docx`), PDF (`.pdf`), or plain text (`.txt`).
>
> **Option A — Chat upload**
> Attach or paste CVs directly in this chat.
>
> **Option B — Drop into workspace folder**
> Place all CV files into:
> `~/HR-Workspace/hr-candidate-assessment/inputs/cvs/`
> Then reply here with "**done**" or "**ready**".
>
> Batch size: 3–20 candidates works best. For very small batches, ranking still applies but statistical patterns won't be reliable.

**When the user replies:**
- If they attached files in chat → use those
- If they said "done" or "ready" → scan `~/HR-Workspace/hr-candidate-assessment/inputs/cvs/` and read every file present
- If both modes were used, combine them

For each file: invoke the `pdf` skill (for PDFs) or `docx` skill (for Word) to extract text.

---

## Step 6 — Score Each Candidate with Blind Review

For every CV:

### 6.1 Extract structured data
Parse into: education, work experience, skills, certifications, projects/publications, and identifiers.

### 6.2 Blind-review separation
**Strip identifying information into a separate lookup** — name, email, phone, physical addresses, photos, dates of birth, and any language signaling gender, ethnicity, nationality, religion, marital status, disability, or age.

Replace with anonymized IDs (**C-01, C-02, C-03, …**) for the entire scoring phase.

### 6.3 Score each dimension (0–100)
For every candidate on every dimension:
- **Cite specific evidence from the CV** — no score is valid without a quote or specific reference
- **No inference of protected characteristics** — do not consider or reference age, gender, ethnicity, nationality, religion, disability, or marital status
- **No score without evidence** — if the CV doesn't show it, the dimension scores 0–30 (rather than guessing high)

### 6.4 Aggregate score
Apply the weights from Step 4 to produce a composite score (0–100).

### 6.5 Apply must-have gate
Depending on the user's threshold choice:
- **Strict:** if any must-have fails, tier the candidate as "Not moving forward" regardless of composite score
- **Moderate:** flag the miss in the notes, keep the composite score
- **Flexible:** must-haves were already scored as dimensions

### 6.6 Detect informational red flags
Flag (but do not auto-penalize):
- Employment gaps longer than 12 months without explanation
- Chronological or credential inconsistencies (e.g., experience with a technology released after claimed start date)
- Vague accomplishment claims lacking specifics or metrics
- Title regressions without context

Flags are informational — final judgment sits with the recruiter.

### 6.7 Reunite identity with scores
Only after all scoring is complete: rejoin each candidate's real identifiers to their score record for the final output.

---

## Step 7 — Rank into Tiers

Assign each candidate to one of three tiers based on composite score and must-have status:

| Tier | Composite Score | Must-Haves | Recommendation |
|---|---|---|---|
| **Advance to interview** | ≥ 75 | All met | Move to interview stage |
| **Second-look pool** | 55–74 | All met (or moderate threshold with flags) | Review with hiring manager before deciding |
| **Not moving forward** | < 55, or must-have failure under strict threshold | — | Do not advance in this cycle |

Compute batch-level statistics:
- Distribution across tiers
- Most common skill gaps across the candidate pool
- Average score per dimension

---

## Step 8 — Produce the Artifacts (PIF-Styled)

### Artifact 1 — Word Document: Assessment Rubric and Methodology

Invoke the `docx` skill.

**Structure:**
1. **Header** — *"Candidate Assessment Rubric — [Role Name]"* (20pt PIF Green `005C4D` bold, Fund Light with Calibri fallback)
2. **Subtitle** — *"[Division] · [Level] · PIF"* (12pt gray `595959`)
3. **Horizontal green divider**
4. **Role Summary** — 1 short paragraph (from Step 1–3 inputs)
5. **Rubric Dimensions** — table with Dimension, Weight, and Description columns; header row in PIF Green with white text
6. **Must-Have Requirements** — bulleted list, framed by the chosen threshold
7. **Tier Definitions** — small table with Tier, Score Range, Recommendation
8. **Blind Review Protocol** — 1 paragraph explaining how identifiers were separated
9. **Batch Summary** — total candidates, distribution across tiers, common gaps
10. **Footer** — *"Confidential — HR use only"* soft gray `9A9A9A` 8pt right-aligned

### Artifact 2 — Excel Scorecard: Ranked Candidate List

Invoke the `xlsx` skill.

**Sheet 1 — Ranked Scorecard**

Columns (in order):
1. Rank
2. Candidate ID (real name after reunite)
3. Tier
4. Composite Score
5. Technical Fit (weighted)
6. Experience (weighted)
7. Leadership (weighted)
8. Domain (weighted)
9. Culture (weighted)
10. Must-Have Status (Pass / Fail with note)
11. Red Flags (list or "None")
12. Key Strengths (2–3 short phrases)
13. Key Gaps (2–3 short phrases)
14. Recommendation

**Styling:**
- Header row: PIF Green `005C4D` fill, white bold text
- Body: alternating white and light gray `F2F2F2`
- Tier cells color-coded:
  - Advance to interview → PIF Green fill (`005C4D`), white text
  - Second-look pool → Tan fill (`C4984F`), white text
  - Not moving forward → Soft gray fill (`9A9A9A`), white text
- Composite Score column: conditional formatting bar chart (0–100)
- Freeze top row

**Sheet 2 — Per-Candidate Evidence**

For each candidate: a small structured block with per-dimension score AND the specific CV quote or reference that supports that score. This is the auditable evidence trail.

### File location and naming
Save both files to:
`~/HR-Workspace/hr-candidate-assessment/outputs/`

- `YYYYMMDD_Assessment_Rubric_[Role_Slug].docx`
- `YYYYMMDD_Scorecard_[Role_Slug].xlsx`

Example: `~/HR-Workspace/hr-candidate-assessment/outputs/20260712_Assessment_Rubric_Senior_Investment_Analyst.docx`

### Confirmation to user
> *"Done. Two files generated in `~/HR-Workspace/hr-candidate-assessment/outputs/`:*
> *• `[rubric filename]` — the scoring methodology and rubric*
> *• `[scorecard filename]` — the ranked candidate list with per-dimension scores and evidence*
>
> *[N] candidates evaluated: [X] advance to interview, [Y] second-look, [Z] not moving forward."*

---

## Key Principles

- **Every score must have evidence.** No dimension score is valid without a specific citation from the CV.
- **Blind review is non-negotiable.** Identifiers are separated before scoring and never influence dimension scores.
- **No inference of protected characteristics.** Age, gender, ethnicity, nationality, religion, disability, and marital status are never used or inferred.
- **Red flags are informational, not disqualifying.** Employment gaps and other flags surface for recruiter judgment — the skill never auto-rejects on flags.
- **Must-have thresholds are user-controlled.** Strict / moderate / flexible modes are the user's choice, not a default.
- **Batch statistics matter as much as individual scores.** A pool with 80% "Not moving forward" is a sourcing problem, not a candidate problem.

## Limitations

- Cannot fully verify credentials or work history — outputs are decision-support, not decisions.
- Culture/values scoring is directional, not clinical — always sanity-check against a real conversation.
- Very small batches (< 3 CVs) don't reveal patterns; results should be treated as individual notes.
- The skill assumes English-language CVs by default; ask the user if Arabic or bilingual review is needed.
- Not a substitute for hiring manager or legal review, especially for compliance-sensitive elements.

## Example Trigger Prompts

- *"Assess candidates for Senior Investment Analyst, Real Estate"*
- *"Screen these CVs for VP of Product, Technology"*
- *"Rank applicants for Director of Strategy, Corporate Functions"*
- *"Shortlist candidates for the Analyst role I posted last week"*
