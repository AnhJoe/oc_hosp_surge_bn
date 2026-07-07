# OC SurgeWatch: Bayesian Modeling of Surge Capacity for Orange County Hospitals

**Joe T. Nguyen** · UC Irvine, Master of Data Science · STATS 205P (Bayesian Statistics), Spring 2026

## Overview

Orange County's Health Care Agency Hospital Surge Plan identifies wildfire, earthquake, and pandemic as the county's top disaster hazards. During the first 72 hours of any such event, responders must estimate which Emergency Receiving Center (ERC) hospitals have the capacity to absorb a surge of new patients — and which are already stretched thin.

This project builds a **Bayesian hierarchical negative-binomial model** on seven years of hospital discharge data to answer two questions:

1. **Which ERC hospitals are most vulnerable to a patient volume surge?**
2. **What facility-level characteristics predict resilience versus fragility?**

The output is a facility-level posterior predictive distribution of 2024 admission volume, converted into a probability of breaching each facility's own 95% bed-capacity threshold — a monitoring framework that could plausibly support real disaster-response decision-making.

## Data

- **Source:** [California HCAI Hospital Inpatient PDD Pivot Profile](https://data.chhs.ca.gov/dataset/hospital-inpatient-characteristics-by-facility-pivot-profile), 2018–2024.
- **Scope:** 21 Orange County ERC hospitals with a complete 7-year panel (147 facility-years). Pediatric-only, rehab/LTACH, psychiatric, elective-specialty, and non-ERC facilities are excluded, as are hospitals with incomplete panels — see the exclusion list in `notebooks/00_preprocess.Rmd`.
- **Primary outcome:** `toc_acute` — total annual acute admissions per facility.
- **Scorecard features:** baseline (2018–2019) occupancy rate and 60+ population share, used as fixed predictors alongside per-facility random intercepts.
- Raw source files live in `data/`; the cleaned analysis panel is cached at `data/processed/pdd_panel.rds`.

## Method

| Model | Question | Structure |
|---|---|---|
| **M1 — Base** | Which facilities are most vulnerable? | `toc_acute ~ year_idx + (1 \| oshpd_id)`, Negative Binomial, fit on 2018–2023, forecast 2024 |
| **M2 — Scorecard** | What predicts vulnerability? | Adds standardized baseline occupancy and 60+ share as fixed effects |

Both models use partial pooling across facilities (each hospital gets its own baseline volume, informed by the system-wide trend) and a Negative Binomial likelihood to absorb the severe overdispersion found in EDA (facility-level variance runs 250–1,000× the Poisson expectation). Models are validated with prior predictive checks, R-hat/ESS convergence diagnostics, LOO comparison, and posterior predictive evaluation against the fully held-out 2024 test year.

## Key Findings

- **85.7%** of 2024 actuals fell inside the base model's 90% posterior credible interval; the scorecard model improved coverage to **90.5%**.
- The model is reliable for mid-sized facilities but tends to **underestimate demand at the largest facilities** — a consequence of the bimodal size distribution (small community hospitals vs. large regional centers) — which is precisely where forecast accuracy matters most in a disaster.
- Facilities already operating at high volume are the most likely to breach capacity; a facility with little buffer today has little buffer during a system-wide surge.
- **Baseline occupancy rate carries meaningful predictive weight; 60+ population share adds noise, not information** (it is nearly collinear with Medicare share and does not improve predictions once occupancy is included).

Full writeup and figures: [`Bayesian OC Hospitals Report.pdf`](./Bayesian%20OC%20Hospitals%20Report.pdf).

## Repo Structure

```
data/                       Raw HCAI PDD pivot files (2018–2024) + cleaned panel (data/processed/)
notebooks/
  00_preprocess.Rmd         Load raw years, bridge schema shifts, build the analysis panel
  01_eda.Rmd                Distribution, overdispersion, temporal, and scorecard EDA
  02_modeling.Rmd           Priors, M1/M2 fitting, convergence, PPC, capacity monitoring
models/                     Cached brms model fits (m_prior, m_base, m_scorecard)
Bayesian OC Hospitals Report.pdf   Slide-deck summary of the full analysis
```

Run the notebooks in order (`00` → `01` → `02`); each stage reads the output of the previous one.

## Limitations

- Data is annual aggregates only — real-time or weekly/monthly granularity would materially improve inference.
- No staffed-bed, nurse-staffing-ratio, ICU-capacity, or diversion-rate data was available; these would be more informative surge indicators than licensed bed count.
- Emergency admission counts are unreliable in several facility-years (values exceeding total admissions, concentrated in 2020–2021 — likely MNAR reporting waivers), so that scorecard feature was dropped.
- Findings should be interpreted as a proof of concept rather than an operational forecasting system.

## License

MIT — see [LICENSE](./LICENSE).
