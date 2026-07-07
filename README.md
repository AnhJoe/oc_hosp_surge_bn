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

Both models use partial pooling across facilities (each hospital gets its own baseline volume, informed by the system-wide trend) and a Negative Binomi