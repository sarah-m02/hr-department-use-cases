---
name: hr-employee-retention
description: Analyzes exit interviews and employee satisfaction surveys to identify why employees are leaving and recommends targeted retention actions. Accepts transcripts, filled-in survey responses (based on the WA State OFM exit interview template), or both. Produces a PIF-styled Word document (retention report) with theme analysis and two distinct quantitative visuals — a workplace-experience matrix (day-to-day experience per respondent) and a departure-reasons chart (why they left). Trigger phrases include "why are [X] leaving [Y]", "understand why [X] are leaving the [Y] division", "analyze retention in [division]", "retention analysis for [division]", "build a retention report", or when the user asks to analyze exit interviews, exit surveys, or diagnose attrition drivers.
metadata:
  version: "1.9.1"
  attribution: Adapted from hr-employee-relations in tuanductran/hr-skills (MIT-licensed), scoped to exit-interview retention analysis and extended with interactive input collection and PIF-styled artifact output.
---

# HR Employee Retention Analysis

## Purpose
Read a batch of exit interview transcripts and produce a structured retention report that identifies *why* employees are actually leaving, ties findings to specific evidence, and recommends ranked actions the business can take to reduce future attrition. Delivers the final report as a PIF-styled Word document.

---

## Trigger Stencils

Activate on user messages that follow patterns like:
- *"I want to understand why [group] are leaving the [division]"*
- *"Why are [role/group] leaving [division]?"*
- *"Analyze retention in [division]"*
- *"Retention analysis for [division]"*
- *"Build a retention report for [division]"*
- Or any explicit request to analyze exit interviews.

---

## Workspace Convention

This skill reads from and writes to a dedicated folder:

**Base path:** `~/HR-Workspace/hr-employee-retention/`

**Structure:**
- `templates/exit_survey_template.doc` — the exit survey template (public domain, adapted from WA State OFM), users can copy and fill it
- `inputs/exit-interviews/` — where the user drops transcript files
- `inputs/survey-data/` — where the user drops filled-in survey responses (Excel, CSV, or Word)
- `outputs/` — where the skill writes the retention report

**On first invocation:** create any missing folders **silently** — do NOT announce folder creation to the user.

---

## Chat-Input Mirroring Rule (Standard Behavior)

**Whenever the user provides content in chat — either pasted text or an attached file — always save a copy to the appropriate `inputs/` subfolder before processing.**

- **Text paste:** save as `YYYYMMDD_HHMMSS_[category].txt` in the correct subfolder (e.g., `inputs/exit-interviews/` or `inputs/survey-data/`)
- **Attachment:** save the file as-is to the correct subfolder, preserving the original filename
- **Confirmation:** mention the saved path in chat so the user has an audit trail

Example: *"Received. Saved a copy to `~/HR-Workspace/hr-employee-retention/inputs/exit-interviews/20260713_101452_interviews.txt`. Analyzing now."*

This applies to every input the user provides, not only via chat — folder drops go straight to the folder, but chat inputs must be mirrored there too.

---

## Step 1 — Ask for Inputs (Concise Version)

Immediately after being triggered, post ONLY this short message in chat. Do NOT say "I'll set up the workspace" or "Created workspace folders" or "Noted from your request" — none of those. Go straight to the input request:

> **Understood. Please provide the following to proceed:**
>
> **1. Exit interview transcripts** — attach to chat or drop into `~/HR-Workspace/hr-employee-retention/inputs/exit-interviews/`
> **2. Exit survey responses** — attach to chat or drop into `~/HR-Workspace/hr-employee-retention/inputs/survey-data/`
>
> At least one input required; both together give the richest analysis. Say "**done**" once completed.
>
> Need a blank survey template? Reply "**generate the template**" and I'll put a copy in your `outputs/` folder.

**Handling the user's response:**

- **If they paste text** → save a copy to the appropriate subfolder (chat-input mirroring rule), then use as input
- **If they said "done" or "ready"** → scan both `inputs/exit-interviews/` and `inputs/survey-data/` and read every file present. **Use the robust scan procedure below — never report empty on the strength of one listing.**
- **If they said "generate the template"** → copy `templates/exit_survey_template.doc` to `outputs/exit_survey_template_[YYYYMMDD].doc` and confirm the path; stop here (user needs to distribute and collect responses before running analysis)
- **If both interviews and survey data are present** → use both

### Robust input-scan procedure (MANDATORY)

Terminal output can be truncated mid-command and a single empty listing is **not** proof that a folder is empty. Before ever telling the user "no files found," do all of the following:

1. **First pass** — list files in each input subfolder directly (`Get-ChildItem <path> -File`).
2. **If the first pass returns empty for either folder**, immediately run a **recursive scan of the entire workspace** (`Get-ChildItem $env:USERPROFILE\HR-Workspace -Recurse -File`). This is cheap and catches truncated output, files placed one level up, or files placed in a legacy sibling folder (e.g. `~/HR-Workspace/exit-surveys-and-interviews/`).
3. **Only if the recursive scan also returns empty** may you report to the user that no inputs were found.
4. **If the recursive scan finds candidate files outside the expected subfolders** (e.g., in a sibling directory), tell the user what you found and ask whether to use those files, rather than silently ignoring them.
5. **If the user pushes back** ("check again", "they're there", "the old ones"), always re-run the recursive scan — do not repeat the failed narrow listing.

**Silent behavior rules for this skill:**
- Do NOT announce workspace folder creation
- Do NOT echo back extracted trigger context ("Noted: Division = X...")
- Do NOT narrate setup steps
- Just ask the user for what's needed, then act If they provide fewer than 3, warn them:
> *"You've shared [N] transcript(s). Pattern analysis is unreliable below 3 — do you want to add more, or should I proceed with a single-case note instead?"*

---

## Step 2 — Preprocess Silently

**Scan the user's original trigger message** for context that already answers framing questions. This happens silently — do NOT echo back what was extracted to the user.

Silently check for:
- **Division / business unit** — e.g., *"Investments Division"*, *"Real Estate"*, *"Corporate Functions"* → skip framing Question 1 if present
- **Time period** — e.g., *"Q1 2026"*, *"last quarter"*, *"first half"* → skip framing Question 2 if present
- **Recipient** — e.g., *"for the division head"*, *"for the CHRO"* → skip framing Question 3 if present

**Do NOT post a "Noted: Division = X" confirmation message.** Just internally hold the extracted values and use them when the framing MCQs are asked (Step 3, after inputs are received).

If **all three** framing inputs are already stated in the trigger, skip Step 3 entirely and proceed to Step 4 (Analysis) once inputs are received.

---

## Step 3 — Gather Any Missing Framing Inputs (via `AskUserQuestion`)

For any framing input NOT already provided in the trigger, use the `AskUserQuestion` tool. Do NOT ask these in free-text chat — use the tool so the user can click through them.

Call `AskUserQuestion` with these three questions:

### Question 1 — Division / Business Unit
- **header:** `Division`
- **question:** *"Which division did these departures come from?"*
- **options (4):**
  - `Investments Division`
  - `Corporate Functions`
  - `Technology`
  - `Other divisions` *(Recommended if unsure — user can type a custom value)*
- **multiSelect:** false

*Meaning:* Which team or function these departing employees worked in. Used to focus the analysis and title the report.

### Question 2 — Time Period Covered
- **header:** `Time period`
- **question:** *"Over what time window did these exits happen?"*
- **options (4):**
  - `Current quarter`
  - `Last quarter`
  - `Last 6 months`
  - `Last 12 months`
- **multiSelect:** false

*Meaning:* When these people left. Anchors the report title and enables trend framing.

### Question 3 — Report Recipient (Optional)
- **header:** `Report for`
- **question:** *"Who is the intended recipient of this report? (Tailors the 'talking points' section — optional)"*
- **options (4):**
  - `Division Head`
  - `CHRO / Head of HR`
  - `HR Business Partner`
  - `No specific recipient`
- **multiSelect:** false

*Meaning:* Who will read the report. If provided, the "talking points" section will be tailored to that audience's decision-making level. If "No specific recipient" is chosen, the talking points are omitted.

**After receiving answers, briefly confirm in chat:**
> *"Analyzing [N] transcripts from [Division] covering [Time Period] — I'll produce a retention report for [Recipient]. One moment."*

Then proceed to Step 3.

---

## Step 4 — Analyze the Inputs

Apply the following analytical framework. Every claim must trace to specific evidence (transcript quote or survey score).

### 4.0 Determine which inputs are present
Before running analysis, check what's available:
- **Transcripts only** → run qualitative analysis (4.1–4.6 below); skip the survey sections in the report
- **Survey data only** → run quantitative analysis (4.7–4.9); the report becomes survey-focused
- **Both** → run all sections — the two quantitative visuals live in their own sections; do NOT combine them into a joint interviews+survey confirmation section

### 4.1 Extract themes and categorize by driver *(if transcripts present)*
Group findings under these standard driver categories (add others only if a genuinely distinct theme emerges):

- **Compensation** — pay level, bonus, equity, benefits
- **Career growth** — promotion opacity, unclear criteria, lack of progression
- **Manager** — availability, quality, continuity, coaching
- **Workload** — hours, unpredictability, burnout
- **Culture** — team dynamics, values fit, psychological safety
- **Leadership / strategy** — trust in direction, confidence in leaders
- **Other** — anything genuinely outside the above

### 4.2 Rank each theme
For every theme:
- **Frequency** — how many of the transcripts raised it
- **Severity** — how central it was to the person's decision to leave (surface complaint vs. root cause)

### 4.3 Distinguish surface complaints from root causes
Cluster related themes and identify the underlying issue. For example, if "comp" and "growth" both surface, the root cause is often *opaque career progression*, not compensation per se. Name the root causes explicitly — this is the most valuable part of the analysis.

### 4.4 Identify preserving signals
Note what departing employees praised. These are things the organization should NOT change while addressing the negatives.

### 4.5 Recommend ranked actions
Produce 3–5 concrete recommendations, ranked by:
- Impact (how many departures it would have prevented)
- Speed (30 / 60 / 90 days to implement)
- Cost (low / medium / high)

Each recommendation must include: owner, timeline, cost bracket, and expected impact.

### 4.6 Draft talking points for the business leader (only if recipient specified)
A 3–5 line narrative the HRBP can bring to the recipient, framing the findings as "N exits, M root causes, K proposed actions."

### 4.7 Compute survey summary metrics *(if survey data present)*
Parse the filled-in exit survey responses (Excel/CSV or extracted from Word). Compute:
- **Overall satisfaction** — mean rating across all respondents
- **eNPS score** — % promoters (9–10) minus % detractors (0–6), if the survey includes a recommend-to-work question
- **Per-driver averages** — mean rating for each of the exit survey dimensions
- **Response count and demographic split** — total respondents, breakdown by tenure or division if the template captured it

### 4.8 Identify weak and strong dimensions
For each driver dimension:
- **Weak** — average < 3.0/5 → flagged as a problem area
- **Neutral** — 3.0–3.9 → context
- **Strong** — ≥ 4.0 → preserve

### 4.9 Extract departure reasons (if the template captures them)
Count the top-selected departure reasons across respondents. Rank highest → lowest. This is a distinct signal from the per-dimension ratings — do NOT collapse the two into a joint table.

---

## Step 5 — Produce the Artifact (PIF-Styled Word Document)

Invoke the `docx` skill to generate a Word document following this structure and styling.

### Document structure

1. **Header** (top of first page)
   - Title: *"Retention Report — [Division] — [Time Period]"* (20pt, PIF Green `005C4D`, bold, Fund Light with Calibri fallback)
   - Subtitle: *"Exit Interview and Survey Analysis with Recommended Actions"* (12pt, gray `595959`)
   - Horizontal line divider in PIF Green

2. **Executive summary** — 1 short paragraph
   Frame: N respondents / interviews analyzed, K root causes identified, top M actions recommended.

The two survey-derived visuals live in **separate sections** and answer different questions. Do NOT cluster them under a single "Survey results" heading, and do NOT introduce any combined interviews-plus-survey confirmation section.

3. **Workplace experience matrix** *(only if survey data present)* — quantitative view of the **day-to-day experience** (what was it like to work here?)
   - Section caption (below heading, small text): *"Each cell shows how one departing employee rated one dimension of their workplace experience (1 = poor, 5 = excellent)."*
   - Format: heatmap / matrix, NOT a bar chart
   - Rows: the 8 survey dimensions (Compensation, Career growth, Manager quality, Workload, Culture, Leadership, Learning and development, Recognition), sorted **by row-average ascending** so the weakest dimension is at the top
   - Columns: one column per respondent (anonymized code, e.g. R1..RN), plus a rightmost `Avg` column showing the row mean
   - Cell fill — the SAME traffic-light rule applies to individual respondent cells AND to the `Avg` column (this is important — average values like 2.2 or 2.3 must render red, not gray):
     - Value **strictly less than 3** → Red `EB466C` (weak — problem area)
     - Value **from 3 up to but not including 4** (i.e. 3 ≤ v < 4) → Gray `9A9A9A` (neutral)
     - Value **4 or higher** (up to 5) → PIF Green `005C4D` (strong)
   - Cell text: the numeric rating (integer for respondent cells, one decimal for `Avg`), white, centered
   - Small in-chart legend showing the three color bands with meanings
   - Generated as 300 DPI PNG via matplotlib, embedded in the Word doc
   - Do NOT show a bar chart of dimension averages — the matrix subsumes it and preserves respondent-level detail (bimodality is important — e.g. manager quality is often a split, not a mean)
   - Do NOT include any overall / mean headline number and do NOT include any meta-commentary about why eNPS or other summary metrics were omitted
   - Do NOT use the word "leaver" or "leavers" anywhere in the section caption, chart labels, or legend — this term is banned across the skill's output. Use "departing employees" or "respondents" instead.

4. **Themes from exit interviews (qualitative signal)** *(only if transcripts present)* — the core qualitative artifact
   - This is the **why they left** table
   - Columns: `Theme` | `Driver` | `Frequency` | `Severity` | `Evidence Quote`
   - Header row: PIF Green fill (`005C4D`), white text, bold
   - Body rows: alternating white and light gray (`F2F2F2`)
   - Evidence Quote column: italic, tan (`C4984F`)

5. **Departure reasons — frequency across respondents** *(only if survey data present and the survey captures reasons)* — quantitative view of the **stated triggers** for leaving; placed directly after the interview themes so the qualitative and quantitative "why they left" evidence sit together
   - Section caption (below heading, small text): *"How often each reason was cited by departing employees. The exit survey asks each respondent to select their top three reasons."*
   - Horizontal bar chart, sorted most-cited to least-cited
   - Top reason (single most-cited) in Tan `C4984F` (focal callout)
   - All other reasons in Gray `9A9A9A` (context)
   - Do NOT use the word "leaver" or "leavers" and do NOT refer to the survey as the "OFM survey" — call it the "exit survey" throughout.

6. **Root cause analysis** — 2–3 short paragraphs
   Section heading in PIF Green. Body in text gray (`595959`).

7. **Preserving signals** — bullet list
   Section heading in PIF Green.

8. **Recommended actions** — numbered
   Each action in a light-bordered box (border `9A9A9A`). Action title in bold PIF Green. Rationale, owner, timeline, cost, expected impact as sub-bullets in gray.

9. **Talking points for the recipient** — highlighted callout box (skip this section if the user chose "No specific recipient")
   - Border: PIF Green
   - Background: very light gray `F8F8F8`
   - Text in body gray

10. **Footer**
    *"Confidential — HR use only"* in soft gray (`9A9A9A`), 8pt, right-aligned.

### Styling specification

| Element | Color | Font | Size |
|---|---|---|---|
| Title | PIF Green `005C4D` | Fund Light (fallback: Calibri) | 20pt bold |
| Section headings | PIF Green `005C4D` | Fund Light | 14pt bold |
| Body text | Text Gray `595959` | Fund Light | 11pt |
| Evidence quotes | Tan `C4984F` | Fund Light | 10pt italic |
| Table header background | PIF Green `005C4D` | — | — |
| Table header text | White `FFFFFF` | Fund Light | 11pt bold |
| Callout box borders | PIF Green `005C4D` | — | — |
| Footer | Soft Gray `9A9A9A` | Fund Light | 8pt |

### File location and naming
Save the file to:
`~/HR-Workspace/hr-employee-retention/outputs/YYYYMMDD_Retention_Report_[Division].docx`

Example: `~/HR-Workspace/hr-employee-retention/outputs/20260712_Retention_Report_Investments.docx`

### File-lock handling (MANDATORY)

If the target `.docx` path is already open in Word, `Document.save()` will raise `PermissionError [Errno 13]`. Do NOT surface this raw error to the user. Instead:

1. Wrap the `doc.save(path)` call in a try/except that catches `PermissionError`.
2. On failure, retry automatically with a numeric suffix: `..._Retention_Report_[Division]_v2.docx`, then `_v3`, up to `_v9`.
3. In the closing message, note the suffix used: *"Report created (saved as ...\_v2.docx because the previous version was open in Word)."*
4. Only if all suffix retries fail (extremely unlikely) surface the underlying error and ask the user to close Word.

This must be built into every generated report script — do not rely on the user closing the file first.

### Closing message to user (concise)

After the file is saved, post **only** a short 3-line closing in chat. Do NOT dump the report content into chat — the doc contains it all.

**Format:**
> *"Report created.*
> *[1-line summary — e.g. "6 exits analyzed, 3 confirmed drivers, 3 recommended actions"]*
> *[clickable markdown link to the file]"*

**Example:**
> *"Report created.*
> *6 exits analyzed from Real Estate Investments (Q2 2026), 3 root causes, 5 recommended actions.*
> *[Open report](file:///C:/Users/Almisned%20Sarah/HR-Workspace/hr-employee-retention/outputs/20260713_Retention_Report_RealEstateInvestments.docx)"*

**Silent behavior rules for closing:**
- Do NOT list "Headline findings" in chat
- Do NOT list "Root causes" in chat
- Do NOT list "Top actions" in chat
- Do NOT include tables, bullet lists of drivers, or analysis excerpts in chat
- The doc contains all findings — the chat closing is just: doc created + one-line summary + clickable path
- Keep the closing under 4 lines total

---

## Key Principles

- **Terminology rules (MANDATORY).** In every generated artifact — report body, chart captions, chart labels, section headings, talking points, closing chat message — obey these:
  - Never use the word "leaver" or "leavers." Use "departing employees" for narrative and "respondents" for survey-count contexts.
  - Never refer to the survey as the "OFM survey," the "OFM exit survey," or "WA State OFM." Call it the "exit survey." The template's provenance can be acknowledged in workspace/README files but not in the deliverable.
- **Every claim must be traceable.** No assertion in the report exists without a quote from the transcripts.
- **Distinguish patterns from anecdotes.** If only 1 of 5 transcripts mentions something, flag it as a single-instance signal, not a theme.
- **Root causes over surface complaints.** Cluster analysis is the value — comp complaints and growth complaints often point to the same underlying issue.
- **Preserve what's working.** Departing employees often praise real strengths. Highlight these so the organization doesn't break them while fixing the negatives.
- **Actionable, not aspirational.** Every recommendation must have a named owner, timeline, and cost estimate.
- **Respect confidentiality.** Never quote anything that could re-identify a specific employee beyond the group being analyzed.

## Limitations

- Single-transcript inputs produce individual case notes, not organizational patterns. Require at least 3 transcripts for pattern analysis.
- Cannot infer beyond what's in the transcripts. If a driver isn't discussed, it can't be diagnosed.
- Not a substitute for professional HR judgment or legal review on sensitive cases (e.g., discrimination signals). Flag sensitive-topic language at the top of the report if detected.

## Example Trigger Prompts

- *"I want to understand why analysts are leaving the Real Estate Investments division"*
- *"Why are our senior analysts leaving?"*
- *"Analyze retention in the Investments Division"*
- *"Build a retention report from Q1 exit interviews"*
