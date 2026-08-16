# Insuring Against the Dismantling of GHGRP:
## A Predictive Climate Intelligence Framework for Corporate Decarbonization Credibility
Author: Souvik Mandal, Project leader, LS100, FAS, Harvard University, [LinkedIn Profile](https://www.linkedin.com/in/souvik-mandal-phd/)

This project builds a public-goods framework for the top 50 U.S. power-sector parent companies — the **Corporate Climate Credibility Score (CCCS)** — that combines 
- corporate pledges using multiple sources (SEC 10-K annual report for each company, Net Zero Tracker, SBTi), scored through multiple LLMs,
- self-reported and regulator-verified emissions data (EPA GHGRP data and CEMS continuous stack measurement data), and
- independent satellite cross-checks (Climate TRACE remote-sensing-derived estimates)

into a single, decomposable credibility score per parent company, surfaced through a self-contained interactive dashboard.

All the data used in this project are publicly available, and can also be downloaded from the `data` folder of this repo. However, not all SEC_10 K raw data has been uploaded to this repo due to size constraints; please follow the `00_Data_Acquisition.ipynb` to obtain the full dataset.

NOTE: This is a proof-of-concept project, not a production system. Though the project analyses only 50 U.S. power-sector parent companies, with further work, this framework can be extended to other companies as well.

---

| Component | Source signal | Notebook |
|---|---|---|
| Pledge Quality Score **PQS** | SBTi + Net-Zero Tracker + SEC 10-K, scored by dual LLMs across 5 criteria | `05_Pledge_Quality.ipynb` |
| EPA Performance Score **EPS** | Rank-percentile of `pct_per_yr` decarbonization rate on EPA GHGRP 2011-2023 | `06_CCCS_Composite.ipynb` |
| Satellite Cross-Validation Score **SCVS** | Rank-percentile of `\|LME intercept\|` between Climate TRACE and EPA GHGRP | `03_Statistical_Modeling.ipynb` → `06_CCCS_Composite.ipynb` |

---

## Quick Start
From starting to running the first notebook can be done within ≈ 5 minutes.

### 1 — Clone and install
```bash
git clone https://github.com/Souvik-Mandal-Harvard/climate-ghgrp-data-science
cd climate-ghgrp-data-science
python -m venv .venv && source .venv/bin/activate   # optional but recommended
pip install -r requirements.txt
```

### 2 — Configure credentials
Copy `.env.example` to `.env` and fill in:
- `EPA_CAMD_API_KEY` — free, instant: https://www.epa.gov/airmarkets/cam-api-portal
- `SEC_EDGAR_UA` — required by SEC EDGAR, format `"Your Name your.email@example.com"`
- For analyzing the corporate climate pledges (in Notebook 5), you need to get  OpenAI and Anthropic API keys.

If you only want to **open the dashboard** (which ships all data inline), neither key is required.

### 3 — Open the dashboard
```bash
open dashboard/index.html        # macOS
xdg-open dashboard/index.html    # Linux
start dashboard\index.html       # Windows
```
The dashboard is a single self-contained HTML file (~4.91 MB, `plotly.js` bundled inline). No server, no install, works offline. Move the three sliders to see how the CCCS ranking is sensitive to weight choice; pick a parent from the dropdown to see its three-axis component breakdown and 2011-2023 GHGRP emissions trajectory; click any column header in the Audit table to sort.

### 4 — Reproduce the analysis from scratch
The notebooks are designed to run **top-to-bottom in numeric order**. The full chain takes ~2–4 hours end-to-end (LLM scoring in NB 05 is the longest step). Per notebook details below.

---

## Notebook Execution Order

| # | Notebook | Purpose | Runtime | Key outputs |
|---|---|---|---|---|
| **00** | `00_Data_Acquisition.ipynb` | Download all raw inputs (EPA GHGRP, EPA CEMS via CAMD API, Climate TRACE, SBTi, Net-Zero Tracker, SEC 10-K) | 20-40 min | `data/raw/**`, `data/processed/cohort_top50.csv`, `data/processed/data_manifest.json` |
| **01** | `01_EDA_GHGRP.ipynb` | Profile the EPA GHGRP universe; identify top-50 cohort; fuel-mix + geographic + threshold-effect EDA | 5 min | EDA figures (inline) |
| **02** | `02_Entity_Resolution.ipynb` | Crosswalk EPA GHGRP × CEMS × Climate TRACE at facility-year; build parent-year panel; coverage diagnostics | 5 min | `cohort_facility_year_panel.csv`, `multi_source_facility_year_panel_v2.csv`, `cohort_parent_year_panel.csv` |
| **03** | `03_LME_and_CEMS.ipynb` | **§2** CEMS validation panel (slope/intercept/R² per facility, Bonferroni-corrected). **§3** Diagnostic LME (`log_ghgrp ~ log_ct + log_load`) with the three-model pedagogical arc and transitivity decomposition. **Outputs SCVS inputs** (per-parent intercepts) | 3 min | `lme_per_parent_intercepts.csv`, `lme_diagnostic_summary.json`, `cems_validation_summary.json` |
| **05** | `05_Pledge_Quality.ipynb` | Dual-LLM (Claude Sonnet 4.6 + GPT-4o, T=0) scoring of multi-source pledge corpus over 5 criteria; inter-model agreement diagnostics; **outputs PQS** | 30-90 min (LLM-bound) | `cohort_pqs.csv`, `cohort_pqs_intermodel_agreement.csv`, `cohort_pqs_llm_reasoning.json` |
| **06** | `06_CCCS_Composite.ipynb` | Compute EPS (`pct_per_yr` ranks), SCVS (`\|intercept\|` ranks), assemble CCCS as weighted geometric mean, 2×2 quadrant assignment, 6-scheme sensitivity, rank-stability flags | 2 min | `cohort_cccs.csv`, `cohort_cccs_components.csv`, `cohort_cccs_rank_stability.csv`, Plotly + matplotlib quadrant figures |
| **07** | `07_Dashboard.ipynb` | Assemble the self-contained HTML dashboard (Macro 2×2 + Drilldown + Audit tiers) | 1 min | `dashboard/index.html` |

**Notebook 04 (Prognostic ML virtual baseline) is intentionally not in the chain** — see "Scope decisions" below.

---

## Data Sources & Licensing

| Source | URL | License / Terms |
|---|---|---|
| EPA Greenhouse Gas Reporting Program (GHGRP) | https://www.epa.gov/ghgreporting | Public domain |
| EPA Clean Air Markets — CAMD API (CEMS) | https://www.epa.gov/airmarkets/cam-api-portal | Public domain (API key required, free) |
| Climate TRACE | https://climatetrace.org | CC BY 4.0 |
| Science Based Targets initiative (SBTi) | https://sciencebasedtargets.org | Public dashboard |
| Net Zero Tracker | https://zerotracker.net | CC BY 4.0 |
| SEC EDGAR 10-K filings | https://www.sec.gov/edgar | Public domain (polite-UA required) |

---
