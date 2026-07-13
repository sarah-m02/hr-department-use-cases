---
name: hr-job-description
description: Drafts inclusive, bias-audited job descriptions with 30/60/90-day success expectations and must-have vs. nice-to-have qualifications. Uses short MCQ context questions to gather any missing details, then produces a PIF-styled Word document. Trigger phrases include "draft a JD for [role]", "write a job description for [role]", "job description for [role] in [division]", "rewrite this JD", or when the user asks to create or refine a job posting.
metadata:
  version: "1.5.0"
  attribution: Adapted from hr-job-description in tuanductran/hr-skills (MIT-licensed), extended with trigger-context preprocessing, MCQ context gathering, and PIF-styled Word artifact output.
---

# HR Job Description Drafting

## Purpose
Turn a short role brief (e.g. *"Senior Investment Analyst, Real Estate"*) into a full, inclusive job description with must-have vs. nice-to-have qualifications and 30/60/90-day success expectations. Delivers the final JD as a PIF-styled Word document.

---

## Trigger Stencils

Activate on user messages that follow patterns like:
- *"Draft a JD for [role]"*
- *"Write a job description for [role]"*
- *"Job description for [role] in [division]"*
- *"Create a JD for [role]"*
- *"Rewrite this JD:"* (followed by an existing description in the same message)
- Or any explicit request to create or refine a job posting.

---

## Workspace Convention

This skill reads from and writes to a dedicated folder:

**Base path:** `~/HR-Workspace/hr-job-description/`

**Structure:**
- `inputs/existing-jds/` — optional; drop an existing JD file here if you want it rewritten
- `outputs/` — where the skill writes the generated JD

**On first invocation:** create any missing folders **silently** — do NOT announce folder creation to the user.

---

## Chat-Input Mirroring Rule (Standard Behavior)

**Whenever the user provides content in chat — either pasted text or an attached file — always save a copy to the appropriate `inputs/` subfolder before processing.**

- **Text paste:** save as `YYYYMMDD_HHMMSS_[category].txt` in the correct subfolder (e.g., `inputs/existing-jds/`)
- **Attachment:** save the file as-is to the correct subfolder, preserving the original filename
- **Confirmation:** mention the saved path in chat so the user has an audit trail

Example: *"Received the JD. Saved a copy to `~/HR-Workspace/hr-job-description/inputs/existing-jds/20260713_101452_existing-jd.txt`. Generating the rewrite now."*

---

## Step 1 — Preprocess: Extract Context Already in the Trigger

**Before asking any questions**, scan the user's trigger message and extract:

- **Role name / title** — always required, usually in the trigger (e.g. *"Senior Investment Analyst"*)
- **Level / seniority** — often implied by the title (e.g. *"Senior"*, *"VP"*, *"Director"*, *"Manager"*, *"Head of"*)
- **Division / business unit** — e.g. *"Real Estate"*, *"Investments"*, *"Corporate Functions"*, *"Technology"*
- **Existing JD pasted?** — if the trigger contains a full JD, treat this as a rewrite task and use the paste as the starting point
- **Existing JD in workspace folder?** — check `~/HR-Workspace/hr-job-description/inputs/existing-jds/`; if any files are present, offer to use them as the rewrite starting point

**Silent behavior rules:**
- Do NOT post a "Noted: Role = X, Level = Y, Division = Z" confirmation message
- Do NOT announce workspace folder creation
- Do NOT narrate setup steps
- Just extract silently and use the values when MCQs are asked

Then proceed to Step 2 (which only asks for what wasn't extracted).

---

## Step 2 — Ask 3–4 MCQ Context Questions (via `AskUserQuestion`)

Use the `AskUserQuestion` tool to ask short multiple-choice questions in a single call. Do NOT ask in free-text chat.

**Rules for which questions to ask:**
- Always ask **Team leadership**, **Employment type**, and **Work arrangement** — these are the 3 highest-signal inputs the trigger usually doesn't include
- If **Level** was NOT clear from the trigger, add it as a 4th question and drop **Work arrangement** (default it to *"On-site (Riyadh)"*)
- If **Division** was NOT in the trigger, add it as a 4th question and drop **Employment type** (default it to *"Full-time (permanent)"*)
- Cap the total at 4 questions

### Question A — Team leadership scope *(always ask)*
- **header:** `Team scope`
- **question:** *"How much people leadership does this role involve?"*
- **options (4):**
  - `Individual contributor (no direct reports)`
  - `Manages a small team (2–4 people)`
  - `Manages a large team (5+ people)`
  - `Manages managers / leads a department`
- **multiSelect:** false

### Question B — Employment type *(always ask unless dropped)*
- **header:** `Employment`
- **question:** *"What is the employment type for this role?"*
- **options (4):**
  - `Full-time (permanent)`
  - `Contract / fixed-term`
  - `Part-time`
  - `Secondment / internal transfer`
- **multiSelect:** false

### Question C — Work arrangement *(always ask unless dropped)*
- **header:** `Work model`
- **question:** *"What's the work arrangement?"*
- **options (4):**
  - `On-site (Riyadh)`
  - `Hybrid (mix of on-site and remote)`
  - `Remote (fully remote)`
  - `Flexible / to be discussed`
- **multiSelect:** false

### Question D — Level *(only if not clear from trigger)*
- **header:** `Level`
- **question:** *"What level is this role?"*
- **options (4):**
  - `Analyst / Associate (early career, 0–3 yrs)`
  - `Senior Analyst / Manager (mid-career, 4–8 yrs)`
  - `VP / Director (senior, 8–15 yrs)`
  - `C-suite / Executive (15+ yrs)`
- **multiSelect:** false

### Question E — Division *(only if not in trigger)*
- **header:** `Division`
- **question:** *"Which division is this role in?"*
- **options (4):**
  - `Investments Division`
  - `Corporate Functions`
  - `Technology`
  - `Other divisions`
- **multiSelect:** false

**After receiving answers, briefly confirm:**
> *"Drafting a JD for [Role] · [Level] · [Division] · [Employment] · [Work model] · [Team scope]. One moment."*

Then proceed to Step 3.

---

## Step 3 — Generate the JD Content

Produce the JD content in memory (chat gets a brief summary only). Apply the following principles:

### 3.1 Role Overview / Purpose (1–2 short paragraphs)
Explain **why this role exists** and its impact on the business. Lead with the outcome the role delivers, not the tasks it performs. Anchor to the division's mission.

### 3.2 Key Responsibilities (5–8 bullets)
Outcome-oriented, not activity-oriented. Each bullet starts with an action verb and describes a result. Avoid vague phrases like *"handle assigned tasks"* or *"other duties as needed."*

**Tailor to team-leadership scope:**
- Individual contributor → emphasize technical delivery and cross-team collaboration
- Manages team → add people-management responsibilities (coaching, hiring, performance)
- Manages managers → add strategic responsibilities (setting direction, capability building)

### 3.3 Must-Have Qualifications
Only genuinely required to perform the role from day 1. Do NOT inflate — every must-have narrows the applicant pool. Typically 4–6 items: experience level, core technical skills, non-negotiable credentials, and language requirements.

**Include a management-experience requirement only if the role manages people.**

### 3.4 Nice-to-Have Qualifications
Things that would make a candidate stand out but are not required. Typically 3–5 items: adjacent skills, secondary languages, specific industry exposure, advanced degrees.

**Why the split matters:** Research shows underrepresented candidates hesitate to apply when they don't meet every listed requirement. Separating must-have from nice-to-have materially widens the pool.

### 3.5 30/60/90-Day Success Expectations
Concrete, observable outcomes at each milestone:
- **First 30 days** — onboarding, relationships, learning
- **First 60 days** — early contributions, first outputs
- **First 90 days** — measurable impact, independence

Tailor depth to the level (Analyst = ramp-heavy; Director = impact-heavy from day one).

### 3.6 Work Details Block
Include a short block near the top with: **Employment type**, **Work arrangement**, **Location** (Riyadh by default), and **Reports to** (if inferrable, else leave placeholder).

### 3.7 What We Offer (1 short paragraph)
Employer value proposition tied to the division. What's genuinely differentiated about working here at this level.

### 3.8 Bias & Inclusivity Audit
Before finalizing, silently audit the draft for:
- Gendered language (e.g. *"rockstar,"* *"aggressive,"* *"dominant"*) → replace with neutral alternatives
- Inflated years of experience (e.g. asking for 10 yrs when 5 is sufficient) → adjust
- Credentials that aren't genuinely required (e.g. specific school, MBA-only) → move to nice-to-have or remove
- Cultural assumptions (e.g. *"native English speaker"*) → replace with proficiency-based phrasing

---

## Step 4 — Produce the Artifact (PIF-Styled Word Document)

Invoke the `docx` skill to generate a Word document following this structure and styling.

### Document structure

1. **Header** (top of first page)
   - Title: *"Job Description — [Role Name]"* (20pt, PIF Green `005C4D`, bold, Fund Light with Calibri fallback)
   - Subtitle: *"[Division] · [Level] · PIF"* (12pt, gray `595959`)
   - Horizontal line divider in PIF Green

2. **Work Details Block** — small table near the top
   - Employment type · Work arrangement · Location · Reports to
   - Fills in from the MCQ answers

3. **Role Overview** — heading in PIF Green, 1–2 paragraphs in body gray

4. **Key Responsibilities** — heading in PIF Green, bulleted list

5. **Must-Have Qualifications** — heading in PIF Green, bulleted list

6. **Nice-to-Have Qualifications** — heading in PIF Green, bulleted list styled inside a tan-bordered callout box (`C4984F` border) to visually distinguish from must-haves

7. **30/60/90-Day Success Expectations** — 3-column table
   - Header row: PIF Green fill, white text (*"First 30 Days"* / *"First 60 Days"* / *"First 90 Days"*)
   - Body rows: alternating white and light gray (`F2F2F2`)

8. **What We Offer** — heading in PIF Green, 1 short paragraph

9. **Footer** — *"PIF Talent Acquisition"* in soft gray (`9A9A9A`), 8pt, right-aligned

### Styling specification

| Element | Color | Font | Size |
|---|---|---|---|
| Title | PIF Green `005C4D` | Fund Light (fallback: Calibri) | 20pt bold |
| Section headings | PIF Green `005C4D` | Fund Light | 14pt bold |
| Body text | Text Gray `595959` | Fund Light | 11pt |
| Nice-to-have callout border | Tan `C4984F` | — | — |
| Table header background | PIF Green `005C4D` | — | — |
| Table header text | White `FFFFFF` | Fund Light | 11pt bold |
| Footer | Soft Gray `9A9A9A` | Fund Light | 8pt |

### File location and naming
Save the file to:
`~/HR-Workspace/hr-job-description/outputs/YYYYMMDD_JD_[Role_Slug].docx`

Example: `~/HR-Workspace/hr-job-description/outputs/20260712_JD_Senior_Investment_Analyst.docx`

### Closing message to user (concise)

After the file is saved, post **only** a short 3-line closing in chat. Do NOT dump the JD content into chat — the doc contains it all.

**Format:**
> *"JD created.*
> *[1-line summary — e.g. "Senior Investment Analyst role for Real Estate, Riyadh-based, full-time"]*
> *[clickable markdown link to the file]"*

**Example:**
> *"JD created.*
> *Senior Investment Analyst, Real Estate Investments, full-time on-site Riyadh.*
> *[Open JD](file:///C:/Users/Almisned%20Sarah/HR-Workspace/hr-job-description/outputs/20260713_JD_Senior_Investment_Analyst.docx)"*

**Silent behavior rules for closing:**
- Do NOT paste the JD text into chat
- Do NOT list the responsibilities, qualifications, or 30/60/90 in chat
- Do NOT explain the bias audit in chat
- The doc contains everything — the chat closing is just: doc created + one-line summary + clickable path
- Keep the closing under 4 lines total

---

## Key Principles

- **Outcome over activity.** Every responsibility should describe a result, not a task.
- **Must-have list should be genuinely non-negotiable.** If in doubt, it's a nice-to-have.
- **Inclusive by default.** Bias audit runs on every draft before saving.
- **Show the ramp, not just the finish line.** 30/60/90-day expectations set realistic mutual expectations.
- **Match responsibilities to team-scope.** IC roles emphasize delivery; management roles emphasize people development.
- **Compensation is not included by default.** If the user wants it, they should specify — many PIF roles keep this off the public JD.

## Limitations

- Cannot access real PIF role data — outputs are template drafts requiring hiring manager review.
- Compensation, benefits, and country-specific contract terms are not generated unless explicitly provided.
- Not a substitute for legal review on protected-class language or country-specific labor requirements.
- Assumes English-language output by default; ask the user if Arabic or another language is required.

## Example Trigger Prompts

- *"Draft a JD for Senior Investment Analyst, Real Estate"*
- *"Job description for VP of Product in Technology"*
- *"Write a JD for Director of Strategy, Corporate Functions"*
- *"Rewrite this JD to be more inclusive: [paste existing JD]"*
