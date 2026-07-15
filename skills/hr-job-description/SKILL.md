---
name: hr-job-description
description: Drafts inclusive, bias-audited job descriptions grounded in PIF-specific context (divisions, EVP language, hiring norms) with 30/60/90-day success expectations and must-have vs. nice-to-have qualifications. Uses a structured MCQ intake plus a hiring-manager brief, then benchmarks the role against public peer-organisation JDs before drafting. Produces a PIF-styled Word document. Trigger phrases include "draft a JD for [role]", "write a job description for [role]", "job description for [role] in [division]", "rewrite this JD", or when the user asks to create or refine a job posting.
metadata:
  version: "1.6.0"
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

## Step 0 — Consult the PIF Context Reference (MANDATORY, silent)

Before any other step, silently load and internalize:

**`~/.claude/skills/hr-job-description/references/pif_context.md`**

This file defines PIF's divisions, EVP language, tone words, "flavor" snippets
per division, division-inference hints for the MCQ, and — critically — the
**hard rules** the skill MUST obey (§10 of the reference file). Every
downstream step must respect those hard rules.

Key rules to internalise from the reference (do NOT restate them to the user):

- Never emit specific comp figures, bonus structure, allowance amounts, or
  vacation-day counts.
- Never emit Vision 2030 target percentages, target dates, or the names of
  national programmes (Saudization, Nitaqat, etc.).
- Never name individuals (governor, chair) in the JD body.
- Never quote specific AUM or headcount figures.
- Never assert Arabic as a required, preferred, or optional language — the
  JD stays silent on Arabic for now.
- Never label any MCQ option as "Recommended" — ordering is signal enough.

If the reference file is missing, warn the user in one line and stop — the
skill's PIF-grounding is not optional.

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

## Step 2 — Stage A: Structured MCQ Intake (via `AskUserQuestion`)

Use the `AskUserQuestion` tool to ask short multiple-choice questions in a single call. Do NOT ask in free-text chat. **Never** label any option as "Recommended."

**Always ask** these 4 in a single `AskUserQuestion` call:
- **Team leadership scope** (Question A)
- **Employment type** (Question B)
- **Work arrangement** (Question C)
- **Reports to** (Question F — new in v1.6.0)

**Conditionally add** (up to a total of 4 questions in the call — if adding pushes over 4, drop Work arrangement first and default it to *"On-site (Riyadh)"*):
- **Level** — add as Question D if not clear from the trigger
- **Division** — add as Question E if not in the trigger; use the **Smart Division rule** below to build its options

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

### Question E — Division *(only if not in trigger — uses the Smart Division rule)*

- **header:** `Division`
- **question:** *"Which division is this role in?"*
- **options (4):** built dynamically using §5 of the PIF context reference file
  (`pif_context.md`). Match the role's title / keywords against the
  Division-inference-hints table and populate the first three options with
  the top-guess, second-guess, and third-guess divisions in that order. The
  fourth option is always `Other divisions`. If no keyword rule fires, use
  the fallback pool listed at the end of §5 (Saudi Sector Development ·
  Corporate Communications · Human Capital · Strategy and Insights · Other).
- **multiSelect:** false
- **Do NOT** label any option as "Recommended." Ordering is signal enough.
- **Do NOT** collapse pool names into the old three-bucket taxonomy
  (Investments / Corporate Functions / Technology) — the JD's Role Overview
  depends on knowing the specific PIF pool or function so it can pull the
  right flavor snippet from §9 of the reference file.

### Question F — Reports to *(always ask)*

- **header:** `Reports to`
- **question:** *"Who will this role report to?"*
- **options (4):**
  - `Head / MD of Division or Pool`
  - `VP / Senior VP within the division`
  - `Manager / Team Lead`
  - `Executive leadership (Governor's office / Investment Committee)`
- **multiSelect:** false

**After receiving Stage A answers, briefly confirm:**
> *"Drafting a JD for [Role] · [Level] · [Division] · [Employment] · [Work model] · [Team scope] · reporting to [Reports to]. Two more quick questions on hiring context. One moment."*

Then proceed to Step 2b (Stage B — hiring-manager brief).

---

## Step 2b — Stage B: Hiring-Manager Brief (via `AskUserQuestion`)

The structured MCQs give facts. This stage gives *context* — the difference
between a generic "Senior Analyst" JD and one that reflects what this
specific hire is meant to fix. Use one `AskUserQuestion` call with 2–3
questions. **No "Recommended" labels.**

### Question G — Why is this role opening?

- **header:** `Opening context`
- **question:** *"Why is this role opening now?"*
- **options (4):**
  - `Backfill (replacing a departure)`
  - `Newly created role`
  - `Capability gap on the existing team`
  - `Scale-up / team expansion`
- **multiSelect:** false

### Question H — Biggest problem in year one

- **header:** `Year-one focus`
- **question:** *"What is the single biggest problem this hire is expected to solve in their first year? (Pick the closest — you can also type a free-text answer.)"*
- **options (4):**
  - `Delivering a specific pipeline / programme of work`
  - `Building a capability the team currently lacks`
  - `Owning a stakeholder relationship or portfolio company`
  - `Turning around underperformance or unblocking delivery`
- **multiSelect:** false

### Question I — Key stakeholders

- **header:** `Stakeholders`
- **question:** *"Which stakeholders will this role interface with most?"*
- **options (4):**
  - `Portfolio companies`
  - `Group / executive leadership`
  - `External advisors, regulators, or investors`
  - `Cross-division PIF teams`
- **multiSelect:** false

The free-text response to Question H is high signal — capture it verbatim
and use it in the Role Overview and 30/60/90 sections.

Then proceed to Step 3 (benchmarking).

---

## Step 3 — Role Benchmarking (light, silent)

Before drafting, ground the qualifications and years-of-experience bands in
what peer organisations actually ask for. Use `WebSearch` and (sparingly)
`WebFetch` on public JDs.

**Peer sets by role type:**
- **Investment roles** → ADIA, Mubadala, GIC, Temasek, Khazanah, QIA, ICD Dubai
- **Corporate functions** → sovereign wealth funds and large listed Saudi
  employers (Aramco, SABIC, STC) for baseline norms
- **Technology / AI** → hyperscaler and top-tier tech employers plus HUMAIN-
  adjacent sovereign initiatives
- **Real estate / giga-projects** → Brookfield, Blackstone Real Estate, MASA,
  Emaar, Aldar, Damac; plus ROSHN and Red Sea Global for internal peers

**Budget and guardrails:**
- Hard cap **5 total web calls** per JD (search + fetch combined)
- Silently proceed on failure — never block the JD generation on benchmarking
- Extract only: typical years of experience, typical certifications, typical
  must-have technical skills, one or two lines of tone from peer "what we
  offer" copy (to sanity-check ours doesn't drift into generic marketing)
- Do NOT copy peer JD language verbatim into ours
- Do NOT ingest anything behind a login wall

**Output of Step 3** (held in memory, not shown to the user): a short
benchmarking note used by Step 4 to calibrate quals. Do NOT dump benchmarking
findings into chat.

---

## Step 4 — Generate the JD Content

Produce the JD content in memory (chat gets a brief summary only). Apply the following principles:

### 4.1 Role Overview / Purpose (1–2 short paragraphs)
Explain **why this role exists** and its impact on the business. Lead with the outcome the role delivers, not the tasks it performs. Anchor to the division's mission.

**PIF-context integration (MANDATORY):** consult §9 (per-division flavor snippets) of `references/pif_context.md` for the division selected in Question E. Paraphrase the snippet into the Role Overview — do NOT paste verbatim, and do NOT skip this step. Additionally, weave in Stage B answers: the year-one problem from Question H (verbatim if the user typed a free-text answer) and the stakeholder set from Question I. This is what turns the overview from generic to PIF-specific.

### 4.2 Key Responsibilities (5–8 bullets)
Outcome-oriented, not activity-oriented. Each bullet starts with an action verb and describes a result. Avoid vague phrases like *"handle assigned tasks"* or *"other duties as needed."*

**Tailor to team-leadership scope:**
- Individual contributor → emphasize technical delivery and cross-team collaboration
- Manages team → add people-management responsibilities (coaching, hiring, performance)
- Manages managers → add strategic responsibilities (setting direction, capability building)

### 4.3 Must-Have Qualifications
Only genuinely required to perform the role from day 1. Do NOT inflate — every must-have narrows the applicant pool. Typically 4–6 items: experience level, core technical skills, non-negotiable credentials.

**Calibrate against the Step 3 benchmarking note.** Years-of-experience bands, common certifications (CFA, ACCA, PMP, ICSC, PhD depending on role type), and language of technical skills should mirror what peer organisations ask for — not exceed it.

**Include a management-experience requirement only if the role manages people.**

**Do NOT include a language requirement about Arabic** (see §8 of `pif_context.md` — Arabic handling is under separate testing and the JD stays silent on it). English is implicit and does not need to be listed.

### 4.4 Nice-to-Have Qualifications
Things that would make a candidate stand out but are not required. Typically 3–5 items: adjacent skills, specific industry exposure, advanced degrees, exposure to specific PIF portfolio company sectors (mining, energy, real estate, tech, tourism) where genuinely relevant.

**Do NOT list Arabic** here — even as nice-to-have — until the language-handling review is complete (see §8 of `pif_context.md`).

**Why the split matters:** Research shows underrepresented candidates hesitate to apply when they don't meet every listed requirement. Separating must-have from nice-to-have materially widens the pool.

### 4.5 30/60/90-Day Success Expectations
Concrete, observable outcomes at each milestone:
- **First 30 days** — onboarding, relationships, learning
- **First 60 days** — early contributions, first outputs
- **First 90 days** — measurable impact, independence

Tailor depth to the level (Analyst = ramp-heavy; Director = impact-heavy from day one). Anchor the 90-day milestone to the year-one problem captured in Stage B Question H — the 90-day success should be a plausible first step toward solving it.

### 4.6 Work Details Block
Include a short block near the top with: **Employment type**, **Work arrangement**, **Location** (Riyadh by default), and **Reports to** (if inferrable, else leave placeholder).

### 4.7 What We Offer (1 short paragraph)

Employer value proposition tied to the division and level. What's genuinely differentiated about working here.

**PIF-context integration (MANDATORY):** consult §6 (tone words and EVP) and §9 (per-division flavor snippet) of `references/pif_context.md`. Weave in one or two of the listed tone words and — where genuinely relevant to the level — reference safe programs from §6 (PIF Academy, Graduate Development Program, Portfolio Management Development Program, international-office exposure, portfolio-company secondments).

**Hard prohibitions (from §6 and §10 of the reference file):**
- Never emit specific comp figures, ranges, bonus structure, or allowance amounts
- Never emit Vision 2030 target percentages, target dates, or the names of national programmes (Saudization, Nitaqat, etc.)
- Never quote specific AUM or headcount figures
- Never name individuals (governor, chair)
- Never assert Arabic as a required, preferred, or optional language
- Never use grandiose or generic-marketing phrasing that could apply to any employer

### 4.8 Bias, Inclusivity & PIF-Content Audit

Before finalizing, silently audit the draft for:

**Bias and inclusivity (existing):**
- Gendered language (e.g. *"rockstar,"* *"aggressive,"* *"dominant"*) → replace with neutral alternatives
- Inflated years of experience (e.g. asking for 10 yrs when 5 is sufficient) → adjust
- Credentials that aren't genuinely required (e.g. specific school, MBA-only) → move to nice-to-have or remove
- Cultural assumptions (e.g. *"native English speaker"*) → replace with proficiency-based phrasing

**PIF-content hard-rule audit (v1.6.0):** re-scan the entire draft for anything on the §6 / §10 "Do NOT" lists from `pif_context.md`:
- Any specific SAR/USD figure or comp band → remove
- Any mention of "Saudization," "Nitaqat," or other named national programs → remove
- Any Vision 2030 target percentage or year (e.g. *"70% by 2030"*) → remove
- Any mention of Arabic as a language requirement or preference → remove
- Any named individual (Governor, Chairman, etc.) in the JD body → remove
- Any specific AUM figure (*"$925B"*, etc.) or specific headcount figure → replace with softer scale language
- Any "Recommended" tag on a listed option — should not exist in JD output, but check

If any of these are found, remove the offending line and continue — do not stop, do not surface the audit to the user.

---

## Step 5 — Produce the Artifact (PIF-Styled Word Document)

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

- **PIF-anchored by default.** Role Overview and What We Offer must consult `references/pif_context.md` — a JD that reads like it could apply to any employer is a bug, not a neutral default.
- **Facts over recommendations.** Never label an MCQ option as "Recommended." Order is the only signal — the user knows their own role.
- **Outcome over activity.** Every responsibility should describe a result, not a task.
- **Benchmarked, not invented.** Qualifications and years-of-experience bands are calibrated against public peer-organisation JDs (Step 3), not inferred from principles.
- **Must-have list should be genuinely non-negotiable.** If in doubt, it's a nice-to-have.
- **Inclusive by default.** Bias audit runs on every draft before saving. The audit also covers PIF-content hard rules (comp, national programs, Vision 2030 phrasings, Arabic silence, named individuals).
- **Show the ramp, not just the finish line.** 30/60/90-day expectations set realistic mutual expectations, anchored to the year-one problem from Stage B.
- **Match responsibilities to team-scope.** IC roles emphasize delivery; management roles emphasize people development.
- **Compensation is never included.** Not by default, not on request without explicit hiring-manager sign-off, and never as a range. See §6 of the reference file.
- **The reference file is the source of truth for PIF facts.** Update `pif_context.md` rather than the skill body when PIF divisions, programs, or EVP language change.

## Limitations

- Cannot access real PIF role data — outputs are informed drafts requiring hiring manager review. The `pif_context.md` reference gives PIF-level grounding, not role-specific ground truth.
- Compensation, benefits, and allowance amounts are never generated. Even on user request, do not emit specific figures — direct the user to their reward/HR partner.
- Not a substitute for legal review on protected-class language, country-specific labor requirements, or Vision 2030 phrasings.
- Arabic-language handling is under separate testing — the skill stays silent on Arabic and does not offer a bilingual variant yet.
- Benchmarking is best-effort from public sources within a 5-web-call budget; do not treat the resulting quals as authoritative peer analysis.

## Example Trigger Prompts

- *"Draft a JD for Senior Investment Analyst, Real Estate"*
- *"Job description for VP of Product in Technology"*
- *"Write a JD for Director of Strategy, Corporate Functions"*
- *"Rewrite this JD to be more inclusive: [paste existing JD]"*
