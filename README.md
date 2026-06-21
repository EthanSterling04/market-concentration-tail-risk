# Supply Chain Concentration and Equity Options Tail Risk

**Ethan Sterling — Senior Economics Thesis**

This repository contains all data, code, and output for my thesis examining whether supply chain network concentration predicts options-implied tail risk across U.S. equity sectors.

## Research Question

Does supplier concentration — measured via the Herfindahl-Hirschman Index (HHI) derived from BEA Input-Output tables and Bloomberg constituent data — predict the one-month-ahead 25-delta implied volatility risk reversal on SPDR sector ETFs?

The thesis covers 11 GICS sectors (Technology, Health Care, Energy, Financials, Industrials, Consumer Staples, Consumer Discretionary, Utilities, Materials, Real Estate, Communication) over January 2005 – February 2026.

## Key Variables

| Variable | Description | Source |
|---|---|---|
| `RR_{j,t+1}` | One-month-ahead 25-delta implied vol risk reversal (dependent) | OptionMetrics Ivy DB |
| `HHI` | Monthly market-cap concentration among S&P 500 sector constituents | Bloomberg Terminal |
| `Supplier_HHI` | Network-weighted upstream supplier concentration via I/O tables | BEA + Bloomberg |
| `Hub_Shock` | Binary: volatility spike in rolling top-3 heavyweight constituents | Bloomberg |
| `ATM_IV` | At-the-money implied volatility | OptionMetrics |
| `Term_Spread` | 10Y–2Y Treasury spread | FRED |
| `Credit_Spread` | Baa–Aaa corporate bond yield spread | FRED |

## Models

Five panel specifications with entity and time fixed effects, standard errors clustered at the sector ETF level:

- **M1 (Baseline Structural):** HHI + Supplier HHI + controls
- **M2 (Hub Shock):** HHI + Hub Shock + controls
- **M3A (Heterogeneous HHI):** HHI + sector × HHI interactions + Supplier HHI + controls
- **M3B / M3C:** Extended specifications with heterogeneous Supplier HHI interactions

Out-of-sample evaluation uses a fixed train/test split: train Jan 2005 – Dec 2019, test Jan 2025 – Feb 2026 (2020–2024 held out as a buffer).

## Repository Structure

```
Thesis/
├── Econ_Thesis_FINAL.pdf          # Written thesis (final submission)
├── Thesis Poster.pdf              # Poster presentation
└── Experiment/
    ├── Experiment.ipynb           # Main notebook: data cleaning, models, figures
    ├── Master_Predictive_Panel.csv  # Final merged panel dataset
    ├── Panel_Sample.csv           # 50-row sample of panel for inspection
    ├── Options_Data_Sample.csv    # 50-row sample of options data
    │
    ├── Data/
    │   ├── Data_Extraction.ipynb        # Bloomberg API data pull (requires Terminal)
    │   ├── HHI/                         # Per-sector monthly HHI CSVs (11 files)
    │   ├── IO_Tables/                   # BEA annual I/O tables 2004–2024 (CSV)
    │   ├── Intermediates/               # Parquet caches: mkt cap + 30d vol per sector
    │   ├── Options_Data.csv             # Raw OptionMetrics options data
    │   ├── Options_Risk_Reversal_OptionMetrics.csv  # Processed risk reversals
    │   ├── Network_Supplier_HHI.csv     # Final supplier HHI panel
    │   ├── Sector_Controls.csv          # ETF total return index + P/E ratios
    │   ├── Sector_Hub_Shock.csv         # Hub Shock binary flags
    │   ├── Sector_Cascade_Proxy.csv     # Earlier cascade trigger (superseded)
    │   ├── Macro_Controls_FRED.csv      # VIX, term spread, credit spread
    │   ├── Realized_Tail_Risk_Placeholders.csv  # Realized skewness + downside vol
    │   ├── omega_matrix_2024.csv        # BEA inter-sector weight matrix (2024)
    │   └── annual_weights.pkl           # Full annual omega matrices (all years)
    │
    ├── Figures/
    │   ├── fig_net_sector_effects.{pdf,png}     # M3A net HHI effects by sector
    │   ├── fig_supplier_hhi_evolution.{pdf,png} # Supplier HHI over time by sector
    │   └── fig_io_heatmap.{pdf,png}             # BEA I/O weight matrix heatmap
    │
    ├── Tables/
    │   ├── table1_main_results.tex              # M1, M2, M3A in-sample results
    │   ├── table2_extended_results.tex          # Extended model results
    │   ├── table3_oos_results.tex               # OOS forecast evaluation
    │   ├── table4_net_sector_effects.tex        # M3A net sector-specific HHI effects
    │   ├── table5_coefficient_magnitudes.tex    # Economic magnitude (bps per 1 SD)
    │   ├── appendix_dk_robustness.tex           # Driscoll-Kraay SE robustness
    │   └── appendix_loo_stability.tex           # Leave-one-out cross-validation
    │
    └── Output/
        ├── regression_results.{tex,html}        # Full in-sample regression output
        ├── oos_fixed_window.{csv,tex}           # OOS forecast results
        ├── dk_robustness.{csv,tex}              # Driscoll-Kraay robustness results
        ├── cv_loo_stability.csv                 # LOO stability summary
        └── cv_loo_folds/                        # Per-fold LOO results (M1, M3A–M3C)
```

## Reproducing the Analysis

### Prerequisites

- Python 3.x with: `pandas`, `numpy`, `statsmodels`, `linearmodels`, `scikit-learn`, `matplotlib`, `pyarrow`
- Bloomberg Terminal access + `xbbg` for data extraction
- OptionMetrics Ivy DB access for options data

### Steps

1. **Data extraction** — Run `Experiment/Data/Data_Extraction.ipynb` on a machine with an active Bloomberg Terminal connection. This pulls HHI constituent data, sector controls, and realized tail risk metrics. BEA I/O tables are downloaded manually from [bea.gov](https://www.bea.gov/industry/input-output-accounts-data).

2. **Main analysis** — Run `Experiment/Experiment.ipynb` sequentially:
   - *Data Cleaning* section: processes I/O tables into omega matrices and Supplier HHI, cleans OptionMetrics data, and merges all sources into `Master_Predictive_Panel.csv`
   - *Models* section: runs in-sample panel regressions (M1–M3C), OOS forecasting, LOO cross-validation, and Driscoll-Kraay robustness checks
   - *Figures* section: generates all three thesis figures

All regression tables are saved to `Experiment/Tables/` and model output to `Experiment/Output/`.

## Data Notes

- `Data_Extraction.ipynb` requires a live Bloomberg Terminal session and cannot be re-run without one; all intermediate outputs are cached in `Data/Intermediates/` as parquets.
- The 2020–2024 period is intentionally excluded from OOS testing (held out as a buffer between training and test windows).
- XLRE (Real Estate) and XLC (Communication) ETFs launched in 2015 and 2018 respectively, so the panel is unbalanced.
- Within-R² values are negative in some specifications because two-way fixed effects absorb substantial variation; inference rests on joint F-tests and individual t-statistics.
