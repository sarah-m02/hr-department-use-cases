---
name: hr-job-description
description: Drafts inclusive, bias-audited job descriptions with 30/60/90-day success expectations and must-have vs. nice-to-have qualifications. Collects role context interactively when needed, then produces a PIF-styled Word document. Trigger phrases include "draft a JD for [role]", "write a job description for [role]", "job description for [role] in [division]", "rewrite this JD", or when the user asks to create or refine a job posting.
metadata:
  version: "1.0.0"
  attribution: Adapted from hr-job-description in tuanductran/hr-skills (MIT-licensed), extended with interactive input collection, trigger-context preprocessing, and PIF-styled Word artifact output.
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
- *"Rewrite this JD:"* (followed by an existing description)
- Or any explicit request to create or refine a job posting.

---

## Step 1 — Optional Context Input

Immediately after being triggered, post this message in chat:

> **I'll draft the job description. If you'd like to give me any additional context (existing JD to modify, specific must-have skills to emphasize, or culture notes to include), paste it here now. Otherwise, just say "skip" or "generate" and I'll produce a first draft from your trigger.**
>
> Accepted input methods:
> - **Paste directly here in chat** — existing JD or free-text context
> - **Attach a document** — Word (`.docx`), PDF (`.pdf`), or plain text (`.txt`)
> - **Say "skip"** — proceed with only the role name from your trigger

Wait for the user's response before moving on. Treat "skip" or "generate" as an explicit signal to proceed without extra context.

---

## Step 2 — Preprocess: Reuse Any Context Already Provided

Before invoking `AskUserQuestion`, **scan the user's original trigger message** for context that already answers the framing questions.

Explicitly check for:
- **Role name / title** — always required, usually in the trigger (e.g. *"Senior Investment Analyst"*)
- **Level / seniority** — often implied by the title (e.g. *"Senior"*, *"VP"*, *"Director"*, *"Manager"*, *"Head of"*) → if clear, skip Question 1
- **Division / business unit** — e.g. *"Real Estate"*, *"Investments"*, *"Corporate Functions"*, *"Technology"* → if mentioned, skip Question 2

Confirm what was extracted in one short message:

> *"Noted from your request: Role = [X], Level = [Y], Division = [Z]. I'll ask about the remaining detail(s) next."*

If **role, level, AND division** are all clear from the trigger, skip Step 3 entirely and proceed to Step 4 (Generate JD content).

---

## Step 3 — Gather Any Missing Framing Inputs (via `AskUserQuestion`)

For any framing input NOT already provided, use the `AskUserQuestion` tool. Do NOT ask in free-text chat.

### Question 1 — Level / Seniority *(only ask if not clear from role name)*
- **header:** `Level`
- **question:** *"What level is this role?"*
- **options (4):**
  - `Analyst / Associate (early career, 0–3 yrs)`
  - `Senior Analyst / Manager (mid-career, 4–8 yrs)`
  - `VP / Director (senior, 8–15 yrs)`
  - `C-suite / Executive (15+ yrs)`
- **multiSelect:** false

### Question 2 — Division / Business Unit *(only ask if not mentioned in trigger)*
- **header:** `Division`
- **question:** *"Which division is this role in?"*
- **options (4):**
  - `Investments Division`
  - `Corporate Functions`
  - `Technology`
  - `Other divisions` *(user can type a custom value)*
- **multiSelect:** false

**After receiving answers, confirm briefly in chat:**
> *"Drafting a JD for [Role] · [Level] · [Division]. One moment."*

Then proceed to Step 4.

---

## Step 4 — Generate the JD Content

Produce the JD content in memory (not in chat — chat will get a brief summary only). Apply the following principles:

### 4.1 Role Overview / Purpose (1–2 short paragraphs)
Explain **why this role exists** and its impact on the business. Lead with the outcome the role delivers, not the tasks it performs. Anchor to the division's mission.

### 4.2 Key Responsibilities (5–8 bullets)
Outcome-oriented, not activity-oriented. Each bullet starts with an action verb and describes a result, not a duty. Avoid vague phrases like *"handle assigned tasks"* or *"other duties as needed."*

### 4.3 Must-Have Qualifications
Only genuinely required to perform the role from day 1. Do NOT inflate — every must-have narrows the applicant pool. Typically 4–6 items covering: experience level, core technical skills, credentials that are genuinely non-negotiable, and language requirements.

### 4.4 Nice-to-Have Qualifications
Things that would make a candidate stand out but are not required. Typically 3–5 items: adjacent skills, secondary languages, specific industry exposure, advanced degrees.

**Why the split matters:** Research shows underrepresented candidates hesitate to apply when they don't meet every listed requirement. Separating must-have from nice-to-have materially widens the pool.

### 4.5 30/60/90-Day Success Expectations
Concrete, observable outcomes at each milestone:
- **First 30 days** — onboarding, relationships, learning
- **First 60 days** — early contributions, first outputs
- **First 90 days** — measurable impact, independence

Tailor the depth to the level (Analyst = ramp-heavy; Director = impact-heavy from day one).

### 4.6 What We Offer (1 short paragraph)
Employer value proposition tied to the division. What's genuinely differentiated about working here at this level.

### 4.7 Bias & Inclusivity Audit
Before finalizing, silently audit the draft for:
- Gendered language (e.g. *"rockstar,"* *"aggressive,"* *"dominant"*) → replace with neutral alternatives
- Inflated years of experience (e.g. asking for 10 yrs when 5 is sufficient) → adjust
- Credentials that aren't genuinely required (e.g. specific school, MBA-only) → move to nice-to-have or remove
- Cultural assumptions (e.g. *"native English speaker"*) → replace with proficiency-based phrasing

---

## Step 5 — Produce the Artifact (PIF-Styled Word Document)

Invoke the `docx` skill to generate a Word document following this structure and styling.

### Document structure

1. **Header** (top of first page)
   - Title: *"Job Description — [Role Name]"* (20pt, PIF Green `005C4D`, bold, Fund Light with Calibri fallback)
   - Subtitle: *"[Division] · [Level] · PIF"* (12pt, gray `595959`)
   - Horizontal line divider in PIF Green

2. **Role Overview** — heading in PIF Green, 1–2 paragraphs in body gray

3. **Key Responsibilities** — heading in PIF Green, bulleted list

4. **Must-Have Qualifications** — heading in PIF Green, bulleted list

5. **Nice-to-Have Qualifications** — heading in PIF Green, bulleted list styled inside a tan-bordered callout box (`C4984F` border) to visually distinguish from must-haves

6. **30/60/90-Day Success Expectations** — 3-column table
   - Header row: PIF Green fill, white text (*"First 30 Days"* / *"First 60 Days"* / *"First 90 Days"*)
   - Body rows: alternating white and light gray (`F2F2F2`)

7. **What We Offer** — heading in PIF Green, 1 short paragraph

8. **Footer** — *"PIF Talent Acquisition"* in soft gray (`9A9A9A`), 8pt, right-aligned

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

### File naming
`YYYYMMDD_JD_[Role_Slug].docx` — e.g., `20260712_JD_Senior_Investment_Analyst.docx`

### Confirmation to user
> *"Job description generated: `[filename]`. Saved to your working directory. Follows PIF visual styling."*

---

## Key Principles

- **Outcome over activity.** Every responsibility should describe a result, not a task.
- **Must-have list should be genuinely non-negotiable.** If in doubt, it's a nice-to-have.
- **Inclusive by default.** Bias audit runs on every draft before saving.
- **Show the ramp, not just the finish line.** 30/60/90-day expectations set realistic mutual expectations.
- **Compensation is not included by default.** If the user wants it in the JD, they should specify — many PIF roles keep this off the public JD.
- **Every JD is a preview, not a wish list.** The role should match what the successful candidate will actually do.

## Limitations

- Cannot access real PIF role data — outputs are template drafts requiring hiring manager review.
- Compensation, benefits, and contract-specific terms are not generated unless explicitly provided.
- Not a substitute for legal review on protected-class language or country-specific labor requirements.
- Assumes English-language output by default; ask the user if Arabic or another language is required.

## Example Trigger Prompts

- *"Draft a JD for Senior Investment Analyst, Real Estate"*
- *"Job description for VP of Product in Technology"*
- *"Write a JD for Director of Strategy, Corporate Functions"*
- *"Rewrite this JD to be more inclusive: [paste existing JD]"*
