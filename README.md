# Behavioral Science Skills

Two Agent Skills for the survey-experiment workflow common in behavioral
science (psychology, organizational behavior, marketing, and adjacent
fields): turning a raw survey data export into a reproducible R analysis,
and turning finished results into a clean written report.

Written to the shared `SKILL.md` spec (a folder with a `SKILL.md` file
carrying `name`/`description` frontmatter), so the same files work
unmodified in both [Claude Code](https://claude.com/claude-code) and
[Codex CLI](https://developers.openai.com/codex/cli) — only the install
path differs between the two.

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

If you use these skills or find them helpful and have a GitHub account,
please star this repo :)

## Installation

Copy the skill folder(s) you want into the skills directory for your tool.

**Claude Code:**
```bash
# Global (available in every project)
cp -r survey-data-analysis ~/.claude/skills/
cp -r results-summary-report ~/.claude/skills/

# Or project-local (available only in one project)
cp -r survey-data-analysis /path/to/project/.claude/skills/
cp -r results-summary-report /path/to/project/.claude/skills/
```

**Codex CLI:**
```bash
# Global (available in every project)
cp -r survey-data-analysis ~/.agents/skills/
cp -r results-summary-report ~/.agents/skills/

# Or repo-local (available only in one project)
cp -r survey-data-analysis /path/to/project/.agents/skills/
cp -r results-summary-report /path/to/project/.agents/skills/
```

Both tools scan their skills directory at startup and pick up each folder's
`SKILL.md` automatically — no other setup needed.

## Usage

Just describe what you need in plain language, e.g.:

> "Here's my Qualtrics CSV, help me write the analysis script."

> "Write up a results summary for this pilot for my advisor."

Each skill starts by asking a short set of questions about your study design
(number of conditions, DV structure, cleaning rules, etc.) before writing
anything — answer those first for the best result.

## License

MIT — see [LICENSE](LICENSE).
