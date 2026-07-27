---
name: survey-data-analysis
description: >
  Generate a complete behavioral-science survey analysis in R QMD (Quarto)
  format. Use when the user wants to analyze a survey data export (Qualtrics,
  Prolific-linked, or similar), write a data-cleaning script, build
  composites, run t-tests / ANOVAs / regressions / mediation / moderation,
  or produce a structured, reproducible QMD from experimental or
  correlational survey data.
---

# Survey Data → QMD Analysis Skill

You are helping a behavioral scientist (psychology, organizational behavior,
marketing, or adjacent field) turn a raw survey data export into a fully
structured, reproducible R QMD analysis file. Ask enough up front to match
their study design and their lab's conventions rather than assuming your own
defaults — every hardcoded value below (column names, seeds, filters) is a
placeholder to be replaced by what the user tells you in STEP 0.

---

## STEP 0 — Gather context before writing a single line of code

Ask (or infer from the CSV / conversation) the answers to all of the
following questions. Do not skip any. If the user provides the CSV, skim the
column names first.

### A. Study Design
1. **How many between-subjects factors?** (0 = correlational; 1 = simple;
   2 = factorial, e.g., 2×2)
2. **How many cells / conditions?** What are the condition labels?
   - 2-cell → t-test or one-way ANOVA
   - 3-cell → one-way ANOVA + pairwise emmeans
   - 2×2 factorial → two-way ANOVA + interaction
   - No conditions → regression / correlational
3. **Is there a within-subjects factor?** (repeated measures)
4. **What column holds the condition?** (e.g., `condition`, `group`, `arm`)
5. **What is the participant ID column?** (e.g., `PROLIFIC_PID`,
   `participantId`, `ResponseId`, `MTurkID`)

### B. DVs and Coding
6. **What are the main DVs?** For each DV:
   - Column name(s) (e.g., items `dv_1`, `dv_2` → composite `dv_composite`)
   - Scale range (1–5, 1–7, etc.)
   - Number of items → composite = `rowMeans` / sum / divided
   - Any **reverse-coded** items? (formula: `scale_max + 1 - item`,
     e.g., for a 7-point scale: `8 - item`)
   - Expected alpha (if known)
7. **Is any DV binary / count?** (e.g., yes/no outcome, an integer score)
   If binary → chi-square + logistic regression
   If count → also check Poisson / negative binomial
8. **What trait / covariate measures exist?** Note any prefix patterns
   used for multi-item scales (e.g., `hexaco_`, `narc_`, `swb_`) so items can
   be selected with `starts_with()`.

### C. Data Cleaning Specifics
9. **How is the sample deduplicated / restricted to valid respondents?**
   Ask what the study uses: participant ID, IP address, completion status,
   consent column, etc. Do not assume a specific platform's field names —
   confirm the exact column names and values from the user or the CSV
   header.
10. **Attention check(s)?** Ask what the attention-check item(s) are and
    what the passing value/string is. Different studies (and different
    platforms) use different formats — don't assume a fixed pattern.
11. **Comprehension checks?** (condition-specific filtering, e.g.
    `comprehension == 1` required only for one condition)
12. **Any study-specific extra cleaning?** (e.g., drop first N rows of a
    condition, merge in data from a second file, etc.)
13. **Extra header/metadata rows?** Some export formats (Qualtrics in
    particular) include 1–2 extra metadata rows below the column names. If
    so, add `slice(-c(1,2))` (or however many rows) right after
    `read.csv()`.
14. **Does the study have bonus pay or behavioral scoring?**
    (e.g., a performance score converted into a binary/continuous outcome)

### D. Analysis Plan
15. **Primary analysis:** t-test, one-way ANOVA, two-way ANOVA (specify
    which), logistic regression, or multiple regression?
16. **Mediation?** If yes: DV, IV (contrast-coded), mediator(s),
    PROCESS model number (4 = simple, 6 = serial two-mediator)
17. **Moderation / interaction?** If yes: IV × moderator, which DV?
    Mean-center both before `lm()`?
18. **Any exploratory regressions?** (trait measures as predictors,
    including interactions with condition)
19. **Reproducibility seed:** ask what seed the user's lab standardizes on
    for bootstrapped analyses (e.g., PROCESS). If none exists, pick one and
    note it explicitly in the file so it's easy to find and reuse.

---

## STEP 1 — QMD Structure

Always produce clearly separated QMD sections, e.g.:

```
## Data Cleaning
## Measures
## Main Analysis
## Other Analyses   (mediation / moderation / exploratory)
```

Use `format: html` and `editor: visual` in the YAML header unless the user
asks for something else (e.g., `format: pdf` for a manuscript appendix).

---

## STEP 2 — Boilerplate to always include

### Package loader (always first chunk)
```r
ipak <- function(pkg){
  new.pkg <- pkg[!(pkg %in% installed.packages()[, "Package"])]
  if (length(new.pkg))
    install.packages(new.pkg, dependencies = TRUE)
  sapply(pkg, require, character.only = TRUE)
}
packages <- c("ggplot2", "tidyverse", "dplyr", "psych", "afex",
               "effectsize", "emmeans", "apaTables")
# Add as needed: "stargazer", "lm.beta", "interactions", "mediation", "MASS"
ipak(packages)
```

### Standard data cleaning pipeline
Adapt every filter to the answers gathered in STEP 0 — the block below is a
template, not a fixed sequence:
```r
raw_data <- read.csv("FILENAME.csv")

# If the export has extra header rows, add:
# raw_data <- raw_data %>% slice(-c(1, 2))

clean_data <- raw_data %>%
  filter(!is.na(PARTICIPANT_ID_COLUMN)) %>%
  distinct(PARTICIPANT_ID_COLUMN, .keep_all = TRUE) %>%  # deduplicate by ID
  distinct(IPAddress, .keep_all = TRUE) %>%              # if IP is available
  filter(Finished == 1) %>%                              # if applicable
  filter(consent == 1) %>%                                # if applicable
  filter(ATTENTION_CHECK_COLUMN == PASSING_VALUE)         # confirm from user

table(clean_data$condition)
```

---

## STEP 3 — Measures section

For **every scale**:
1. Convert items to numeric
2. Check alpha with `psych::alpha(..., check.keys=TRUE)`
3. Create composite
4. `psych::describe()` + `hist()`

```r
# Convert to numeric
clean_data <- clean_data %>%
  mutate(across(c("item_1", "item_2", "item_3"), as.numeric))

# Alpha
psych::alpha(clean_data %>% select("item_1", "item_2", "item_3"))

# Composite (simple mean)
clean_data <- clean_data %>%
  mutate(composite_dv = (item_1 + item_2 + item_3) / 3)

# Composite with rowSums (for prefix-based selection)
clean_data <- clean_data %>%
  mutate(all_scale = rowSums(across(starts_with("prefix_")), na.rm = TRUE) / n_items)

psych::describe(clean_data$composite_dv)
hist(clean_data$composite_dv)
```

**Reverse coding** (for a 7-point scale: `8 - item`):
```r
clean_data <- clean_data %>%
  mutate(
    item_2 = 8 - item_2,   # reverse
    item_4 = 8 - item_4    # reverse
  )
```

**Mean-centering** (always do before interactions/moderation):
```r
clean_data <- clean_data %>%
  mutate(
    mc_dv        = as.numeric(scale(dv, center = TRUE, scale = FALSE)),
    mc_moderator = as.numeric(scale(moderator, center = TRUE, scale = FALSE))
  )
```

**Correlation table** (always include, put at end of Measures section):
```r
apa.cor.table(clean_data %>%
  select("var1", "var2", "var3", ...),
  filename = "cor_table.doc",
  table.number = 1)
```

**Group summary** (always include):
```r
clean_data %>%
  group_by(condition) %>%
  summarise(across(c("dv1", "dv2"),
                   list(mean = mean, sd = sd), na.rm = TRUE))
```

---

## STEP 4 — Main Analysis

Choose the right test based on the study design:

### 2-cell between-subjects (t-test)
```r
t.test(dv ~ condition, var.equal = TRUE, data = clean_data)
cohens_d(dv ~ condition, data = clean_data, pooled_sd = TRUE)
```

For binary DV:
```r
tab <- table(clean_data$condition, clean_data$binary_dv)
chisq.test(tab, correct = FALSE)
prop.test(x = c(sum_a, sum_b), n = c(n_a, n_b), correct = FALSE)
```

### One-way ANOVA (3+ cells)
```r
anova_result <- afex::aov_car(
  dv ~ condition + Error(PARTICIPANT_ID_COLUMN),
  data = clean_data,
  anova_table = list(correction = "none", es = "pes"),
  type = 3, digits = 3
)
summary(anova_result)
effectsize::eta_squared(anova_result, partial = TRUE)
emmeans(anova_result, ~ condition, contr = "pairwise", adjust = "tukey")
```

> **Pilot studies:** default to `adjust = "none"` for pairwise comparisons
> unless the user says otherwise.
> **Confirmatory studies:** do not assume Tukey. Check the pre-registration
> and ask explicitly which adjustment (if any) was specified before writing
> the emmeans call.

### 2×2 Factorial ANOVA
```r
anova_result <- afex::aov_car(
  dv ~ factor_A + factor_B + factor_A:factor_B + Error(PARTICIPANT_ID_COLUMN),
  data = clean_data,
  anova_table = list(correction = "none", es = "pes"),
  type = 3, digits = 3
)
summary(anova_result)
effectsize::eta_squared(anova_result, partial = TRUE)
emmeans(anova_result, pairwise ~ factor_B | factor_A, adjust = "none")
emmeans(anova_result, pairwise ~ factor_A | factor_B, adjust = "none")
```

### Standard emmeans bar plot (use for ALL ANOVAs)
```r
emm_results <- emmeans(anova_result, ~ condition) %>% as.data.frame()

ggplot(emm_results, aes(x = condition, y = emmean)) +
  geom_col(fill = "steelblue") +
  geom_errorbar(aes(ymin = emmean - SE, ymax = emmean + SE), width = 0.2) +
  theme_minimal() +
  labs(title = "DV by Condition", x = "Condition", y = "DV Label") +
  theme(text = element_text(size = 14))
```

For 2×2 factorial (grouped bars):
```r
emm_results <- emmeans(anova_result, ~ factor_A * factor_B) %>% as.data.frame()

ggplot(emm_results, aes(x = factor_A, y = emmean, fill = factor_B)) +
  geom_col(position = position_dodge(0.7), width = 0.6) +
  geom_errorbar(
    aes(ymin = emmean - SE, ymax = emmean + SE),
    position = position_dodge(0.7), width = 0.2
  ) +
  theme_minimal() +
  labs(x = "Factor A", y = "DV", fill = "Factor B")
```

### Logistic regression (binary DV)
```r
# Create contrast-coded IV first
clean_data <- clean_data %>%
  mutate(condition_con = case_when(
    condition == "control" ~ -1,
    condition == "treatment" ~ 1
  ))

summary(glm(binary_dv ~ condition_con + covariate, family = "binomial",
            data = clean_data))
```

---

## STEP 5 — Other Analyses

### Moderation (always mean-center first)
```r
library(interactions)

mod_model <- lm(mc_dv ~ condition_con * mc_moderator, data = clean_data)
summary(mod_model)

interact_plot(mod_model,
              pred = mc_moderator,
              modx = condition_con,
              interval = TRUE,
              x.label = "Mean-Centered Moderator",
              y.label = "Mean-Centered DV")

sim_slopes(mod_model,
           pred = mc_moderator,
           modx = condition_con)
```

### Mediation (PROCESS macro)
Requires Andrew Hayes' `process.R` file (downloadable from
processmacro.org) in the same directory. Use the reproducibility seed
confirmed in STEP 0.
```r
source("process.R")

# Contrast code IV (e.g., -1 = control, 1 = treatment)
clean_data <- clean_data %>%
  mutate(condition_con = case_when(
    condition == "control"   ~ -1,
    condition == "treatment" ~  1
  ))

# Simple mediation (Model 4)
process(data = clean_data,
        y = "dv", x = "condition_con", m = "mediator",
        model = 4, total = 1, normal = 1, boot = 10000, seed = SEED)

# Serial two-mediator (Model 6)
process(data = clean_data,
        y = "dv", x = "condition_con", m = c("med1", "med2"),
        model = 6, total = 1, normal = 1, boot = 10000, seed = SEED)
```

**Note on contrast coding for 3-cell designs:**
```r
# Collapse two conditions into one contrast (e.g., focal vs. rest)
clean_data <- clean_data %>%
  mutate(condition_con = case_when(
    condition == "control" ~ -1,
    condition == "focal"   ~  2,
    condition == "other"   ~ -1
  ))
```

### Exploratory regressions with trait measures
```r
# Always mean-center trait measures before interactions
summary(lm(dv ~ condition_con * mc_trait, data = clean_data))
summary(lm(dv ~ condition + trait1 + trait2 + trait3, data = clean_data))
```

### Effect coding (for 3+ cell ANOVA in regression)
```r
clean_data$effect_condition <- factor(clean_data$condition,
                                       levels = c("cell_A", "cell_B", "cell_C"))
contrasts(clean_data$effect_condition) <- contr.sum(3)
summary(lm(dv ~ effect_condition * mc_trait, data = clean_data))
```

---

## STEP 6 — Output conventions

- Name the QMD file to match the user's project naming convention (ask if
  unclear); a common pattern is `studyname_studynumber.qmd`
  (e.g., `pilot01.qmd`, `study2b.qmd`)
- The data CSV should be in the same directory — use a relative path
- `process.R` must also be in the same directory if mediation is used
- Always end the file with a blank code chunk so Quarto doesn't complain
- Report alphas as inline comments above the alpha chunk
  (e.g., `# alpha = 0.88`)

---

## Multi-survey / linked designs

Some studies collect data across two or more linked surveys — e.g. a sender
and a receiver, an actor and a rater, or a dyadic pairing where each side
fills out a separate instrument. The clean workflow for these:

### Cleaned CSV generation (one-time R script)

Run a separate `generate_cleaned.R` script once, producing a single cleaned
CSV that the analysis QMD reads from (rather than re-cleaning raw exports
inside the analysis file itself).

```r
clean_side_a <- raw_side_a %>%
  filter(!is.na(PARTICIPANT_ID) & PARTICIPANT_ID != "") %>%
  distinct(PARTICIPANT_ID, .keep_all = TRUE) %>%
  distinct(IPAddress, .keep_all = TRUE) %>%
  filter(ATTENTION_CHECK_COLUMN == PASSING_VALUE) %>%
  filter(Finished == 1)
clean_side_a$link_id <- seq_len(nrow(clean_side_a))  # assigns a shared linking ID

clean_side_b <- raw_side_b %>%
  filter(!is.na(PARTICIPANT_ID) & PARTICIPANT_ID != "") %>%
  distinct(PARTICIPANT_ID, .keep_all = TRUE) %>%
  distinct(IPAddress, .keep_all = TRUE) %>%
  filter(ATTENTION_CHECK_COLUMN == PASSING_VALUE) %>%
  filter(Finished == 1)
clean_side_b$link_id <- as.integer(clean_side_b$link_id)  # already present in CSV, or joined in
```

**Combine:** use `intersect(common_cols, colnames(df))` to handle column
differences between the two surveys safely.
```r
all_valid <- bind_rows(
  clean_side_a[, intersect(common_cols, colnames(clean_side_a))],
  clean_side_b[, intersect(common_cols, colnames(clean_side_b))]
)
```

**Cleaned CSV columns:** raw items, reversed items, composites, any
human-coded variables merged in from a separate coding file, and an
`include` flag marking the final analysis sample.

### Building multiple analysis datasets in the QMD

If the study has both a pre-registered sample definition and a broader
exploratory sample, build both explicitly and label which is which:
```r
all_valid <- read.csv("study_cleaned.csv") %>%
  mutate(condition_effect = ifelse(condition == "side_a", -1, 1))

# Dataset 1: all valid participants, no additional exclusions
data_full <- all_valid

# Dataset 2: pre-registered sample (e.g., strict 1:1 linking)
data_prereg <- data_full %>%
  group_by(link_id) %>%
  filter(sum(condition == "side_a") == 1 & sum(condition == "side_b") >= 1) %>%
  group_modify(~ bind_rows(
    .x %>% filter(condition == "side_a") %>% slice_head(n = 1),
    .x %>% filter(condition == "side_b") %>% slice_head(n = 1)
  )) %>%
  ungroup()
```

### De-identification for OSF / submission release

Every cleaned CSV that will be shared publicly (OSF, supplementary
materials) should have identifying columns removed, even though they're
fine to use in the working/raw files mid-pipeline: participant ID,
IP address, geolocation, name/email, platform response ID, timing
metadata, and demographics (age, gender, ethnicity, education, employment,
income). Compute and report demographics once inside a cleaning QMD (for
the manuscript's sample-description paragraph) but don't persist them in
the shared CSV.

Keep: sequential self-generated IDs and any linking/stimulus IDs — these
identify pairings or stimuli, not participants, so they're fine to retain.

If any measure has an open-ended free-text column (e.g., a
participant-written response used as a stimulus for other participants), a
column-name scan for PII isn't enough — do a manual read-through of that
column before release, since a participant can incidentally type an
identifying detail into an open field even when every structured column is
clean.

---

## Pre-submission QMD cleanup (do this pass right before sharing/submission)

Apply to **every** `*.qmd` in the project — analysis notebooks and cleaning
notebooks alike, since the same patterns creep into both:

- **Strip manual header numbering** if `number-sections` isn't set in the
  YAML — hand-typed numbers like `## 1. Data`, `## 4. Main analysis` should
  become `## Data`, `## Main analysis`.
- **Remove rationale-style parentheticals from headers**, e.g. `## Main
  analysis (preregistered: strict 1:1 matching)` → `## Main analysis`.
  That's the header explaining its own reasoning, which reads as AI
  narration. **Keep** parentheticals that are pure citations needed for
  reproducibility (pre-registration IDs, filenames, table numbers) — those
  aren't interpretation, they're documentation a reviewer needs.
- **Remove hypothesis-statement headers/prose**, e.g. `### H2: Group B
  reports greater X than Group A.` as a header, or the same sentence as
  bare prose above a chunk. Replace with a plain header naming the DV
  (`### Perceived ambiguity`) or delete the line if a clear header already
  exists — the reader can see the test direction from the code below.
- **Remove `cat("label:", value, "\n")` print-wrapper chunks.** Replace
  with the bare expression so Quarto auto-prints the number without manual
  string-formatting scaffolding.
- **Remove top-of-file intro paragraphs** that just restate provenance
  (e.g., "Reproduces the preregistered analysis from the cleaned data
  produced by X_cleaning.qmd") — this repeats across every file in a
  project and reads as AI narration once you notice the pattern. Folder
  structure and filenames already make provenance clear.
- **Remove methodology-description prose under a header** that just
  restates what the code chunk directly below already shows. Applies
  equally to short section-label sentences like "Build the analysis
  datasets:" sitting alone above a chunk.
- **Remove standalone "why is this empty/missing" comments** sitting alone
  in an otherwise-empty chunk. Delete the whole chunk (and its header, if
  the header has no other content under it) rather than leaving a
  comment-only placeholder.
- **Keep** comments that are directly attached to a real, executing line
  and needed to understand it — e.g. a note on which sample a dataset
  represents, or why an item is reverse-worded. These aren't narration,
  they're the only place a non-obvious variable name or scale direction is
  explained. The test: does the comment explain something the code doesn't
  already show (rationale/absence/narration → cut), or is it the only
  label for an otherwise-opaque line (necessary → keep)?
- Re-render every file you touch and confirm it still exits cleanly before
  moving on; these are text/formatting-only changes and should never alter
  any computed number.

---

## Common packages by use-case

| Need | Package |
|------|---------|
| ANOVA | `afex` |
| Effect sizes | `effectsize` |
| Pairwise comparisons | `emmeans` |
| Descriptives + alpha | `psych` |
| Correlation table (APA) | `apaTables` |
| Standardized betas | `lm.beta` |
| Interaction plots | `interactions` |
| Negative binomial | `MASS` |
| Mediation (causal) | `mediation` |
| Mediation (PROCESS) | `source("process.R")` |
| Tables for paper | `stargazer` |

---

## Checklist before delivering the QMD

- [ ] `ipak()` includes all packages used in the file
- [ ] Attention-check / cleaning filters match what the user described in STEP 0
- [ ] All scale items converted to numeric before `psych::alpha()`
- [ ] Reverse-coded items handled before composite calculation
- [ ] `table(clean_data$condition)` printed after cleaning
- [ ] `apa.cor.table()` included
- [ ] Emmeans bar plots for every ANOVA DV
- [ ] Condition contrast-coded before `process()` / `lm()` interactions
- [ ] Reproducibility seed confirmed with the user and used consistently in `process()` calls
- [ ] File saved as `.qmd` in the same folder as the data CSV
- [ ] Pairwise adjustment: if pilot, default `adjust = "none"`; if confirmatory, check the pre-registration and ask explicitly before choosing an adjustment method
