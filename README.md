# Replication Package — Transplanting Credit-Domain Automated-Decision Controls into Insurance

Replication code for the empirical demonstration in the manuscript
*"Specifying the Governance Gap in Insurance AI: Transplanting Credit-Domain
Decision Controls Across Underwriting, Pricing, Claims, and Fraud Detection"*
(submitted to *Risk Management and Insurance Review*, Perspectives;
author(s) anonymized for double-blind peer review).

> **Anonymity notice.** This repository is intentionally author-blind. It names
> no author, institution, or target journal, and prints no local file paths, so
> it can accompany a double-blind submission. Please do not add identifying
> information until after the review process.

## What this package demonstrates

The study transfers decision-level governance controls from the (mature) automated
**credit-evaluation** domain to the (under-specified) **insurance** domain. The
notebooks provide an artifact-level demonstration across three legs that mirror
that transfer:

| Notebook | Leg | Controls demonstrated |
|---|---|---|
| `01_credit_donor_controls` | Credit (donor domain) | Proxy detection (T-1); decision-level explanation / recourse (T-2) |
| `02_pricing_fairness_health` | Insurance pricing / underwriting | Proxy-leakage under input restriction; **effect-based (ex post) fairness testing** (T-1, T-8) |
| `03_fds_fairness_auto` | Insurance fraud detection (FDS) | False-positive disparity; false-positive / restoration metrics vs. accuracy-only tuning (T-4, T-7, T-9) |
| `04_llm_explanations` | Explanation layer (LLM) | Non-circular evaluation of adverse-decision explanations (T-2, T-3): the model's true drivers are hidden from the LLM; **faithfulness** = overlap of the LLM's inferred reasons with the model's local drivers, **recourse validity** = whether the LLM's proposed (plausible, non-protected, in-range) change actually flips the model decision, plus a protected-attribute-leakage check, over `N_CASES=15` moderately-declined cases per model tier |
| `05_bootstrap_cis` | All legs | 95% confidence intervals for every headline metric (Wilson for proportions; percentile bootstrap, 2,000 resamples, for ratios/AUCs/means) and regeneration of the four bar-chart figures **with error bars**. Runs fully offline (no API): re-fits the credit/health/fraud models and reads the committed LLM per-case results |

## Data

Public datasets, fetched automatically by each notebook from public mirrors into
`data/` if not already present (raw files are git-ignored):

- **German Credit** (Statlog) — donor-domain credit decisions.
- **Medical Cost Personal** — health-insurance charges (pricing fairness).
- **Auto Insurance Claims** — automobile claims with a fraud label (FDS).

Dataset licenses vary; see notes in `data/` and verify terms before
redistribution.

## Reproducing the results

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute notebooks/01_credit_donor_controls.ipynb --inplace
jupyter nbconvert --to notebook --execute notebooks/02_pricing_fairness_health.ipynb --inplace
jupyter nbconvert --to notebook --execute notebooks/03_fds_fairness_auto.ipynb --inplace
# 04 requires an OpenAI key and is intended to run locally:
jupyter nbconvert --to notebook --execute notebooks/04_llm_explanations.ipynb --inplace
# 05 computes confidence intervals + error-bar figures; runs OFFLINE (no API),
# reading the committed results/tables/llm_explanation_per_case.csv:
jupyter nbconvert --to notebook --execute notebooks/05_bootstrap_cis.ipynb --inplace
```

### Confidence intervals and error bars

`05_bootstrap_cis` writes `results/tables/bootstrap_cis.csv` with the point
estimate and 95% CI for every reported metric, and overwrites the four
bar-chart figures (`fig_credit_selection_rate`, `fig_health_proxy_auc`,
`fig_fds_fpr_by_sex`, `fig_llm_faithfulness`) with error bars. Proportions use
Wilson intervals; ratios, AUCs, and means use a percentile bootstrap (2,000
resamples, holding the cross-validated predictions fixed). The LLM leg's
faithfulness interval is bootstrapped from the committed per-case scores
(`n = 15` per tier), and the credit-model protected-attribute reliance is
recomputed over the same committed case indices — neither step calls the API.

Each notebook is a **single self-contained code cell**. Figures are written to
`results/figures/` as both `.png` and `.pdf` at 600 dpi in grayscale; tables are
written to `results/tables/` as `.csv` and LaTeX (`.tex`). All random seeds are
fixed (`random_state=42`).

### LLM configuration (notebook 04)

Create a file named `.env` in the repository **root** (one level above
`notebooks/`):

```
OPENAI_API_KEY=...
LLM_MODEL_WEAK=gpt-4o-mini
LLM_MODEL_MID=gpt-5.4-mini
LLM_MODEL_STRONG=gpt-5.4
```

The notebook loads it with `python-dotenv` (`find_dotenv`) and reads the three
model tiers from the environment. API access may be restricted in cloud
sandboxes; run this notebook on a local machine.

## Repository layout

```
.
├── data/                 # public datasets (git-ignored; auto-downloaded)
├── notebooks/            # one self-contained cell per notebook
├── results/
│   ├── figures/          # .png + .pdf, 600 dpi, grayscale
│   └── tables/           # .csv + .tex
├── requirements.txt
├── .env.example
└── README.md
```
