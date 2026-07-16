---
name: hr-job-description
description: Drafts inclusive, bias-audited job descriptions grounded in PIF-specific context (divisions, EVP language, hiring norms) with 30/60/90-day success expectations and must-have vs. nice-to-have qualifications. Uses a structured MCQ intake plus a hiring-manager brief, then benchmarks the role against public peer-organisation JDs before drafting. Produces a PIF-styled Word document. Trigger phrases include "draft a JD for [role]", "write a job description for [role]", "job description for [role] in [division]", "rewrite this JD", or when the user asks to create or refine a job posting.
metadata:
  version: "1.9.0"
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

## Step 0 — Consult the PIF Glossary (MANDATORY, silent)

Before any other step, silently load and internalize:

**`~/.claude/skills/hr-job-description/references/pif_context.md`**

**Treat this file as a strict GLOSSARY, not a description.** Every PIF-specific
term in the JD (division name, portfolio name, ecosystem name, program name,
tagline, EVP phrase) must appear in that glossary. If a PIF-specific term
is needed and it is not in the glossary → **do NOT invent one**. Either
ask the user to provide it (via the AskUserQuestion "Other" free-text
fallback) or omit the reference entirely. This is the single most important
rule in v1.7.0.

Key hard rules to internalise from the glossary (do NOT restate them to
the user):

- Never emit specific comp figures, bonus structure, allowance amounts, or
  vacation-day counts.
- Never emit Vision 2030 target percentages, target dates, or the names of
  Saudi national programmes (Saudization, Nitaqat, azm, HRDF, etc.).
- Never name individuals (Governor, Chairman, Deputy Governor, division heads)
  in the JD body.
- Never quote specific AUM or headcount figures.
- Never assert Arabic as a required, preferred, or optional language.
- Never label any MCQ option as "Recommended" — ordering is signal enough.
- Never use the word "pool" — PIF's own word is "portfolio."
- Never use the historical six-pool taxonomy ("Saudi Equity Holdings,"
  "Saudi Sector Development," etc.) in a 2026+ JD — the current structure
  is the three-portfolio framework (Vision / Strategic / Financial).
- Do NOT invent a department, division, sector, section, or team name
  beyond what glossary §4 and §5 list. Many corporate functions ARE now
  verified in §5 (Human Capital, Corporate Communications, Legal, Risk,
  Compliance, Financial Accounting & Control Department, Investments
  Strategy Division, People Development Department, PMO Governance and
  Quality Team, PPA Department, Governor's Office, Shared Services, and
  others) — use them verbatim when the role fits. If a needed unit is NOT
  in §4/§5, do not invent it: ask the user via the "Other" free-text
  fallback and use their answer verbatim.
- Verbatim names of the six Vision Portfolio ecosystems (glossary §3) must
  be preserved exactly — no shortening, reordering, or renaming.
- PIF's org hierarchy has five tiers (glossary §6): Portfolio → Ecosystem →
  Sector → Division/Department → Section/Team. Anchor the JD at the most
  specific verified tier available.

If the glossary file is missing or empty, stop and tell the user in one
line — the skill's PIF-grounding is not optional.

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

### Question E — Portfolio, division, or team *(only if not in trigger — uses the glossary-driven Smart Division rule)*

- **header:** `Where it sits`
- **question:** *"Where does this role sit in PIF?"*
- **options (4):** built dynamically using **§12 of the PIF Glossary
  (`pif_context.md`)**. Match the role's title keywords against the tables
  in §12 (A/B/C/D) and populate the four options in the order the glossary
  prescribes. The fourth option is always `Other`. All options must be
  verbatim glossary terms — do NOT compress or paraphrase names.
- **multiSelect:** false
- **Do NOT** label any option as "Recommended."
- **For corporate-function role keywords** (Legal, Finance, HR, Comms,
  Risk, Compliance, Audit, Technology, Strategy, PMO, etc.), §12 table C
  now supplies verified PIF department names as options (e.g. Human Capital
  for HR-flavored roles; Legal, Risk, Compliance directly). Use these
  verbatim. Only fall back to the "Other" free-text option if none of the
  §12 table C rules match — never invent a department name.
- **Do NOT** use the historical six-pool taxonomy in any 2026+ JD.
- **Do NOT** use the word "pool" — PIF's own word is "portfolio."

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

Explain **why this role exists** and its impact on the business. Lead with the outcome the role delivers, not the tasks it performs.

**PIF-context integration (MANDATORY):** anchor the overview to whatever portfolio, division, or team was selected in Question E, using **only verbatim glossary language** from `references/pif_context.md`:

- If the answer is **Vision Portfolio** or a specific ecosystem → paraphrase the description in glossary §2 and §3
- If **Strategic Portfolio** or **Financial Portfolio** → paraphrase glossary §2
- If a **named investment division** (International Investments Division, MENA Investments Division, Local Real Estate Investment Division, Investments Strategy Division, Chief of Staff Division) → reference the division by its verbatim glossary name (§4)
- If a **verified department or function** from glossary §5 (Human Capital, Corporate Communications, Legal, Risk, Compliance, People Development Department, Financial Accounting & Control Department, PMO Governance and Quality Team, PPA Department, Governor's Office, Shared Services, etc.) → reference it by its verbatim §5 name
- If a **user-specified team** (via "Other" free-text) → use the user's verbatim team name; do NOT map to a guessed department

Additionally weave in Stage B answers: the year-one problem from Question H (verbatim if the user typed free text) and the stakeholder set from Question I. Do NOT paste glossary language verbatim — paraphrase — but do NOT introduce PIF terms that aren't in the glossary.

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

**Do NOT include a language requirement about Arabic** (see §11 of `pif_context.md` — Arabic handling is under separate testing and the JD stays silent on it). English is implicit and does not need to be listed.

### 4.4 Nice-to-Have Qualifications
Things that would make a candidate stand out but are not required. Typically 3–5 items: adjacent skills, specific industry exposure, advanced degrees, exposure to specific PIF portfolio company sectors (mining, energy, real estate, tech, tourism) where genuinely relevant.

**Do NOT list Arabic** here — even as nice-to-have — until the language-handling review is complete (see §11 of `pif_context.md`).

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

Employer value proposition. What's genuinely differentiated about working at PIF.

**PIF-context integration (MANDATORY):** use only verbatim EVP language from glossary §9 (safe to quote or paraphrase). Where genuinely relevant, reference programs from glossary §8:
- **PIF Academy** — safe to mention for any level
- **Graduate Development Program (GDP)** — mention ONLY for entry-level roles that are Saudi-national-eligible; do NOT mention in senior or expat JDs
- Do NOT mention "Faces of PIF," "azm," or "PIF Private Sector Hub" in a JD (they are marketing/programme assets, not role benefits)

**Hard prohibitions (from glossary §10 and §11):**
- Never emit specific comp figures, ranges, bonus structure, or allowance amounts
- Never emit Vision 2030 target percentages, target dates, or Saudi national programme names (Saudization, Nitaqat, azm, HRDF, etc.)
- Never quote specific AUM or headcount figures
- Never name individuals
- Never assert Arabic as a language requirement
- Never introduce a PIF-specific term that isn't in the glossary

### 4.8 Bias, Inclusivity & PIF-Content Audit

Before finalizing, silently audit the draft for:

**Bias and inclusivity (existing):**
- Gendered language (e.g. *"rockstar,"* *"aggressive,"* *"dominant"*) → replace with neutral alternatives
- Inflated years of experience (e.g. asking for 10 yrs when 5 is sufficient) → adjust
- Credentials that aren't genuinely required (e.g. specific school, MBA-only) → move to nice-to-have or remove
- Cultural assumptions (e.g. *"native English speaker"*) → replace with proficiency-based phrasing

**PIF-content hard-rule audit:** re-scan the entire draft for anything on the §10 / §11 "Do NOT" lists from `pif_context.md`:
- Any specific SAR/USD figure or comp band → remove
- Any mention of "Saudization," "Nitaqat," or other named national programs → remove
- Any Vision 2030 target percentage or year (e.g. *"70% by 2030"*) → remove
- Any mention of Arabic as a language requirement or preference → remove
- Any named individual (Governor, Chairman, etc.) in the JD body → remove
- Any specific AUM figure (*"$925B"*, etc.) or specific headcount figure → replace with softer scale language
- Any "Recommended" tag on a listed option — should not exist in JD output, but check

If any of these are found, remove the offending line and continue — do not stop, do not surface the audit to the user.

---

## Step 5 — Produce the Artifact via the LOCKED TEMPLATE (`build_jd.py`)

**Do NOT invoke the `docx` skill and do NOT hand-generate `python-docx` code inside a JD run.** Every JD from this skill must go through the locked template:

**`~/.claude/skills/hr-job-description/references/build_jd.py`**

Same code every run → pixel-identical formatting (fonts, colors, table widths, callout dimensions, bullet indents, footer position, spacing). Font is Fund Light, PIF Green `005C4D`, tan `C4984F`, text gray `595959` — all baked into the script.

### How to invoke it

1. Assemble a JSON blob matching the schema below (in memory or as a temp file).
2. Run `py "<path to build_jd.py>" "<path to json>"` (Windows) — the script reads JSON from the argument or stdin.
3. The script writes to `~/HR-Workspace/hr-job-description/outputs/YYYYMMDD_JD_<Role_Slug>.docx` and automatically retries with `_v2..._v9` suffixes if the target is open in Word.

### JSON contract (v1.9.0)

```json
{
  "role": {
    "name": "Senior Analyst",
    "level": "Senior",
    "portfolio_or_division": "Investments Strategy Division",
    "employment_type": "Full-time (permanent)",
    "work_arrangement": "On-site (Riyadh)",
    "location": "Riyadh",
    "reports_to": "Head of Investments Strategy Division"
  },
  "role_overview": ["paragraph 1", "paragraph 2"],
  "responsibilities": ["bullet 1", "bullet 2", "..."],
  "must_haves": ["bullet 1", "bullet 2", "..."],
  "nice_to_haves": ["bullet 1", "bullet 2", "..."],
  "day_30": ["bullet 1", "bullet 2"],
  "day_60": ["bullet 1", "bullet 2"],
  "day_90": ["bullet 1", "bullet 2"],
  "what_we_offer": "paragraph"
}
```

All string fields must be **pre-audited** by Step 4.8 before being placed in the JSON — no comp figures, no national programme names, no Vision 2030 phrasings, no invented department names, no Arabic language mention.

### Rules

- **The JSON is the ONLY thing that varies per JD.** Fonts, colors, table widths, callout dimensions, spacing — all locked in the script. Do NOT edit `build_jd.py` per JD. If a genuine template change is needed, edit the script (bumping the skill version) — never inline.
- **Do not** write a per-JD Python file. Do not embed docx code in chat. Do not invoke the `docx` skill for the JD render step.
- **If the script fails** (missing dependency, JSON schema mismatch, filesystem error), surface the error to the user and stop — do not fall back to hand-generated docx.

### Document structure

1. **Header** (top of first page)
   - Title: *"Job Description — [Role Name]"* (20pt, PIF Green `005C4D`, bold, Fund Light with Calibri fallback)
   - Subtitle: *"[Portfolio / Division / Team] · [Level] · PIF"* (12pt, gray `595959`) — use the verbatim glossary term from the Question E answer
   - Horizontal line divider in PIF Green

2. **Work Details Block** — small table
   - Employment type · Work arrangement · Location · Reports to
   - Fills in from the MCQ answers

3. **Role Overview** — heading in PIF Green, 1–2 paragraphs in body gray

4. **Key Responsibilities** — heading in PIF Green, bulleted list

5. **Must-Have Qualifications** — heading in PIF Green, bulleted list

6. **Nice-to-Have Qualifications** — heading in PIF Green, bulleted list styled inside a tan-bordered callout box (`C4984F` border) — **the callout box MUST be built exactly per the callout-box spec in §Styling below**

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

### Nice-to-Have callout box — exact structure (v1.7.0, MANDATORY)

The v1.6.0 output had a bug where the first bullet in the Nice-to-Have callout sat outside the tan border with a different indent from the others. Root cause: mixed paragraph styles or a stray paragraph outside the table cell. The build code MUST enforce all of the following:

1. The callout is a **single 1×1 table** with a tan (`C4984F`) 1.5pt border on all four sides
2. Every bullet is a paragraph **inside** that single cell — nothing appears before or after the cell
3. Every bullet paragraph uses the **same** style (`List Bullet`) and the **same** `paragraph_format.left_indent` value (recommend `Cm(0.4)`), explicitly set on every paragraph, not inherited
4. The first paragraph inside the cell is a bullet — do NOT emit a blank paragraph or a plain paragraph before the first bullet
5. Verify visually after generation: all bullets align on the same left edge, all sit *inside* the tan border. If they don't, the build script is wrong and must be fixed before shipping.

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
- **Compensation is never included.** Not by default, not on request without explicit hiring-manager sign-off, and never as a range. See §11 of the reference file.
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
