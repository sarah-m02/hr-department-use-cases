---
name: hr-employee-retention
description: Analyzes exit interview transcripts to identify why employees are leaving and recommends targeted retention actions. Collects transcripts and context interactively, then produces a PIF-styled Word document (retention report). Trigger phrases include "why are [X] leaving [Y]", "understand why [X] are leaving the [Y] division", "analyze retention in [division]", "retention analysis for [division]", "build a retention report", or when the user asks to analyze exit interviews or diagnose attrition drivers.
metadata:
  version: "1.3.0"
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
- `inputs/exit-interviews/` — where the user drops transcript files
- `outputs/` — where the skill writes the retention report

**On first invocation:** if any of these folders don't exist, create them silently before asking the user for input.

---

## Step 1 — Ask for the Transcripts (Two Input Modes)

Immediately after being triggered, post this message in chat:

> **To produce your retention report, please provide your exit interview transcripts.**
>
> Choose either input mode:
>
> **Option A — Chat paste**
> Paste transcripts directly in this chat (any format: structured Q&A, bullet notes, or free-flowing text).
>
> **Option B — Drop files into the workspace folder**
> Place transcript files (Word `.docx`, PDF `.pdf`, or plain text `.txt`) into:
> `~/HR-Workspace/hr-employee-retention/inputs/exit-interviews/`
> Then reply here with "**done**" or "**ready**".
>
> Guidance:
> - Aim for 3 or more transcripts (fewer produces individual case notes, not organizational patterns)
> - Names or identifying details can be anonymized before sharing
>
> Once received, I'll ask a few short questions to frame the report.

**When the user replies:**
- If they pasted text → use that as the input
- If they said "done" or "ready" → scan `~/HR-Workspace/hr-employee-retention/inputs/exit-interviews/` and read every file present, ingesting each as a transcript
- If both modes were used, combine them If they provide fewer than 3, warn them:
> *"You've shared [N] transcript(s). Pattern analysis is unreliable below 3 — do you want to add more, or should I proceed with a single-case note instead?"*

---

## Step 2 — Preprocess: Reuse Any Context Already Provided

Before invoking `AskUserQuestion`, **scan the user's original trigger message** and the surrounding chat for context that already answers the framing questions. This prevents asking for information the user has already stated.

Explicitly check for:
- **Division / business unit** — e.g., *"Investments Division"*, *"Real Estate"*, *"Corporate Functions"*, *"Technology"* → if mentioned, skip Question 1
- **Time period** — e.g., *"Q1 2026"*, *"last quarter"*, *"first half"*, *"last 6 months"* → if mentioned, skip Question 2
- **Recipient** — e.g., *"for the division head"*, *"for the CHRO"*, *"my HRBP"* → if mentioned, skip Question 3

Confirm what was extracted in a single short message before moving on:

> *"Noted from your request: Division = [X], Time Period = [Y]. I'll ask about the remaining framing detail(s) next."*

If **all three** framing inputs are already stated in the trigger, skip Step 3 entirely and proceed to Step 4 (Analysis) after confirming what was extracted.

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

## Step 4 — Analyze the Transcripts

Apply the following analytical framework. Every claim in the output must trace to a specific quote.

### 4.1 Extract themes and categorize by driver
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

---

## Step 5 — Produce the Artifact (PIF-Styled Word Document)

Invoke the `docx` skill to generate a Word document following this structure and styling.

### Document structure

1. **Header** (top of first page)
   - Title: *"Retention Report — [Division] — [Time Period]"* (20pt, PIF Green `005C4D`, bold, Fund Light with Calibri fallback)
   - Subtitle: *"Exit Interview Analysis and Recommended Actions"* (12pt, gray `595959`)
   - Horizontal line divider in PIF Green

2. **Executive summary** — 1 short paragraph
   Frame: N exits analyzed, K root causes identified, top M actions recommended.

3. **Theme table** — the core artifact
   - Columns: `Theme` | `Driver` | `Frequency` | `Severity` | `Evidence Quote`
   - Header row: PIF Green fill (`005C4D`), white text, bold
   - Body rows: alternating white and light gray (`F2F2F2`)
   - Evidence Quote column: italic, tan (`C4984F`)

4. **Root cause analysis** — 2–3 short paragraphs
   Section heading in PIF Green. Body in text gray (`595959`).

5. **Preserving signals** — bullet list
   Section heading in PIF Green.

6. **Recommended actions** — numbered
   Each action in a light-bordered box (border `9A9A9A`). Action title in bold PIF Green. Rationale, owner, timeline, cost, expected impact as sub-bullets in gray.

7. **Talking points for the recipient** — highlighted callout box (skip this section if the user chose "No specific recipient")
   - Border: PIF Green
   - Background: very light gray `F8F8F8`
   - Text in body gray

8. **Footer**
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

### Confirmation to user
> *"Retention report generated:*
> *`~/HR-Workspace/hr-employee-retention/outputs/[filename]`*
> *Follows PIF visual styling."*

---

## Key Principles

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
