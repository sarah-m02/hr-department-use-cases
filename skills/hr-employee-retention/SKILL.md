---
name: hr-employee-retention
description: Analyzes exit interview transcripts to identify why employees are leaving and recommends targeted retention actions. Interactively collects transcripts and context from the user, then produces a PIF-styled Word document (retention report). Use when the user asks to analyze exit interviews, understand attrition drivers, diagnose retention risks, or build a retention plan from employee feedback.
metadata:
  version: "1.0.0"
  attribution: Adapted from hr-employee-relations in tuanductran/hr-skills (MIT-licensed), scoped to exit-interview retention analysis and extended with PIF-styled artifact output.
---

# HR Employee Retention Analysis

## Purpose
Read a batch of exit interview transcripts and produce a structured retention report that identifies *why* employees are actually leaving, ties findings to specific evidence, and recommends ranked actions the business can take to reduce future attrition. Delivers the final report as a PIF-styled Word document.

---

## Step 1 — Collect Inputs from the User

When invoked, ALWAYS ask the user for the required inputs before running any analysis. Do not proceed without them.

Post this exact prompt to the user:

> **To produce your retention report, please provide:**
>
> 1. **Exit interview transcripts** — paste 3 or more (any format works: structured Q&A, bullet notes, or free-flowing text)
> 2. **Business unit / division** — which team or function these departures came from (e.g. *"Real Estate Investments"*)
> 3. **Time period** — the quarter or window covered (e.g. *"Q1 2026"*)
> 4. **[Optional]** Business leader receiving the report (e.g. *"Division Head"*) — used to tailor the "talking points" section
>
> Once you paste these, I'll analyze them and produce the report as a Word document.

Wait for the user's response before proceeding to Step 2.

If the user provides only some inputs, ask for the missing ones. If they provide fewer than 3 transcripts, warn them that pattern analysis becomes unreliable below that threshold and ask whether they want to proceed anyway.

---

## Step 2 — Analyze the Transcripts

Apply the following analytical framework to the provided transcripts. Every claim in the output must trace to a specific quote.

### 2.1 Extract themes and categorize by driver
Group findings under these standard driver categories (add others only if a genuinely distinct theme emerges):

- **Compensation** — pay level, bonus, equity, benefits
- **Career growth** — promotion opacity, unclear criteria, lack of progression
- **Manager** — availability, quality, continuity, coaching
- **Workload** — hours, unpredictability, burnout
- **Culture** — team dynamics, values fit, psychological safety
- **Leadership / strategy** — trust in direction, confidence in leaders
- **Other** — anything genuinely outside the above

### 2.2 Rank each theme
For every theme:
- **Frequency** — how many of the transcripts raised it
- **Severity** — how central it was to the person's decision to leave (surface complaint vs. root cause)

### 2.3 Distinguish surface complaints from root causes
Cluster related themes and identify the underlying issue. For example, if "comp" and "growth" both surface, the root cause is often *opaque career progression*, not compensation per se. Name the root causes explicitly — this is the most valuable part of the analysis.

### 2.4 Identify preserving signals
Note what departing employees praised. These are things the organization should NOT change while addressing the negatives.

### 2.5 Recommend ranked actions
Produce 3–5 concrete recommendations, ranked by:
- Impact (how many departures it would have prevented)
- Speed (30 / 60 / 90 days to implement)
- Cost (low / medium / high)

Each recommendation must include: owner, timeline, cost bracket, and expected impact.

### 2.6 Draft talking points for the business leader
A 3–5 line narrative the HRBP can bring to the division head, framing the findings as "N exits, M root causes, K proposed actions."

---

## Step 3 — Produce the Artifact (PIF-Styled Word Document)

Invoke the `docx` skill to generate a Word document following this structure and styling.

### Document structure

1. **Cover header** (top of first page)
   - Title: *"Retention Report — [Division] — [Time Period]"* (size 20pt, PIF Green `005C4D`, bold, Fund Light font with Calibri fallback)
   - Subtitle: *"Exit Interview Analysis and Recommended Actions"* (size 12pt, gray `595959`)
   - Horizontal line divider in PIF Green

2. **Executive summary** (1 short paragraph)
   - Frame: N exits analyzed, K root causes identified, top M actions recommended

3. **Theme table** — the core artifact
   - Columns: `Theme` | `Driver` | `Frequency` | `Severity` | `Evidence Quote`
   - Header row: PIF Green background (`005C4D`), white text
   - Body rows: alternating white and light gray (`F2F2F2`)
   - Evidence Quote column: italic, tan callout color (`C4984F`)

4. **Root cause analysis** (2–3 short paragraphs)
   - Section heading in PIF Green
   - Body in text gray (`595959`)

5. **Preserving signals** (bullet list)
   - Section heading in PIF Green
   - What NOT to disrupt

6. **Recommended actions** (numbered)
   - Each action card in a light-bordered box (border color `9A9A9A`)
   - Action title in bold PIF Green
   - Rationale, owner, timeline, cost, expected impact — as sub-bullets in gray

7. **Talking points for the business leader** (highlighted callout box)
   - Border: PIF Green
   - Background: very light gray (`F8F8F8`)
   - Text in body gray

8. **Footer**
   - "Confidential — HR use only" in soft gray (`9A9A9A`), 8pt, right-aligned

### Styling specification (bake into the docx generation)

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

### File naming
Save the file as: `YYYYMMDD_Retention_Report_[Division].docx` — e.g., `20260712_Retention_Report_RealEstate.docx`

### Confirmation to user
After generating the file, tell the user:
> "Retention report generated: `[filename]`. It's saved to your working directory and follows PIF visual styling."

---

## Key Principles

- **Every claim must be traceable.** No assertion in the report exists without a quote from the transcripts.
- **Distinguish patterns from anecdotes.** If only 1 of 5 transcripts mentions something, flag it as a single-instance signal, not a theme.
- **Root causes over surface complaints.** The value of this skill is in cluster analysis — comp complaints and growth complaints often point to the same underlying issue.
- **Preserve what's working.** Departing employees often praise real strengths. Highlight these so the organization doesn't break them while fixing the negatives.
- **Actionable, not aspirational.** Every recommendation must have a named owner, timeline, and cost estimate.
- **Respect confidentiality.** Never quote anything from the transcripts that could re-identify a specific employee beyond the group being analyzed.

## Limitations

- Single-transcript inputs produce individual case notes, not organizational patterns. Require at least 3 transcripts for pattern analysis.
- Cannot infer beyond what's in the transcripts. If a driver isn't discussed, it can't be diagnosed.
- Not a substitute for professional HR judgment or legal review on sensitive cases (e.g., discrimination signals). Flag sensitive-topic language at the top of the report.

## Example Trigger Prompts

- "Analyze these exit interviews and produce a retention report."
- "I have exit interviews from Q1 2026 for the Investments Division — help me understand why people left."
- "Build a retention report from the attached transcripts."
