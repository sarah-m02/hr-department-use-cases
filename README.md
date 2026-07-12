# HR Department Use Cases

AI-augmented HR skills for use with Claude Code, organized around a simplified 3-pillar HR operating structure (Ulrich model).

## Framework

| Pillar | What it is | Skill |
|---|---|---|
| **Advise** | HR partners embedded in the business | `hr-employee-retention` |
| **Design** | Specialist teams designing HR programs | `hr-job-description` |
| **Deliver** | Operational execution | `hr-onboarding` |
| **Cross-pillar** | General-purpose HR conversation analysis | `hr-transcript-analysis` |

## Skills

Each skill in `skills/` is a self-contained Claude Code Agent Skill (`SKILL.md`).
Each one is designed to:

1. Prompt the user for the specific inputs it needs
2. Run its analysis
3. Produce a downloadable artifact (Word, Excel, or PDF) styled with the PIF visual identity — dark teal green (`#005C4D`), tan (`#C4984F`), gray (`#595959`), and Fund typography

## Installation

Copy the `skills/` folder contents into your Claude Code skills directory:

**Windows:** `C:\Users\<you>\.claude\skills\`
**macOS / Linux:** `~/.claude/skills/`

Restart Claude Code — the skills will be discovered on startup.

## Attribution

Four of the pillar skills are adapted from the open-source [tuanductran/hr-skills](https://github.com/tuanductran/hr-skills) library (MIT-licensed), scoped and extended for PIF use — added interactive input collection and PIF-styled artifact output.

The cross-pillar `hr-transcript-analysis` skill is original — built to route interview types (applicant / exit / grievance / focus group) through one common analytical backbone.
