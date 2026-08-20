# OWAGDP — Ordered Weighted Averaging of Institutional GDP Forecasts

Data, code, and results for the paper:

> H. S. Baraheem and J. M. Merigó, "Ordered Weighted Averaging of Institutional
> GDP Forecasts." 

The paper aggregates the 2026 and 2027 real GDP growth forecasts of the IMF,
the OECD, and the European Commission for 37 economies with an ordered weighted
averaging (OWA) operator, orders the forecasts by past accuracy through an
induced operator, and propagates historical forecast error through the
aggregation by exact enumeration of an empirical bootstrap distribution.

## Repository structure

Each numbered folder is one pipeline stage. Every stage has a `Data/` folder
holding its inputs (copies of the previous stage's outputs, so each notebook
runs standalone) and a `Result/` folder holding its outputs.

| Stage | Notebook | Inputs | Main outputs |
|---|---|---|---|
| 01_build_panel | `01_build_panel.ipynb` | Current-vintage IMF / OECD / EC forecasts | `panel.csv` (74 rows: 37 economies × 2 years) |
| 02_aggregate | `02_aggregates.ipynb`, `fig1.ipynb` | `panel.csv` | `aggregates.csv` (OWAGDP at α = 0.1–0.9, spread, range), `weights.csv`, Fig. 1 |
| 03a_build_error | `03a_build_error.ipynb` | Archived OECD EO 108–112, IMF Historical WEO, AMECO vintages | `forecast_errors.csv` (181 rows: archived forecasts and realised outcomes, five rounds Autumn 2020–Autumn 2022) |
| 03b_accuracy | `03b_accuracy.ipynb` | `aggregates.csv`, `forecast_errors.csv` | `mae.csv` (per-country MAE, induced order, separation), `induced.csv` (IOWAGDP vs OWAGDP per cell) |
| 03c_rescore | `03c_rescore.ipynb` | `forecast_errors.csv`, AMECO Spring 2026 | `rescore.csv` (robustness of the accuracy ranking to the outcome series) |
| 04_uncertainty | `04_uncertainty.ipynb` | `aggregates.csv`, `forecast_errors.csv` | `uncertainty_full.csv`, `uncertainty_excl2021.csv` (exact 2.5–97.5 percentile ranges), Fig. 2 |
| 05_verification | `05_verification.ipynb` | all of the above | prints every number quoted in the paper against the result files |

Note on naming: `forecast_errors.csv` contains archived **forecasts and
realised outcomes** (columns `iso, round, target, IMF, OECD, EC, actual`);
signed and absolute errors are derived inside each notebook as
`forecast − actual`.

## Method notes

- OWA weights are maximum-entropy vectors on a declared orness grid
  (geometric form `w_i ∝ h^(i−1)`); displayed tables are rounded, all
  computations use unrounded weights.
- The accuracy-induced ordering uses `u = 1/MAE` per country; only the rank
  of `u` enters.
- The uncertainty calculation enumerates all 5³ = 125 independent error
  combinations per economy (4³ = 64 excluding the 2021 target) and takes
  percentiles from the inverse empirical distribution function
  (`numpy.percentile(..., method='inverted_cdf')`). No random draws and no
  seed are involved; results are exactly reproducible.
- At α = 0.5 the weights are equal, so the percentile-range width is
  identical for the 2026 and 2027 forecasts of the same economy by
  construction.

## Running

The notebooks were written for Google Colab. Each notebook locates its input
files by column signature, so it can be run either

1. from a Google Drive folder mounted in Colab, or
2. from a local clone, with the working directory set to the stage folder.

Run order: 01 → 02 → 03a → 03b → 03c → 04 → 05. Stage 05 should end with
`86 of 86 checks passed`.

Dependencies: Python 3, `numpy`, `pandas`, `matplotlib`. NumPy ≥ 1.22 is
required for `method='inverted_cdf'`.

## Data sources

- IMF, World Economic Outlook (April 2026) and the
  [Historical WEO Forecasts Database](https://www.imf.org/external/pubs/ft/weo/data/WEOhistorical.xlsx)
- OECD, Economic Outlook No. 119 (June 2026) and archived editions 108–112
  via the [OECD Data Explorer](https://data-explorer.oecd.org/)
- European Commission, Spring 2026 Forecast and archived
  [AMECO](https://economy-finance.ec.europa.eu/economic-research-and-databases/economic-databases/ameco-database_en)
  vintages, Autumn 2020–Autumn 2022

Raw source extracts are redistributed here only to the extent needed for
reproducibility; the originals remain subject to the terms of their
publishers.

## License
Code and derived data are released under the MIT License (see `LICENSE`).

## Citation
If you use this repository, please cite the paper above. A DOI for the
archived release is available via Zenodo (see the badge at the top of this
page once the release is archived).
