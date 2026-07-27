---
name: results-summary-report
description: >
  Create a clean, minimal results summary report from a completed
  behavioral-science study or pilot. Use when the user wants to write up
  findings for an advisor, lab meeting, co-author, or their own records.
  Default output is Markdown; an optional HTML slide-style variant is
  available for presentations.
---

# Results Summary Report Skill

You are generating a **minimalistic, editable results summary** from a
behavioral-science study (experiment, survey, or pilot). The output should:

- Be **plain and portable**: Markdown by default, so it can be pasted into
  an email, a lab wiki, GitHub, or a document without reformatting
- Look **clean and academic**: no decoration, no icons, minimal color
- Be **easy to edit**: the user will tweak wording and numbers before
  sharing
- Optionally convert to a **slide-style presentation** (HTML or LaTeX
  Beamer) when the user is presenting live rather than sharing a written
  summary

---

## STEP 0 — Ask before generating

Before writing, confirm:

1. **What study / pilot is this?** (study number, brief topic)
2. **What is the key finding / message?** (one sentence if possible)
3. **What figures / tables exist?** (are there saved image files to embed,
   or should placeholders be left?)
4. **Who is this for?** (advisor, lab meeting, co-author, personal record)
   — this affects tone and how much procedural detail to include, not the
   underlying structure
5. **Output format:** Markdown (default) vs. HTML slide-style vs. LaTeX
   Beamer?
6. **Is this a pilot report or a full study report?**
   - Pilot → include "What worked / what needs fixing" section
   - Full study → include implications / next steps

---

## STEP 1 — Report Structure

Use this section order for **every** report. Skip sections that don't apply
(e.g., no mediation? skip that section). Keep it to the point — one key
idea per section.

```
1. Study Overview
2. Design & Procedure
3. Participants & Data Quality
4. Results — Manipulation Check
5. Results — Main DVs
6. Results — Secondary / Exploratory
7. Results — Mediation / Moderation (if applicable)
8. Summary & Implications / Next Steps
```

For pilot reports, replace section 8 with:

```
8. What Worked / Issues Found / Changes for Next Study
```

---

## STEP 2 — Markdown format (default)

Use this template. Fill in every `[bracket]`; delete sections that don't
apply rather than leaving them empty.

```markdown
# [Study Number & Title]

**Project:** [Project Name] · **Date:** [Date]

## Overview

**Research question:** [One sentence]

- **Hypothesis:** [State H1 clearly]
- **Design:** [e.g., 2 (condition: A vs. B) between-subjects]
- **Platform:** [e.g., Prolific, MTurk, university subject pool]
- **Sample:** *N* = [X] (after exclusions)

## Design & Procedure

**[Manipulation name]**

- [Condition 1 description]
- [Condition 2 description]

[Brief procedure narrative — 2–3 sentences max]

## Participants & Data Quality

- Started: [N_raw]
- Excluded (attention check): [n]
- Excluded (duplicate / incomplete): [n]
- **Final sample:** *N* = [N_final]

| Condition | *N* |
|---|---|
| [Cond 1] | [n1] |
| [Cond 2] | [n2] |

## Manipulation Check

[Was the manipulation successful?]

> [Key stat: e.g., *t*(N) = X.XX, *p* = .XXX, *d* = X.XX — condition
> successfully raised/lowered [measure] relative to control]

![Manipulation check figure]([figure_path.png])
*Figure 1. [Caption]*

## Main Results

**[Key finding headline — one sentence]**

**[DV 1 name]**
- *F*([df1], [df2]) = [F], *p* = [p], η²p = [eta]
- [Pairwise: condition A > condition B, *p* = .XXX]

**[DV 2 name]**
- [stat]

![Main results]([figure_path.png])
*Figure 2. [Caption]*

## Mediation / Moderation

[Does [mediator] explain the effect of [IV] on [DV]?]

- a-path: [stat]
- b-path: [stat]
- Indirect effect: *B* = [X], 95% CI [LL, UL] — [significant / not]
- Direct effect: *B* = [X], *p* = [p]

## Summary & Next Steps

[One-line takeaway]

- [Key finding 1]
- [Key finding 2]
- [Limitation or open question]

**Next:**
- [Planned study / change / pre-registration]
```

---

## STEP 3 — HTML slide-style format (optional, for presentations)

Use this when the user is presenting live rather than sharing a written
summary. It renders as a clean, slide-like scrollable page; each `<section>`
is one "slide."

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Study Report — [STUDY NAME]</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: "Helvetica Neue", Arial, sans-serif;
    font-size: 16px;
    color: #1a1a1a;
    background: #fff;
    max-width: 860px;
    margin: 0 auto;
    padding: 40px 40px 80px;
  }
  section {
    border-top: 2px solid #1a1a1a;
    padding: 36px 0 28px;
    page-break-inside: avoid;
  }
  section:first-of-type { border-top: none; }
  h1 { font-size: 1.05em; font-weight: 600;
       color: #555; text-transform: uppercase;
       letter-spacing: .08em; margin-bottom: 18px; }
  h2 { font-size: 1.6em; font-weight: 700;
       line-height: 1.25; margin-bottom: 12px; }
  p  { font-size: 1em; line-height: 1.65;
       color: #2a2a2a; margin-bottom: 10px; }
  ul { padding-left: 1.4em; margin-bottom: 10px; }
  li { margin-bottom: 6px; line-height: 1.55; }
  .stat {
    font-size: 2.4em; font-weight: 700;
    color: #1a1a1a; line-height: 1;
    margin: 12px 0 4px;
  }
  .stat-label { font-size: .9em; color: #666; }
  .cols { display: flex; gap: 40px; align-items: flex-start; }
  .cols > div { flex: 1; }
  figure { margin: 18px 0; }
  figure img {
    max-width: 100%; border: 1px solid #e0e0e0;
    border-radius: 4px; display: block;
  }
  figcaption {
    font-size: .82em; color: #666;
    margin-top: 6px; text-align: center;
  }
  table {
    border-collapse: collapse; width: 100%;
    margin: 16px 0; font-size: .9em;
  }
  th {
    border-bottom: 2px solid #1a1a1a;
    text-align: left; padding: 6px 10px;
    font-weight: 600;
  }
  td { border-bottom: 1px solid #ddd; padding: 6px 10px; }
  tr:last-child td { border-bottom: none; }
  .box {
    background: #f5f5f5;
    border-left: 3px solid #1a1a1a;
    padding: 12px 16px; margin: 14px 0;
    font-size: .95em; line-height: 1.6;
  }
  @media print {
    section { page-break-before: always; }
    section:first-of-type { page-break-before: auto; }
  }
</style>
</head>
<body>

<section>
  <h1>[Project Name] · [Date]</h1>
  <h2>[Study Number & Title]</h2>
  <p style="color:#555; font-size:.95em;">Prepared for: [Audience] &nbsp;|&nbsp; [Your Name]</p>
</section>

<section>
  <h1>Overview</h1>
  <h2>[One-sentence research question]</h2>
  <ul>
    <li><strong>Hypothesis:</strong> [State H1 clearly]</li>
    <li><strong>Design:</strong> [e.g., 2-cell between-subjects]</li>
    <li><strong>Platform:</strong> [e.g., Prolific]</li>
    <li><strong>Sample:</strong> N = [X] (after exclusions)</li>
  </ul>
</section>

<section>
  <h1>Design & Procedure</h1>
  <h2>[Manipulation Name]</h2>
  <ul>
    <li>[Condition 1 description]</li>
    <li>[Condition 2 description]</li>
  </ul>
  <p>[Brief procedure narrative — 2–3 sentences max]</p>
</section>

<section>
  <h1>Participants & Data Quality</h1>
  <div class="cols">
    <div>
      <p class="stat">[N_final]</p>
      <p class="stat-label">Final sample (after exclusions)</p>
      <ul style="margin-top:14px">
        <li>Started: [N_raw]</li>
        <li>Excluded (attention): [n]</li>
        <li>Excluded (duplicate / incomplete): [n]</li>
      </ul>
    </div>
    <div>
      <table>
        <tr><th>Condition</th><th>N</th></tr>
        <tr><td>[Cond 1]</td><td>[n1]</td></tr>
        <tr><td>[Cond 2]</td><td>[n2]</td></tr>
      </table>
    </div>
  </div>
</section>

<section>
  <h1>Manipulation Check</h1>
  <h2>[Was the manipulation successful?]</h2>
  <div class="box">
    [Key stat: e.g., t(N) = X.XX, p = .XXX, d = X.XX]
  </div>
  <figure>
    <img src="[figure_path.png]" alt="Manipulation check figure">
    <figcaption>Figure 1. [Caption]</figcaption>
  </figure>
</section>

<section>
  <h1>Main Results</h1>
  <h2>[Key finding headline — one sentence]</h2>
  <div class="cols">
    <div>
      <p><strong>[DV 1 name]</strong></p>
      <ul>
        <li>F([df1], [df2]) = [F], p = [p], η²p = [eta]</li>
        <li>[Pairwise: condition A > condition B, p = .XXX]</li>
      </ul>
      <p><strong>[DV 2 name]</strong></p>
      <ul>
        <li>[stat]</li>
      </ul>
    </div>
    <div>
      <figure>
        <img src="[figure_path.png]" alt="Main results">
        <figcaption>Figure 2. [Caption]</figcaption>
      </figure>
    </div>
  </div>
</section>

<section>
  <h1>Mediation / Moderation</h1>
  <h2>[Does [mediator] explain the effect of [IV] on [DV]?]</h2>
  <ul>
    <li>a-path: [stat]</li>
    <li>b-path: [stat]</li>
    <li>Indirect effect: B = [X], 95% CI [LL, UL] — [significant / not]</li>
    <li>Direct effect: B = [X], p = [p]</li>
  </ul>
</section>

<section>
  <h1>Summary & Next Steps</h1>
  <h2>[One-line takeaway]</h2>
  <ul>
    <li>[Key finding 1]</li>
    <li>[Key finding 2]</li>
    <li>[Limitation or open question]</li>
  </ul>
  <p><strong>Next:</strong></p>
  <ul>
    <li>[Planned study / change / pre-registration]</li>
  </ul>
</section>

</body>
</html>
```

---

## STEP 4 — LaTeX / Beamer format (optional, when requested)

Use this when the user asks for LaTeX / Overleaf output. It compiles
directly in Overleaf and exports to PDF; use Pandoc to convert to PPTX if
needed.

```latex
\documentclass[aspectratio=169, 11pt]{beamer}

\usetheme{default}
\usecolortheme{default}
\setbeamertemplate{navigation symbols}{}
\setbeamertemplate{footline}[frame number]
\setbeamertemplate{frametitle}{\vspace{.4em}\insertframetitle\par\vspace{.2em}\rule{\textwidth}{0.4pt}}
\setbeamerfont{frametitle}{size=\large, series=\bfseries}
\setbeamercolor{frametitle}{fg=black}
\setbeamercolor{normal text}{fg=black}
\setbeamercolor{structure}{fg=black}

\usepackage{fontenc}
\usepackage{inputenc}
\usepackage[T1]{fontenc}
\usepackage{helvet}
\renewcommand{\familydefault}{\sfdefault}

\usepackage{booktabs}
\usepackage{graphicx}
\usepackage{amsmath}
\usepackage{xcolor}
\usepackage{tikz}

\title{[Study Number \& Title]}
\subtitle{[Project Name]}
\author{[Your Name]}
\date{\today}

\begin{document}

\begin{frame}
  \titlepage
\end{frame}

\begin{frame}{Overview}
  \textbf{Research Question:} [One sentence] \\[.8em]
  \begin{itemize}
    \item \textbf{H1:} [Hypothesis]
    \item \textbf{Design:} [e.g., 2-cell between-subjects]
    \item \textbf{N:} [X] (after exclusions)
  \end{itemize}
\end{frame}

\begin{frame}{Design \& Procedure}
  [Condition descriptions as itemize]
\end{frame}

\begin{frame}{Main Results}
  \begin{columns}
    \column{.5\textwidth}
      \begin{itemize}
        \item [DV 1]: $F([df1],[df2]) = [F]$, $p = [p]$, $\eta^2_p = [eta]$
        \item [DV 2]: [stat]
      \end{itemize}
    \column{.5\textwidth}
      \includegraphics[width=\linewidth]{figure.png}
  \end{columns}
\end{frame}

\begin{frame}{Summary \& Next Steps}
  \begin{itemize}
    \item [Finding 1]
    \item [Finding 2]
  \end{itemize}
  \vspace{1em}
  \textbf{Next:} [Planned action]
\end{frame}

\end{document}
```

---

## STEP 5 — Filling in the content

When filling the template, pull content from:
1. The analysis output (F-stats, t-stats, p-values, effect sizes)
2. The user's description of the study / findings
3. Any figures provided

**Formatting stats in the report (APA style):**
- Italicize test statistics: *F*, *t*, *p*, *d*, *r* (Markdown: `*F*`;
  HTML: `<em>F</em>`)
- Effect sizes: always include (*d*, η²p, or indirect effect CI)
- *p*-values: report to 3 decimal places; `p < .001` if below
- Round means to 2 decimal places
- Never fabricate or estimate a number — leave `[INSERT STAT]` explicitly
  if a value hasn't been confirmed from the analysis output

---

## STEP 6 — Saving and converting

- **Save as:** `[study_name]_report_[date].md` (or `.html` / `.tex`)
- **Markdown → shareable doc:** paste directly into email, Notion, or a
  GitHub issue/PR — no conversion needed
- **HTML → PPTX:** open in Chrome → Print → Save as PDF → import into
  PowerPoint via "Insert → Photo Album," or import the PDF pages directly
- **LaTeX → PPTX:** compile in Overleaf to PDF, then convert via Adobe
  Acrobat or an online PDF-to-PPTX tool

---

## Checklist before delivering the report

- [ ] Every section has one clear headline
- [ ] Stats are APA-formatted (italicized statistic symbols, 3 decimal places for *p*)
- [ ] Figures are either embedded or clearly placeholder-labeled
- [ ] No section is overloaded — keep bullets short
- [ ] Pilot reports include a "what needs fixing" section
- [ ] No number is invented — anything unconfirmed is left as an explicit placeholder
- [ ] File is saved with the correct extension (`.md` / `.html` / `.tex`) in the project folder
