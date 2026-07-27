# Behavioral Science Skills

Two [Claude Code](https://claude.com/claude-code) skills for the survey-experiment
workflow common in behavioral science (psychology, organizational behavior,
marketing, and adjacent fields): turning a raw survey data export into a
reproducible R analysis, and turning finished results into a clean written
report.

## Skills

- **[`survey-data-analysis/`](survey-data-analysis/SKILL.md)** — generates a
  structured, reproducible R QMD (Quarto) file from a survey CSV export:
  data cleaning, composite scores, alpha reliability, t-tests/ANOVAs/
  regression/mediation/moderation, and plots. Asks about your study design
  and cleaning conventions up front rather than assuming a fixed format.
- **[`results-summary-report/`](results-summary-report/SKILL.md)** —
  generates a clean summary report of a study's results, in Markdown by
  default (with optional HTML slide-style or LaTeX Beamer output for
  presentations).

Use them together (data → analysis → report) or independently.

## Installation

Copy the skill folder(s) you want into your Claude Code skills directory:

```bash
# Global (available in every project)
cp -r survey-data-analysis ~/.claude/skills/
cp -r results-summary-report ~/.claude/skills/

# Or project-local (available only in one project)
cp -r survey-data-analysis /path/to/project/.claude/skills/
cp -r results-summary-report /path/to/project/.claude/skills/
```

Claude Code will pick up the `SKILL.md` in each folder automatically.

## Usage

Just describe what you need in plain language, e.g.:

> "Here's my Qualtrics CSV, help me write the analysis script."

> "Write up a results summary for this pilot for my advisor."

Each skill starts by asking a short set of questions about your study design
(number of conditions, DV structure, cleaning rules, etc.) before writing
anything — answer those first for the best result.

## Adapting these skills

These are templates, not a rigid framework. Everything in brackets or
described as "ask the user" is meant to be filled in with your lab's own
conventions — column-naming schemes, exclusion rules, reproducibility seeds,
and house style are all deliberately left open rather than hardcoded. Fork
and edit `SKILL.md` directly to bake in your own defaults if you'd rather
skip the STEP 0 questions each time.

## License

MIT — see [LICENSE](LICENSE).
