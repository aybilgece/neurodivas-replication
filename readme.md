# Reproducing NEURODIVAS: AI vs. Human Advice Sources

**Rebuilding a 2×2 factorial vignette study from its published summary statistics — and re-deriving every reported result.**

This repository takes the *output* of a psychology study (the SPSS tables: cell
means, standard deviations, Ns, and the correlation matrix) and reconstructs a
participant-level dataset that reproduces the study's manipulation-check t-test,
correlation structure, and two-way MANOVA. The point isn't to re-run someone's
`.sav` file with one click — it's to show that the published statistical
conclusions can be regenerated transparently, in open tooling, from numbers
alone.

> **The interesting constraint:** the raw data file was never available — only
> the SPSS output PDF. So the dataset here is *reconstructed from published
> summary statistics*, not the original responses. See **Data & ethics** below.

---

## The original study

A between-subjects 2×2 vignette experiment (PSY 204, TED University, 2026)
crossing **Emotional Context** (high = relationship advice / low = language-learning
advice) with **Source Type** (human friend / generative-AI tool). Four perceived
social attributes were measured (N = 59):

- **SA** — Social Attraction (sum of 3 items)
- **ES** — Emotional Support (sum of 4 items)
- **PC** — Perceived Competence (mean of 3 items)
- **PW** — Perceived Warmth (mean of 3 items)

The headline finding: a significant **main effect of emotional context** (advice
sources were rated as *more competent* in the low-emotion context), with **no**
significant effect of source type and **no** interaction.

## What this repository reproduces

Running `R/01_analysis.R` on the committed dataset returns:

| Result | Reconstruction | Published |
|---|---|---|
| Manipulation-check *t*-test (context) | *t*(57) = 3.80, *p* < .001 | *t*(57) = 3.93, *p* = .001 |
| MANOVA — Emotional Context (Pillai) | *F*(4,52) = 3.59, *p* = .012 | *F*(4,52) = 3.59, *p* = .012 |
| MANOVA — Source Type (Pillai) | *F*(4,52) = 1.75, *p* = .154 (n.s.) | *F*(4,52) = 1.17, *p* = .334 (n.s.) |
| MANOVA — Context × Source (Pillai) | *F*(4,52) = 0.89, *p* = .477 (n.s.) | *F*(4,52) = 0.90, *p* = .467 (n.s.) |
| Competence: Low vs. High context | M = 3.63 vs. 3.16 | M = 3.60 vs. 3.22 |

Correlations among the four DVs land within ≈.07 of the published Table 1
(e.g. PW–ES .77 vs .81; SA–ES .68 vs .74). The descriptive cell means and SDs
match the published values *by construction*.

## How the reconstruction works

1. Draw correlated multivariate-normal data per cell using the published 4×4
   DV correlation matrix.
2. Within each cell, z-standardise and rescale each variable to the **exact**
   published cell mean and SD — so the descriptives are reproduced by design.
3. Round to the authentic Likert response grid (item sums / item means).
4. The two-way MANOVA uses **effect (sum) contrasts with Type III sums of
   squares**, which is what reproduces SPSS's main-effect tests when an
   interaction term is present.

The Python script in `validation/` is the reference that generated and checked
the committed dataset; the R pipeline is the primary, runnable analysis.

## Repository structure

```
neurodivas-replication/
├── data/
│   ├── neurodivas_synthetic.csv   # validated canonical dataset (N = 59)
│   └── DATA_DICTIONARY.md
├── R/
│   ├── 00_simulate_data.R         # reconstruction method (documented)
│   └── 01_analysis.R              # reproduces t-test, correlations, MANOVA, figures
├── validation/
│   └── generate_and_validate.py   # Python reference: builds + checks the dataset
├── outputs/                       # tables + figures written here on run
├── LICENSE
└── README.md
```

## Reproduce it

```bash
git clone <your-repo-url>
cd neurodivas-replication
Rscript R/01_analysis.R          # writes tables + figures to outputs/
```

First run installs `car`, `rstatix`, `ggplot2`, `dplyr`, `tidyr` if missing.
To re-validate the dataset itself: `python3 validation/generate_and_validate.py`
(needs `numpy`, `pandas`, `scipy`, `statsmodels`).

## Data & ethics

The committed dataset is **fully synthetic**. It contains no real participant
responses — every row is generated from the study's published aggregate
statistics. The original consent covered classroom/educational use of the data,
so the *raw* responses are deliberately **not** redistributed here; reconstructing
from public summary numbers keeps this open repository ethically clean while
still demonstrating the full analytic pipeline. Synthetic demographic columns
(age, sex, SES) are included only to mirror the dataset's shape and are not used
in any analysis.

## Honest limitations

- This reproduces the *reported statistics and conclusions*, not the original
  individual responses — synthetic data cannot recover information the summary
  statistics never contained (e.g. the true joint distribution within cells).
- The bootstrap CIs reported in the paper are not reproduced point-for-point;
  the focus here is the descriptive, correlational, and MANOVA results.
- A different random seed yields a dataset with the same descriptives and the
  same inferential conclusions, but slightly different exact test statistics.

## Citation

Aydemir, A., Tahmasbi, P., Hasan, D., Bilsin, D., Çoban, G., Şimşek, S. Z., &
Arslan, Z. A. (2026). *The Psychology of AI Companionship* (PSY 204 research
report). TED University, Department of Psychology.

Reconstruction & reproducible analysis by Aybilge.
