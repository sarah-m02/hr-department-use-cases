---
name: hr-candidate-assessment
description: Assesses candidates against a role by parsing their CVs, scoring against a role-specific rubric, and producing a ranked long-list with per-candidate scorecards. Uses blind review to reduce bias. Produces PIF-styled Word document (rubric + methodology) and Excel scorecard (ranked candidates with per-dimension scores). Trigger phrases include "assess candidates for [role]", "screen CVs for [role]", "rank these candidates for [role]", "candidate assessment for [role]", or when the user asks to evaluate or shortlist applicants.
metadata:
  version: "1.5.0"
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

## Step 8 — Produce the Artifacts via the LOCKED TEMPLATE (`build_assessment.py`)

**Do NOT invoke the `docx` or `xlsx` skills and do NOT hand-generate `python-docx` or `openpyxl` code inside an assessment run.** Every rubric + scorecard from this skill must go through the locked template:

**`~/.claude/skills/hr-candidate-assessment/references/build_assessment.py`**

Same code every run → pixel-identical formatting on both artifacts (fonts, colors, table borders, tier-cell coloring, column widths, evidence-sheet layout). Fund Light font, PIF Green `005C4D`, tan `C4984F`, text gray `595959` all baked into the script.

### How to invoke it

1. Assemble a JSON blob matching the schema below (in memory or as a temp file).
2. Run `py "<path to build_assessment.py>" "<path to json>"` (Windows) — the script reads JSON from the argument or stdin.
3. The script writes both files to `~/HR-Workspace/hr-candidate-assessment/outputs/`:
   - `YYYYMMDD_Assessment_Rubric_<role_slug>.docx`
   - `YYYYMMDD_Scorecard_<role_slug>.xlsx`
4. File-lock retry (`_v2`…`_v9` suffixes) is automatic if either file is open in Word / Excel.

### JSON contract (v1.5.0)

```json
{
  "role": {
    "name": "Senior Analyst",
    "role_slug": "Senior_Analyst",
    "level": "Senior Analyst / Manager (mid-career, 4-8 yrs)",
    "division": "Investments Strategy Division",
    "summary_paragraph": "Role summary + scoring emphasis + must-have threshold, in one paragraph."
  },
  "dimensions": [
    {"name": "Functional & Technical Fit", "weight_pct": 30, "description": "..."},
    {"name": "Relevant Experience",        "weight_pct": 25, "description": "..."},
    {"name": "Leadership & Ownership",     "weight_pct": 20, "description": "..."},
    {"name": "Domain / Sector Alignment",  "weight_pct": 15, "description": "..."},
    {"name": "Culture & Values Fit",       "weight_pct": 10, "description": "..."}
  ],
  "must_haves": {
    "threshold_note": "Threshold: ...",
    "requirements": ["req 1", "req 2", "..."]
  },
  "tiers": [
    {"name": "Advance to interview", "range": "≥ 75, all must-haves met", "recommendation": "..."},
    {"name": "Second-look pool",     "range": "55–74 (...)",              "recommendation": "..."},
    {"name": "Not moving forward",   "range": "< 55, or must-have failure",    "recommendation": "..."}
  ],
  "blind_review_note": "Before scoring, each CV was stripped of identifying information...",
  "batch_summary": ["line 1", "line 2", "line 3"],
  "candidates": [
    {
      "rank": 1,
      "name": "Real Name",
      "tier": "Advance to interview",
      "composite": 85,
      "scores": [
        {"dim_name": "Functional & Technical Fit", "raw": 88, "weighted": 26.4, "evidence": "..."},
        {"dim_name": "Relevant Experience",        "raw": 85, "weighted": 21.25, "evidence": "..."},
        {"dim_name": "Leadership & Ownership",     "raw": 80, "weighted": 16,    "evidence": "..."},
        {"dim_name": "Domain / Sector Alignment",  "raw": 90, "weighted": 13.5,  "evidence": "..."},
        {"dim_name": "Culture & Values Fit",       "raw": 82, "weighted": 8.2,   "evidence": "..."}
      ],
      "must_have_status": "Pass — all must-haves met",
      "red_flags": "None",
      "key_strengths": "...",
      "key_gaps": "...",
      "recommendation": "..."
    }
  ]
}
```

### Rules

- **The JSON is the ONLY thing that varies per run.** Fonts, colors, table borders, column widths, tier-cell coloring — all locked in the script.
- **Do not** write a per-run Python file, embed docx/xlsx code in chat, or invoke the `docx`/`xlsx` skills for the render step.
- **All string fields must be pre-audited by Steps 6–7** before landing in the JSON. Scores must have evidence. Every claim must be traceable.
- **If the script fails** (missing dep, schema mismatch, filesystem error), surface the error and stop — do not fall back to hand-generated code.

### File location and naming (produced by the script)

- `YYYYMMDD_Assessment_Rubric_<role_slug>.docx`
- `YYYYMMDD_Scorecard_<role_slug>.xlsx`

Both saved to `~/HR-Workspace/hr-candidate-assessment/outputs/`.

Example: `~/HR-Workspace/hr-candidate-assessment/outputs/20260716_Assessment_Rubric_Senior_Analyst.docx`

### Closing message to user (concise)

After both files are saved, post **only** a short closing in chat. Do NOT dump the rubric content, per-candidate scores, or the ranked list into chat — the artifacts contain all of it.

**Format:**
> *"Rubric and scorecard created.*
> *[1-line summary — e.g. "N candidates evaluated for [Role]: X advance, Y second-look, Z not moving forward"]*
> *[clickable markdown link to rubric]*
> *[clickable markdown link to scorecard]"*

**Example:**
> *"Rubric and scorecard created.*
> *5 candidates evaluated for Senior Investment Analyst: 2 advance, 2 second-look, 1 not moving forward.*
> *[Open rubric](file:///C:/Users/Almisned%20Sarah/HR-Workspace/hr-candidate-assessment/outputs/20260713_Rubric_Senior_Investment_Analyst.docx)*
> *[Open scorecard](file:///C:/Users/Almisned%20Sarah/HR-Workspace/hr-candidate-assessment/outputs/20260713_Scorecard_Senior_Investment_Analyst.xlsx)"*

**Silent behavior rules for closing:**
- Do NOT list per-dimension scores in chat
- Do NOT paste evidence quotes or the ranked list in chat
- Do NOT explain the rubric methodology in chat
- The artifacts contain everything — the chat closing is just: files created + one-line summary + two clickable paths
- Keep the closing under 5 lines total

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
